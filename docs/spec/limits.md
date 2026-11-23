# Limits and edge cases

All values are preliminary and may change. In code they are set via configuration, not constants.

## 1. Account

| Parameter | Value |
|---|---|
| Time to confirm email (after it, an unconfirmed account is deleted) | 24 hours |
| Email resend | at most once per minute, at most 5 per hour |
| Nickname | 3–30 characters |
| Password | 8–128 characters |

## 2. Sessions

| Parameter | Value |
|---|---|
| Idle lifetime | 14 days |
| Absolute lifetime | 30 days |
| Last activity update | at most once per 5 minutes |

## 3. Email tokens

| Type | Lifetime |
|---|---|
| Email confirmation | same as the time to confirm email (section 1) |
| Password recovery | 30 minutes |
| Password change, email change, account deletion | 1 hour |

## 4. Files and folders

| Parameter | Value |
|---|---|
| Max file size | 500 MB |
| Concurrent uploads per user | 3 |
| Folder nesting depth | 32 levels |
| Objects per folder | 1,000 |
| Name length | 1–255 characters |
| Forbidden characters in names | `/ \ : * ? " < > |` and control characters |
| Forbidden names | `.`, `..`, whitespace-only names, names ending with a space or a dot |
| In-browser preview: text | up to 1 MB |
| In-browser preview: images | up to 20 MB |
| In-browser preview: PDF | up to 50 MB |
| Search results | up to 100 |

## 5. Storage and quotas

| Parameter | Value |
|---|---|
| Free quota | 500 MB |
| Trash retention | 7 days |
| Incomplete upload retention | 24 hours |
| Total storage limit for the whole service | 50 GB |

When the total limit is reached, uploads are temporarily unavailable for all users with the message "The service is temporarily not accepting new files", and the admin is notified. This protects against an unexpected cloud bill.

## 6. Plans (test mode)

| Size | Price | Duration |
|---|---|---|
| 500 MB | 10 ₽ | 30 days |
| 1 GB | 20 ₽ | 30 days |
| 1.5 GB | 30 ₽ | 30 days |

Expiry reminder — 3 days before. Renewal and plan-combination rules are defined separately.

## 7. Sharing

| Parameter | Value |
|---|---|
| Access grants by email | at most 20 per day per user |
| Waiting access for an unregistered email (functional.md, 4.2) | 30 days; if the email owner does not register and confirm the email within this time, the access is removed |
| Public link creation | at most 50 per day per user |

## 8. Rate limits

| Action | Limit |
|---|---|
| Sign-in | 10 attempts per minute per IP |
| Failed sign-in to an account | after 5 failures — a delay that doubles with each further failure (up to 15 minutes) |
| Registration | 5 per hour per IP |
| Password recovery | 3 per hour per email, 10 per hour per IP |
| Other API requests | 10 per second per user |

When a limit is exceeded, `429 Too Many Requests` is returned with the time after which the request can be retried.

## 9. File URLs and log

| Parameter | Value |
|---|---|
| Upload presigned URL (time within which the upload must start) | 15 minutes |
| Download and preview presigned URL (time within which the download must start) | 5 minutes |
| Activity log retention | 180 days |

## 10. Edge cases

### Upload
- **Quota reservation.** When an upload URL is issued, space for the declared file size is reserved. The reservation is released when the upload completes, is cancelled or expires. This way parallel uploads cannot exceed the quota together.
- **Upload interrupted.** The file stays in the "uploading" state and is not shown in the list. After 24 hours the record and the partially uploaded object are deleted and the reservation is released.
- **Target folder deleted during upload.** Upload completion is rejected, the object is deleted, and the user sees an error in the upload panel.
- **Plan expired during upload.** An upload already in progress completes: space for it was reserved.
- **Actual file size differs from the declared size.** The object is deleted and the upload is considered failed.

### Object actions
- **Moving a folder into itself or its subfolder.** Rejected with a clear error.
- **Name conflict** on upload, create, move, rename, restore — an index is added (functional.md, 2.3). Uniqueness is guaranteed by a database constraint, not only a check in code: two concurrent requests cannot create the same name.
- **Object changed in another tab.** An action on an already deleted or moved object returns `404`; the user sees "This object no longer exists" and the list refreshes.
- **Permanent deletion.** The object disappears at once and the quota is freed. The files are deleted from storage by a background job through a deletion queue (database.md). On error the job retries; if deletion keeps failing, the admin is notified.

### Account
- **Following a confirmation link again.** Message "Email already confirmed" and a "Go to drive" button.
- **Expired or invalid link.** Message and a button to request a new one.
- **Registration with an email taken by an unconfirmed account.** The email is considered taken until the account is deleted (up to 24 hours).
- **Block during an open session.** All sessions are deleted at the moment of blocking; the user's next request gets `401`, and the sign-in page shows the reason and the block duration.
- **Nickname change.** Granted access is kept: it is bound to the user, not the nickname.
- **Owner account deleted.** The owner's objects disappear from "Shared with me" for all recipients, and public links stop working.

### Quota
- **Admin reduced the quota below the used space.** The quota-exceeded mode is enabled (functional.md, 5.1).

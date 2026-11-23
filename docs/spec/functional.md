# Functional requirements

Numeric limits (sizes, durations, counts) are in limits.md and referenced here as [L: ...].

## 1. Account

### 1.1 Registration
- The user registers with an email, a nickname and a password.
- Registration with a blacklisted email is not possible.
- After registration, an email with a confirmation link is sent.
- The user can request the email again [L: resend rate].

### 1.2 Unconfirmed account
- Until the email is confirmed, every page shows a warning with the remaining time and a "Resend email" button.
- An unconfirmed user can use the drive but cannot share files or folders.
- If the email is not confirmed within 24 hours after registration, the account and all its files are deleted. The email and nickname become available again.

### 1.3 Nickname
- Required, set during registration.
- Unique, case-insensitive: `Damir` and `damir` are the same nickname.
- Allowed characters: Latin letters, digits, `_`, `.`, `-` [L: length].
- Can be changed in settings without email confirmation.
- The old nickname becomes available after a change.
- Other users see the nickname when files are shared with them.

### 1.4 Sign in and sign out
- Sign in with email or nickname and password.
- An unconfirmed user can sign in.
- A blocked user cannot sign in and sees a message with the reason and the end date of the block.
- Sign out of the current session and "Sign out on all devices".
- The user sees the list of signed-in devices (device and browser, IP address, last activity) and can sign out any of them.

### 1.5 Password recovery (user forgot the password)
- Via "Forgot password?" on the sign-in page, the user enters an email and receives a link to set a new password.
- The response is the same whether or not the email exists.
- After a new password is set, all active sessions are terminated.

### 1.6 Account settings changes
- **Password change** (user knows the password): the user enters the current and the new password; the change takes effect after following the link in the email. All other sessions are terminated. If the current password is forgotten, recovery (1.5) is used.
- **Email change:** a link is sent to the new address. The email changes after the link is followed. A notification about the change is sent to the old address.
- **Account deletion:** the user enters the password, then confirms the deletion via the link in the email. After confirmation, the account, all files and all granted access are permanently deleted.

## 2. Files and folders

### 2.1 Structure
- Every user has a root folder.
- Folders can be created, including empty ones [L: nesting depth, objects per folder].

### 2.2 Upload
- Only files can be uploaded; uploading whole folders is not supported.
- Several files can be selected at once or dragged into the window.
- An upload is rejected if the file exceeds the limit [L: max file size] or does not fit into the remaining quota.
- Upload progress is shown; an upload can be cancelled.

### 2.3 Names
- A file or folder name is unique within its parent folder, case-insensitive.
- On a name conflict (upload, create, move, restore from trash), an index is added to the new object automatically: `report.pdf` → `report (1).pdf` → `report (2).pdf`. For folders, the index is added to the end of the name.
- Forbidden characters and max name length [L: names].

### 2.4 Actions
- On one object: open, download, rename, move, delete, share.
- On several selected objects: move, delete.
- A folder cannot be moved into itself or into its own subfolder.

### 2.5 In-browser preview
- Images: JPEG, PNG, GIF, WebP.
- PDF.
- Text: TXT, MD and common code and configuration formats [L: max preview size].
- Other files can only be downloaded.

### 2.6 Search and sorting
- Search by part of the name, case-insensitive, across the user's whole drive. Results show the path of each object.
- Sorting by name, modification date, size and type, ascending and descending. Folders are always listed before files.

## 3. Trash
- A deleted file or folder is moved to the trash. A folder goes there as a whole, as a single object.
- An object stays in the trash for 7 days, then is permanently deleted.
- Objects in the trash still count towards the quota.
- In the trash the user can: restore an object, permanently delete an object, empty the trash.
- An object is restored to its original folder. If that folder no longer exists, it is restored to the root.
- Objects in the trash are not accessible via links or granted access.

## 4. Sharing

### 4.1 Public link
- A public link can be created for a file or a folder. One object has at most one link.
- Links to different objects are independent: a file can be accessible both via its own link and via the link of any folder containing it.
- Anyone can open an object via its link, including unregistered users.
- For a folder, the link gives access to all its contents, including subfolders.
- A link can be disabled. A disabled link stops working forever; a new link will be different.

### 4.2 Access for a specific user
- The owner can grant access to a file or folder by nickname or by email.
  - By nickname — only to an existing user.
  - By any email, whether or not its owner is registered. The response is the same in both cases.
- The recipient gets a notification email. If the email is not registered, the email contains an invitation to register. Access appears after registration and confirmation of that email.
- The recipient sees the object in the "Shared with me" section.
- If a user has access both to a folder and to an object inside it, only the top-level folder is shown in "Shared with me". If access to the folder is revoked, the nested object is shown separately again.
- The recipient can remove an object from their "Shared with me" section; this revokes their access.
- Shared objects do not count towards the recipient's quota.

### 4.3 Permissions
- Any access (link or user) allows only viewing and downloading.

### 4.4 Access management
- The owner sees all their links and granted access in settings and can remove any of them.
- Objects with any access granted are marked with an icon in the file list.
- When an object is moved to the trash, access to it is suspended; on restore it works again. On permanent deletion, access is removed.
- If an object is moved out of a shared folder, access to it through that folder is lost.

## 5. Quota
- Every user gets a free quota [L: free quota].
- Total quota = free quota + active purchased plans.
- Used and free space is shown as a bar on the main drive page and stays visible when navigating folders and sections.
- The quota includes all the user's files, including the trash.

### 5.1 Quota exceeded
Happens when a purchased plan expires or an admin reduces the quota, and the used space exceeds the available space.
- The user sees a warning on every page.
- Allowed: view and download files, delete to trash and permanently, empty the trash, move and rename, buy a plan.
- Forbidden: upload files and create new objects.
- Granted access and links keep working.
- Restrictions are lifted automatically as soon as the used space is below the quota.

## 6. Extra storage
- The user buys a plan for 30 days. There is no auto-renewal; renewal is manual.
- A reminder email is sent 3 days before the plan expires.
- Purchased space is added to the quota after the payment provider confirms the payment.
- Payments work in test mode; no real money is charged. The payment provider sits behind an interface: a stub first, a real provider in test mode later.
- Plans [L: plans].

## 7. Administration
- The admin role is assigned manually (in the database or configuration), not through the UI.
- An admin can:
  - view the user list: email, nickname, registration date, status, used space;
  - view the user activity log (sign-in, upload, deletion, sharing, account changes) — overall and per user;
  - view recently uploaded files — overall and per user;
  - open and download users' files for moderation;
  - delete users' files;
  - change a user's quota.
- Sanctions:
  - **temporary block** until a given date with a reason. All sessions are terminated; public links and granted access stop working until the block ends. The block is lifted automatically when it ends;
  - **account deletion with the email blacklisted.** The account and all files are deleted; registration with this email is no longer possible.
- All admin actions are recorded in the log.
- The terms of service (a page on the site) state that the administration may view files during moderation and delete prohibited content.

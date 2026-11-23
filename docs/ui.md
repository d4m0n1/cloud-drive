# UI: pages and actions

Functional rules are in functional.md; this file covers only what the user sees and does. UI labels are given in English; Russian translations live in the locale files.

## General

- **Site language:** Russian and English, Russian by default. Switcher on the sign-in screens and in settings. The choice is saved in the account. Emails are always in Russian.
- **Theme:** light and dark, system default. Switcher in the corner of the sign-in and drive screens, and in settings.
- **Responsive layout:** all pages work correctly from 360 px width. On mobile:
  - the sidebar collapses into a menu (burger button);
  - the file list is simplified: icon, name, size;
  - actions are available via the "⋯" button on an object;
  - upload only via the button, no drag and drop.
- **Action feedback:** short pop-up messages (toasts) in the corner of the screen — success, error.
- **Confirmations:** permanent deletion, emptying the trash, disabling a link, revoking access and deleting the account require confirmation in a dialog.
- **Loading:** while data is loading, a thin progress bar is shown at the top of the page and a loading indicator in place of the content, not an empty screen.
- **Time** is shown in the browser's time zone.

## Routes

| Route | Page | Access |
|---|---|---|
| `/` | Landing | everyone |
| `/login` | Sign in | guests |
| `/register` | Registration | guests |
| `/forgot-password` | Password recovery | guests |
| `/reset-password#token=` | New password | link from email |
| `/confirm#token=` | Confirmation result (email, email change, password change, account deletion) | link from email |
| `/disk/*` | My drive (folder path in the URL) | users |
| `/shared` | Shared with me | confirmed users |
| `/trash` | Trash | users |
| `/plans` | Plans and payment | users |
| `/settings` | Settings | users |
| `/s/{token}` | Public object page | everyone |
| `/admin/*` | Admin panel | admins |
| `/terms`, `/privacy` | Terms and privacy policy | everyone |

Tokens in email links are passed after `#`: this part of the URL is not sent to the server and does not end up in logs. The page immediately removes the token from the address bar and sends it to the backend in a POST request body.

A guest opening a user-only page is redirected to sign in and returned to the original page afterwards. A signed-in user opening `/login` or `/register` is redirected to the drive.

## Landing `/`
- Name, short description of the project and its features.
- Links to the GitHub repository and the author's Telegram.
- "Sign in" and "Sign up" buttons. If the user is already signed in — a single "Open drive" button.

## Sign in `/login`
- Fields: "Email or nickname", "Password" (with show/hide button).
- Links: "Forgot password?", "Sign up".
- One error for all invalid-credential cases: "Invalid login or password".
- A blocked user sees the reason and the end date of the block.
- Language and theme switchers, links to the terms and privacy policy.

## Registration `/register`
- Fields: "Email", "Nickname", "Password", "Repeat password".
- Password and nickname requirements are shown under the fields; errors appear while typing.
- Checkbox "I accept the terms of service and privacy policy" (links open the text in a dialog). The button is disabled without it.
- After registration the user goes straight to the drive and sees the "Confirm your email" dialog (see "Unconfirmed email").

## Password recovery `/forgot-password`
- "Email" field, "Send link" button.
- After submitting, always the same message: "If an account with this email exists, we have sent a link to it".

## New password `/reset-password`
- Fields "New password", "Repeat password".
- On success — a message and a "Sign in" button.
- If the link is invalid or expired — a message and a "Request a new link" button.

## Confirmation result `/confirm`
- Success message with a "Go to drive" button, or an error (link invalid, expired, already used) with a hint on what to do.

## Main layout (drive, shared with me, trash, plans, settings)

### Sidebar (left, always visible on desktop)
- "Upload file" and "Create folder" buttons.
- Sections: "My drive", "Shared with me", "Trash", "Plans", "Settings". For admins also "Admin".
- At the bottom: used-space bar ("340 MB of 500 MB used") and an "Get more space" button (leads to plans). When the quota is exceeded, the bar is red.

### Top bar
- Drive search.
- Theme switcher.
- Avatar menu: nickname, email, "Settings", "Sign out".

### Banners above the content
- **Unconfirmed email:** red banner "Confirm your email, otherwise the account will be deleted in N h." with a "Resend email" button. Cannot be closed. Right after registration a dialog with the same text is also shown; it can be closed.
- **Quota exceeded:** banner "Out of space: uploads are unavailable. Free up space or buy a plan" with a "Plans" button.
- **Plan expiring soon** (3 days before): banner with the end date and a "Renew" button. Can be closed.

## My drive `/disk/*`
- Heading — current folder name, with breadcrumbs above it (path from root, each item clickable).
- Top right: sorting (name, modification date, size, type; ascending/descending) and a view switcher (table / grid). The choice is remembered.
- **Table:** type icon, name, size, modification date, sharing icon.
- **Grid:** icon or image thumbnail, name below.
- **Opening:** double click (tap on mobile) on a folder opens it, on a file opens the preview.
- **Object actions:** via the "⋯" button and right click. Items: open, download, rename, move, share, delete.
- **Multiple selection:** checkboxes on hover (long press on mobile). When objects are selected, a bar appears at the top: "Selected: N", "Move", "Delete", "Clear selection".
- **Dragging files** from the computer into the window uploads them to the current folder; the drop area is highlighted while dragging.
- **Rename:** dialog with a field, the name is selected without the extension.
- **Move:** dialog with a folder tree; the current folder and invalid destinations are disabled.
- **Empty folder:** illustration, "Nothing here yet" text, and "Upload file" and "Create folder" buttons.
- **Search results:** list of found objects with paths; clicking a path opens the folder.

### Upload panel
- Appears in the bottom right corner during uploads: list of files with progress, a cancel button for each, a summary "Uploaded N of M" and per-file errors.

### File preview (dialog)
The dialog is the same for all files:
- **Title:** file name.
- **Content:** images, PDF and text are shown inside the dialog; other files show a large type icon.
- **Details:** size, type (format), upload date, modification date, location (path).
- **Buttons:** "Download", "Share", "Delete", "Close".
- Arrows to go to the previous and next file in the folder.
- Closes with the cross, the Esc key or a click outside.

### Share dialog
- **Public link:** "Access by link" toggle, a field with the link and a "Copy" button.
- **User access:** "Email or nickname" field, "Share" button. Below — a list of everyone with access (nickname, or email for not yet registered users) with a remove button for each.
- For an unconfirmed user the dialog is unavailable; a hint to confirm the email is shown instead.

## Shared with me `/shared`
- List of objects shared with the user: name, owner (nickname), date access was granted.
- Folders open inside the section; breadcrumbs start with the shared folder name.
- Available actions: open, download, "Remove from list".

## Trash `/trash`
- List: name, original path, deletion date, time left until permanent deletion.
- Object actions: "Restore", "Delete permanently". At the top — "Empty trash".
- Empty trash: "Trash is empty" text.

## Plans `/plans`
- Current quota: free part and active plans with end dates.
- Plan cards: size, price, duration, "Buy" button.
- A plan the user already has active cannot be bought: instead of "Buy" — "Active until …". Renewal and plan-combination rules are defined separately.
- "Buy" leads to the payment provider's (or stub's) payment page. After returning — a success or failure message.
- A note that payments work in test mode and no real money is charged.

## Settings `/settings`
Sections:
- **Profile:** nickname (change), email (change with confirmation).
- **Security:** change password, delete account.
- **Devices:** list of active sessions: device and browser, IP address, last activity; the current session is marked. A "Sign out" button for each session and "Sign out on all devices".
- **Access:** compact list of all the user's public links and granted access. Each entry: object, access type (link or user), to whom, date granted, and a remove button.
- **Interface:** language, theme.
- **Documents:** links to the terms of service and privacy policy.

## Public page `/s/{token}`
- A separate simple page without the sidebar.
- File: name, size, preview (if supported), "Download" button.
- Folder: list of contents with navigation into subfolders, file preview and download.
- At the bottom: "Store your files in cloud-drive" and a "Sign up" button.
- Link disabled, object deleted or owner blocked: "Link unavailable" page.

## Admin panel `/admin/*`
Same site and same sign-in. The section is visible only to users with the admin role; the backend checks the role on every request to `/api/v1/admin/**`.
- **Users:** table (email, nickname, registration date, status, used space), search by email and nickname, filter by status.
- **User card:** details, quota with the ability to change it, the user's files (with preview, download, delete), the user's activity log. Buttons "Block until…" (date and reason), "Unblock", "Delete and blacklist email".
- **Activity log:** event feed (time, user, action, object), filters by user, action type and date.
- **Recent uploads:** name, size, owner, date; a click opens the preview.

## Terms and privacy `/terms`, `/privacy`
- Separate pages with text. Also open in a dialog from the registration page.

## 404 and errors
- **404:** "Page not found" and a "Home" button ("Go to drive" for signed-in users).
- **General error:** "Something went wrong", "Reload page" and "Home" buttons.
- **No connection to the server:** "No connection" banner, retries automatically.

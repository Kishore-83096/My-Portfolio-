# Parrot - Unified Communication Platform

Parrot is a web communication platform with account management, saved contacts, real-time messaging, file attachments, and browser-based end-to-end encrypted messaging.

**Try the app:** [https://parrot-react.onrender.com/](https://parrot-react.onrender.com/)

## Table Of Contents

- [About The App](#about-the-app)
- [Services](#services)
- [Architecture](#architecture)
- [Screenshots](#screenshots)
- [Main Features](#main-features)
- [Security Model](#security-model)
- [Frontend User Manual](#frontend-user-manual)
- [Create An Account](#1-create-an-account)
- [First Login And First Device Setup](#2-first-login-and-first-device-setup)
- [Login On Another Device](#3-login-on-another-device)
- [Recovery Key Updates](#4-recovery-key-updates)
- [Linked Devices](#5-linked-devices)
- [Contacts And Messaging](#6-contacts-and-messaging)
- [Profile And Account](#7-profile-and-account)

## About The App

Parrot is a private messaging app for people who want a simple place to manage contacts and chat without giving the server readable access to their conversations. A user can create an account, save contacts by account number, send messages and attachments, see delivery/read updates, manage linked devices, and recover encrypted messages on another trusted browser with a recovery key.

The app is built around the idea that Parrot should help deliver and sync messages, but should not be the place where plain conversation text is kept. Messages are locked in the browser before they leave the user's device. The Messenger service stores the locked message data so conversations can load again later, but it does not receive the private keys needed to read those messages.

User conversations are stored by the Messenger service as encrypted message records. The service can keep room history, message status, unread counts, replies, edits, deletes, device public keys, and recovery backup metadata. Message text is stored as encrypted content, not as readable chat text. Attachments are also encrypted before upload; the encrypted files are stored in Cloudinary, and Messenger keeps the approved upload records and encrypted file references.

Account details are stored separately by the Parent service. Parent keeps login identity, profile details, saved contacts, aliases, and block rules. Parent does not store message text, message private keys, linked-device private keys, default-device passwords, or recovery keys.

User secrets are handled with separate protections:

- Login passwords are saved as password hashes, not plain passwords.
- The default-device password is checked by Messenger, but Messenger stores only its password hash.
- Private encryption keys stay in the user's browser/device. Messenger stores public keys so other devices can send encrypted messages to that user.
- The recovery key is created and checked in the browser. Messenger stores only the encrypted recovery backup and the data needed to verify it, not the plain recovery key.
- Non-default devices can use a recovery key to unlock old messages, but the app does not store or display the plain recovery key on those devices.
- Cloudinary API secrets and internal service secrets stay on the backend and are loaded from environment variables, not committed to the repository or sent to the browser.

## Services

The project is split into three services:

| Project | Role | Tech Stack | Repository |
|---|---|---|---|
| [`React/`](React/) | Browser frontend for login, profile, contacts, chat, linked devices, and encrypted-message recovery. | React 19, TypeScript, Vite, Axios, React Router, Lucide icons, Tailwind CSS tooling, libsodium-wrappers | [Kishore-83096/parrot-react](https://github.com/Kishore-83096/parrot-react) |
| [`Parent/`](Parent/) | Account and contact service for login, profiles, JWT sessions, contact rules, blocking, and Messenger authorization. | Python 3.12, Flask, Flask-SQLAlchemy, Flask-Migrate, Alembic, Marshmallow, Flask-JWT-Extended, PostgreSQL/SQLite, Cloudinary, Gunicorn, Waitress, Docker | [Kishore-83096/Parent](https://github.com/Kishore-83096/Parent) |
| [`Messenger/`](Messenger/) | Messaging service for rooms, messages, E2EE device keys, encrypted file uploads, delivery state, and WebSockets. | Python 3.12, Django, Django Channels, Redis, SQLite/PostgreSQL, Cloudinary, PyJWT, cryptography | [Kishore-83096/Parrot-messenger](https://github.com/Kishore-83096/Parrot-messenger) |

## Architecture

<p align="center">
  <img src="images/productarhietecture.png" alt="Parrot architecture diagram" width="1000">
</p>

## Screenshots

| App | Login |
|---|---|
| <img src="images/Parrot.png" alt="Parrot app" width="420" height="260"> | <img src="images/parrot%20login.png" alt="Parrot login page" width="420" height="260"> |

| Login Modal | Registration Modal |
|---|---|
| <img src="images/Parrotloginmodal.png" alt="Parrot login modal" width="420" height="260"> | <img src="images/Parrotregisterationmodal.png" alt="Parrot registration modal" width="420" height="260"> |

| Chat | Chat Room |
|---|---|
| <img src="images/Parrot%20chat.png" alt="Parrot chat" width="420" height="260"> | <img src="images/Parrotchatroom.png" alt="Parrot chat room" width="420" height="260"> |

## Main Features

- Parent account registration and login.
- Profile editing with profile-picture upload.
- Contact search by account number.
- Saved contacts with aliases, blocking, unblocking, and deletion.
- Direct messaging with replies, edits, deletes, delivery state, read state, and unread counts.
- File attachment support with encrypted upload flow.
- Presence and inbox events through WebSockets.
- End-to-end encrypted messages using local browser keys.
- Linked-device management with a signed default-device security model.
- Recovery key backup and verification for encrypted-message recovery.

## Security Model

- Parent access and refresh tokens protect Parent APIs.
- Parent issues short-lived Messenger JWTs for Messenger APIs and WebSockets.
- Messenger asks Parent to authorize sends based on contacts and block state.
- Message content is encrypted in the browser before it is sent to Messenger.
- Encrypted attachments are uploaded directly from React to Cloudinary only after Messenger authorizes the message and returns short-lived signed upload intents. Cloudinary secrets stay on Messenger.
- Messenger stores device public keys and encrypted message payloads; it does not receive message private keys.
- React queues outgoing messages in the browser and sends them to Messenger one at a time with unique client message ids, so fast sends keep first-come-first-serve order and duplicate retries stay safe.
- Realtime messaging uses both inbox and room websockets; the room socket is kept alive with pings, and the open conversation also accepts inbox message/status events as a fallback.
- React keeps an account-scoped UI cache for contacts, room list, selected conversation header data, and fetched conversation pages. The cache is refreshed by API responses and realtime events, and it is removed when that account logs out.
- Device management uses a separate local Ed25519 management key.
- The authenticated React app is mounted per account, so contacts, rooms, selected conversations, and runtime E2EE caches are discarded when a different account logs in on the same browser.
- Default-device actions are signed locally and verified by Messenger. Making a device default also requires the default-device password; Messenger stores only its password hash and rate-limits failed verification.
- Only the default linked device can update the recovery key. The default device can make another active device default with the password, and a non-default device can make only itself default with the password.
- Non-default devices can enter a recovery key for verification/recovery, but the app does not store or display the plain recovery key on those devices.
- Non-default device logout removes that device's Messenger database row and clears local E2EE browser state. Default-device logout keeps the default device row and local E2EE state so the same trusted browser can be recognized again.

## Frontend User Manual

### 1. Create An Account

1. Open the Parrot web app.
2. Choose register.
3. Enter first name, last name, username, password, and confirm password.
4. After registration, login with the username and password.

The Parent service creates the account, account number, login tokens, and profile record.

### 2. First Login And First Device Setup

On the first browser/device for a new account:

1. The app creates a local encrypted-messaging device identity.
2. The device is registered with Messenger.
3. Because no default device exists yet, the app opens linked-device setup.
4. Create the default-device password and make this device default.
5. Create a recovery key.
6. Save the recovery key somewhere outside the browser.

The first default device is important because it becomes the only device allowed to manage recovery-key updates. The default-device password is required later if default permission is moved to another trusted browser.

### 3. Login On Another Device

When the same account logs in on another browser/device:

1. The app creates/registers a new device identity.
2. If a recovery backup exists, the app asks for the recovery key.
3. The user gets 5 attempts.
4. If all attempts fail, the app logs out and clears local E2EE browser state.
5. If recovery succeeds, old encrypted messages can decrypt on that device.
6. The device remains non-default unless the user enters the default-device password to make that current device default, or the current default browser makes it default.
7. When the non-default device logs out, its Messenger device row and local E2EE browser state are deleted.

Non-default devices can read messages after recovery, but cannot view/update the recovery key or manage other devices.

### 4. Recovery Key Updates

When the default device updates the recovery key:

1. Messenger broadcasts a `recovery.key_updated` event to the user.
2. Other linked non-default devices fetch the encrypted backup.
3. They open a modal asking for the current recovery key.
4. The key is verified locally against the encrypted backup.
5. The key is discarded and only a local acknowledgement marker is stored.

This prevents non-default devices from showing or retaining the plain recovery key.

### 5. Linked Devices

Open the account menu and choose linked devices. The device tab separates the current default device at the top from active devices below it. On the default browser, it shows all active linked devices. On a non-default browser, it shows the current browser and the current default browser.

Default device can:

- make another active device default after password verification
- update the default-device password after current-password verification
- remove another non-default device
- update the recovery key
- view the locally saved recovery key if it exists on that browser
- log out without removing its local E2EE identity or Messenger default-device row

Any current device can:

- log itself out
- make itself default after password verification
- verify the recovery key when prompted

Non-default devices cannot:

- view the recovery key
- update the recovery key
- revoke other devices
- make another device default

Non-default logout is a cleanup action: React clears that browser's E2EE local storage and Messenger deletes that device row from the database. This keeps the linked-device list clean when the same user logs in again from the same browser.

### 6. Contacts And Messaging

1. Open the contacts tab.
2. Search a Parrot account number.
3. Save the contact with an alias.
4. Open a contact or chat from the left panel.
5. Send text or attachments.
6. Send repeatedly if needed; React queues outgoing messages and sends them first-come-first-serve.
7. Use the room header for contact details, blocking, unblocking, release of blocked messages, or contact deletion.

Messages and attachment payloads are encrypted in the frontend before being sent through Messenger. Attachments use signed upload intents: Messenger authorizes the recipient first, React uploads encrypted blobs directly to Cloudinary, then Messenger verifies the Cloudinary response before the message can reference the completed upload.

Contacts, chat rooms, conversation headers, and already fetched message pages open from the local account cache first, then refresh from the APIs and WebSockets. This makes mobile and desktop views feel instant after navigation or reload while still keeping Messenger as the source of truth. Logging out clears that account's messenger UI cache from the browser.

### 7. Profile And Account

The account menu supports:

- profile viewing and editing
- profile-picture upload
- password change
- account deletion
- linked-device and recovery-key management
- logout

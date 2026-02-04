# ANANTA

Real-time chat and identity platform built as independent, production-style microservices with a modern React UI. This README explains the application concept, the APIs, views, and model design for each backend service, plus how everything connects.

## Live

```
https://anantafrontend.vercel.app/
```

## Product Theme

ANANTA is a clean, minimal, contact-first chat experience where identity and communication stay modular. It starts with a strong auth identity, then layers profile depth, contact graph, room access control, and message delivery as separate services. This mirrors a production system where each domain can scale or evolve without risking the others.

## Why Microservices Here

- `AUTH_MS` isolates credentials and JWT security.
- `PROFILE_MS` focuses on personal details and user data without auth logic.
- `acac_ms` owns invites, relationships, and room access.
- `message_ms` specializes in room/message persistence and real-time broadcast.

This keeps ownership clear, failure domains smaller, and scaling more predictable.

## At A Glance

| Component | Path | Purpose |
| --- | --- | --- |
| Frontend | `ANANTA_FRONTEND/frontend` | React + Vite UI for the platform. |
| Auth Service | `AUTH_MS` | User registration/login, JWT issuance, and auth validation. |
| Profile Service | `PROFILE_MS` | User profile, address, and card management (auth delegated to Auth Service). |
| Contacts and Access Service | `CHAT_SERVICE/acac_ms` | Chat contact graph: profiles, invites, contacts, and room access checks. |
| Message Service | `CHAT_SERVICE/message_ms` | Chat rooms and message storage with real-time delivery hooks. |

## Architecture Snapshot

```
Frontend (React/Vite)
   |
   |---> AUTH_MS (JWT, user identities)
   |---> PROFILE_MS (profile, address, cards)
   |---> CHAT_SERVICE / acac_ms (contacts, invites, room access)
   |---> CHAT_SERVICE / message_ms (rooms, messages, realtime)
```

## AUTH_MS (Authentication Microservice)

### Purpose
Handles user registration, login, JWT issuance, and account management operations like avatar and password resets.

### Core Models
`apps/accounts/models.py`

| Model | Key Fields | Notes |
| --- | --- | --- |
| `User` | `email`, `username`, `password`, `person_id`, `avatar` | Custom user using email as login, auto-generates 6-digit `person_id`, Cloudinary avatar storage. |

### Key Serializers
`apps/accounts/serializers.py`

| Serializer | Role |
| --- | --- |
| `RegisterSerializer` | Creates user, hashes password. |
| `LoginSerializer` | Authenticates user and returns JWT access and refresh. |
| `UserMeSerializer` | Current user profile including avatar URL. |
| `AvatarUploadSerializer` | Validates avatar size, updates Cloudinary asset. |
| `ForgotPasswordSerializer` | Accepts email for password reset request. |
| `ResetPasswordSerializer` | Validates reset token and new password. |

### Views and API Endpoints
`apps/accounts/views.py` + `apps/accounts/urls.py`

| Endpoint | Method | View | Description |
| --- | --- | --- | --- |
| `/api/auth/protected/` | GET | `ProtectedView` | JWT-protected test endpoint. |
| `/api/auth/register/` | POST | `RegisterView` | Register a new user. |
| `/api/auth/login/` | POST | `LoginView` | Login and receive JWT tokens. |
| `/api/auth/me/` | GET | `UserMeAPIView` | Get logged-in user info. |
| `/api/auth/upload-avatar/` | POST | `AvatarUploadView` | Upload/update avatar. |
| `/api/auth/remove-avatar/` | DELETE | `AvatarRemoveView` | Remove avatar from Cloudinary and user profile. |
| `/api/auth/logout/` | POST | `LogoutView` | Blacklist refresh token. |
| `/api/auth/token/refresh/` | POST | `TokenRefreshView` | Refresh access token. |
| `/api/auth/forgot-password/` | POST | `ForgotPasswordAPIView` | Initiate reset (prints link in logs). |
| `/api/auth/reset-password/` | POST | `ResetPasswordAPIView` | Reset password with token. |

### Authentication Flow
- Access and refresh JWT tokens are issued on login.
- Protected endpoints require `Authorization: Bearer <access_token>`.
- Logout blacklists refresh tokens.

Details: `AUTH_MS/Readme.md`

## PROFILE_MS (Profile Microservice)

### Purpose
Manages user profile data, addresses, and cards. It delegates authentication to `AUTH_MS` and never stores passwords.

### Core Models
`apps/profiles/models.py`

| Model | Key Fields | Notes |
| --- | --- | --- |
| `UserProfile` | `person_id`, `email`, `first_name`, `last_name`, `primary_phone`, `alternate_phone`, `gender`, `date_of_birth` | `person_id` and `email` come from Auth MS. |
| `Address` | `address_type`, `line1`, `line2`, `country`, `state`, `city`, `zip_code`, `phone_number`, `is_default` | Unique per `user` and `address_type`. |
| `Card` | `card_type`, `card_brand`, `card_number`, `card_holder_name`, `expiry_month`, `expiry_year`, `is_default` | Enforces max 4 cards per user and valid brand by type. |

### Key Serializers
`apps/profiles/serializers.py`

| Serializer | Role |
| --- | --- |
| `UserProfileSerializer` | Read/update profile fields, `person_id` and `email` are read-only. |
| `AddressSerializer` | Full CRUD on address fields. |
| `CardSerializer` | Full CRUD on card fields with brand validation. |

### Views and API Endpoints
`apps/profiles/views.py` + `apps/profiles/urls.py`

| Endpoint | Method | View | Description |
| --- | --- | --- | --- |
| `/profiles/health/` | GET | `HealthCheckView` | Service health check. |
| `/profiles/test-auth/` | GET | `TestAuthView` | Validates Auth MS token. |
| `/profiles/profile/` | GET | `UserProfileView` | Get profile, auto-creates on first access. |
| `/profiles/profile/` | PUT | `UserProfileView` | Update profile fields. |
| `/profiles/addresses/` | GET | `AddressListCreateView` | List addresses. |
| `/profiles/addresses/` | POST | `AddressListCreateView` | Create address. |
| `/profiles/addresses/<id>/` | GET | `AddressDetailView` | Read one address. |
| `/profiles/addresses/<id>/` | PUT | `AddressDetailView` | Update one address. |
| `/profiles/addresses/<id>/` | DELETE | `AddressDetailView` | Delete one address. |
| `/profiles/cards/` | GET | `CardListCreateView` | List cards. |
| `/profiles/cards/` | POST | `CardListCreateView` | Create card (max 4). |
| `/profiles/cards/<id>/` | GET | `CardDetailView` | Read one card. |
| `/profiles/cards/<id>/` | PUT | `CardDetailView` | Update one card. |
| `/profiles/cards/<id>/` | DELETE | `CardDetailView` | Delete one card. |

### Authentication Flow
- All endpoints validate the JWT by calling Auth MS.
- The token is passed in `Authorization: Bearer <access_token>`.

Details: `PROFILE_MS/Readme.md`

## CHAT_SERVICE / acac_ms (Contacts and Access Control)

### Purpose
Manages chat-related identities, user invites, contacts, and access checks for chat rooms.

### Core Models
`apps/mainapp/models.py`

| Model | Key Fields | Notes |
| --- | --- | --- |
| `UserProfile` | `person_id`, `username`, `email`, `bio`, `status`, `avatar` | Local chat profile synced from Auth MS. |
| `UserInvite` | `sender`, `recipient`, `accepted`, `created_at` | `accepted` can be null for pending. |
| `Contact` | `user1`, `user2`, `room_id`, `created_at` | Unique pair, `room_id` is a UUID. |

### Key Serializers
`apps/mainapp/serializers.py`

| Serializer | Role |
| --- | --- |
| `UserProfileSerializer` | Profile read/write fields. |
| `UserProfileSearchSerializer` | Minimal user search payload. |
| `UserInviteSerializer` | Includes sender and recipient metadata. |
| `ContactSerializer` | Includes both contact participants and room id. |

### Views and API Endpoints
`apps/mainapp/views.py` + `apps/mainapp/urls.py`

| Endpoint | Method | View | Description |
| --- | --- | --- | --- |
| `/acac/health/` | GET | `HealthCheckAPIView` | Service health check. |
| `/acac/test-auth/` | GET | `AuthTestAPIView` | Validates Auth MS token. |
| `/acac/profile/` | GET | `MeProfileView` | Get current chat profile. |
| `/acac/profile/` | PATCH | `MeProfileView` | Update profile fields. |
| `/acac/search/` | POST | `UserSearchByPersonIDView` | Search by `person_id`. |
| `/acac/send-invite/` | POST | `SendInviteView` | Send invite to another user. |
| `/acac/received-invite/` | GET | `ReceivedInvitesView` | List pending invites. |
| `/acac/accept-invite/` | POST | `AcceptInviteView` | Accept or reject invite. |
| `/acac/rejected-invites/` | GET | `SentRejectedInvitesView` | List sent invites that were rejected. |
| `/acac/rejected-invite-delete/` | POST | `DeleteUserInviteView` | Delete a rejected invite. |
| `/acac/contacts/` | GET | `ListContactsView` | List all accepted contacts. |
| `/acac/check-room-access/` | POST | `CheckRoomAccessAPIView` | Validate user access to a room. |

### Authentication Flow
- Uses Auth MS authentication class for protected views.
- Auto-creates local chat profile on first access from Auth MS identity.

## CHAT_SERVICE / message_ms (Message and Realtime Delivery)

### Purpose
Stores chat rooms and messages, validates room access via contacts service, and broadcasts messages using Channels.

### Core Models
`apps/chat_messages/models.py`

| Model | Key Fields | Notes |
| --- | --- | --- |
| `Room` | `id`, `name`, `created_at` | `id` is UUID, `name` optional for groups. |
| `Message` | `id`, `room`, `sender_person_id`, `content`, `reply_to`, `is_seen`, `is_delivered` | UUID messages with reply threading. |

### Views and API Endpoints
`apps/chat_messages/views.py` + `apps/chat_messages/urls.py`

| Endpoint | Method | View | Description |
| --- | --- | --- | --- |
| `/messages/` | GET | `health_check` | Service status. |
| `/messages/test-auth/` | GET | `TestAuthView` | Validates Auth MS token. |
| `/messages/test-chat-profile/` | GET | `TestChatProfileView` | Retrieves chat profile via contacts service. |
| `/messages/check-chat-access/` | POST | `CheckChatAccessProxyAPIView` | Proxies room access to contacts service. |
| `/messages/send-message/` | POST | `SendMessageAPIView` | Sends message, broadcasts via Channels. |
| `/messages/fetch-messages/` | POST | `FetchMessagesAPIView` | Fetches messages for a room. |

### Realtime Delivery
- Uses Django Channels with Redis for group broadcasting.
- Sends to `room_<room_id>` channel group on message creation.

## Frontend

- React + Vite application
- Connects to the backend microservices for auth, profile, and chat

Details: `ANANTA_FRONTEND/frontend/README.md`

## Tech Stack (High Level)

- Backend: Django 6.x, Django REST Framework
- Auth: JWT (SimpleJWT)
- Realtime: Django Channels + Redis (message service)
- Storage: PostgreSQL (for all the services with different databases)
- Media: Cloudinary (avatar and display pictures services)
- Frontend: React + Vite

## Notes

- Each service is self-contained and can be run independently.
- Environment variables are required for secrets, database, and service URLs.
- See each service README for specific setup and environment configuration.

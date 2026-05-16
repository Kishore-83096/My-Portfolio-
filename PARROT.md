# <img src="images/favicon.svg" width="40" height="40" alt="Parrot" style="margin-right: 10px; vertical-align: middle;" /> [Parrot - Unified Communication Platform](https://parrot-react.onrender.com/)

> **Real-time messaging with contact management** 
>
> A modern **multi-service messaging platform** for secure authentication, user profiles, contact management, and real-time instant messaging across web. Built with microservices architecture separating user management from real-time communication.

---

## 📸 Screenshots

| | | |
|:-:|:-:|:-:|
| ![Home Screen](images/Parrot.png)<br>**Landing Page** | ![Login](images/Parrotlogin.png)<br>**User Page** | ![Chat Interface](images/Parrotchat.png)<br>**Chat Interface** |
| ![Chat Room](images/Parrotchatroom.png)<br>**Chat Room** | ![Registration](images/Parrotregisterationmodal.png)<br>**Registration** | ![Login Modal](images/Parrotloginmodal.png)<br>**Login Modal** |

---

## 🔗 Repositories

| Service | Repository |
|---------|-----------|
| 🎨 **React Frontend** | [parrot-react](https://github.com/Kishore-83096/parrot-react) |
| 👤 **Parent Service** | [parrot-parent](https://github.com/Kishore-83096/Parent) |
| 💬 **Messenger Service** | [parrot-messenger](https://github.com/Kishore-83096/Parrot-messenger) |

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture](#architecture)
- [Services](#-services)
- [Technology Stack](#-technology-stack)
- [Installation & Setup](#-installation--setup)
- [Running the Services](#running-the-services)
- [Deployment](#-deployment)
- [Security](#-security)
- [Contributing](#-contributing)

---

## 🎯 Project Overview

**Parrot** is a distributed microservices messaging platform designed to handle:

- **User Management**: Registration, authentication, profile management
- **Contact Management**: Search, save, and organize contacts with blocking features
- **Real-Time Messaging**: WebSocket-based instant messaging with status tracking
- **Message Features**: Text, attachments (images, videos, audio, documents), replies
- **Security**: JWT-based authentication, service-to-service authorization

### Key Features

✅ **Secure registration and JWT-based authentication**  
✅ **Customizable user profiles with picture upload**  
✅ **Contact discovery and management with blocking**  
✅ **Real-time messaging with WebSocket support**  
✅ **Message attachments (images, videos, audio, documents)**  
✅ **Message replies and threading**  
✅ **Read receipts and delivery status**  
✅ **Typing indicators**  
✅ **Direct and group chat support**  
✅ **Block detection and delivery blocking**  
✅ **Message history and persistence**  
✅ **Docker support for all services**  
✅ **Environment-based configuration**

---

<a id="architecture"></a>

## 🏗️ Architecture

### System Overview

Parrot uses a **microservices architecture** with clear separation between user management and real-time communication:

```
Frontend (React)
    ↓
Parent Service (User/Auth/Contacts)
    ↓ (internal auth)
Messenger Service (Real-time Chat)
    ↓
PostgreSQL + Redis
```

### Key Components

| Layer | Component | Purpose |
|-------|-----------|---------|
| **Frontend** | React + Vite | Chat UI, user authentication, contact management |
| **User Service** | Parent (Flask) | Authentication, profiles, contacts, authorization |
| **Messaging Service** | Messenger (Django) | Real-time messaging, WebSocket, message persistence |
| **Real-time** | Redis + Channels | WebSocket pub/sub, session management |
| **Storage** | PostgreSQL | Users, contacts, messages, room data |
| **Files** | Cloudinary | Image and file hosting |

### Detailed Architecture Diagram

![Parrot System Architecture](images/productarhietecture.png)

---

## 🔧 Services

### 1. **Parent Service** - User Management & Authentication

**Framework**: Flask  
**Port**: 5000  
**Purpose**: User registration, authentication, profile management, and contact management

#### Core Features

| Feature | Endpoints |
|---------|-----------|
| **Authentication** | `POST /auth/register`, `POST /auth/login`, `POST /auth/refresh`, `POST /auth/change-password` |
| **Profiles** | `GET /profile`, `PUT /profile` |
| **Contacts** | `GET /contacts`, `GET /contacts/<account>`, `POST /contacts`, `PATCH /contacts/alias`, `DELETE /contacts` |
| **Contact Control** | `POST /contacts/block`, `POST /contacts/unblock` |
| **Search** | `POST /users/search` |
| **Account** | `DELETE /account`, `POST /messaging/token` |
| **Internal Auth** | `POST /internal/messaging/authorize` |

#### Database Models

- **User**: Credentials, account number, premium status
- **Profile**: Personal info, phone, addresses, card details, profile picture
- **Contact**: Saved contacts with aliases and block status

### 2. **Messenger Service** - Real-Time Messaging

**Framework**: Django + Django Channels  
**Port**: 8000  
**Purpose**: Real-time message delivery, WebSocket management, and message persistence

#### Core Features

| Feature | Endpoints/Events |
|---------|---------|
| **Rooms** | `GET /rooms`, `GET /rooms/<id>/messages` |
| **Messages** | `POST /messages/send`, `GET /rooms/<id>/messages` |
| **Status Tracking** | `POST /rooms/<id>/delivered`, `POST /rooms/<id>/read` |
| **Authorization** | `POST /messages/authorize`, `POST /internal/messaging/authorize` |
| **Block Management** | `POST /rooms/<id>/blocked-messages/release` |
| **WebSocket** | `ws://localhost:8000/ws/chat/<room_id>/` |
| **Health** | `GET /health` |

#### WebSocket Events

- **typing.started** - User started typing
- **typing.stopped** - User stopped typing
- **connection.accepted** - WebSocket connection established
- **room.event** - Room-level events (typing, participant updates)

#### Database Models

- **Room**: Direct (1-to-1) and group chats
- **RoomParticipant**: Members with roles (member, admin)
- **Message**: Text, attachments, status (sent/delivered/read), replies
- **MessageAttachment**: Files with metadata (type, size, dimensions, thumbnails)

#### Message Features

- **Status Tracking**: sent → delivered → read
- **Replies**: Messages can reply to other messages
- **Attachments**: Images, videos, audio, documents
- **Blocking**: Delivery blocking for blocked contacts
- **Timestamp**: Creation and edit times

### 3. **React Frontend** - User Interface

**Framework**: React 19 + TypeScript  
**Build Tool**: Vite  
**Styling**: Tailwind CSS + shadcn/ui  
**Port**: 5173 (dev) / 80 (prod)

#### Features

**Authentication**
- Registration with username/password
- Login
- Session persistence with JWT tokens

**Profile Management**
- View and edit profile
- Upload profile pictures
- Update personal information

**Contact Management**
- Search users by account number
- Save contacts
- Update contact aliases
- Block/unblock contacts
- View contact list

**Messaging**
- Real-time chat with WebSocket
- Direct 1-to-1 messaging
- Group chats
- Message attachments (upload and preview)
- Message replies with context
- Typing indicators
- Read receipts (single check) and delivery status (double check)
- Message history pagination
- Attachment gallery with tabs (All, Images, PDF, Other)
- Download attachments
- Message deletion and editing

**UI Components**
- Modern, responsive design
- Dark/light theme support
- Real-time notifications
- Toast messages for feedback
- Modal dialogs

---

## 📚 Technology Stack

### Backend Services

**Parent Service (Flask)**
- Flask 3.1.3, Flask-RESTful 0.3.10
- PostgreSQL with SQLAlchemy 2.0
- JWT authentication (Flask-JWT-Extended 4.7.1)
- Cloudinary for image uploads
- Gunicorn for production

**Messenger Service (Django)**
- Django 5.2+, Django REST Framework 3.17+
- Django Channels 4.3+ for WebSocket
- PostgreSQL + Redis for messaging
- Daphne ASGI server
- Cloudinary for media uploads

### Frontend (React)

- React 19.2.5 with TypeScript
- Vite 8.0.10 (build tool)
- Tailwind CSS 4.2.4
- shadcn/ui + Radix UI
- React Router 7.14.2
- Axios 1.16.0
- React Hook Form 7.75.0
- TanStack Query 5.100.9
- Framer Motion 12.38.0

### Infrastructure

- **Database**: PostgreSQL 13+
- **Cache/Queue**: Redis 7+
- **Containerization**: Docker & Docker Compose
- **File Storage**: Cloudinary CDN

---

## 🔍 Data Flow

### Authentication Flow
1. User registers with username/password
2. Parent service generates unique account number (7XXXXXXXXX format)
3. Password hashed with Argon2, stored securely
4. Login returns JWT access token (15min) and refresh token (7 days)
5. Refresh endpoint generates new access token without re-entering credentials

### Messaging Flow
1. User requests messaging token from Parent service
2. Parent returns token with user ID and account number
3. User authenticates WebSocket connection with token
4. Messenger service validates token and authorizes room access
5. Real-time messages broadcast to room participants via WebSocket
6. Messages persist to database with status tracking
7. Delivery and read status updated as messages are delivered/read

### Contact Flow
1. User searches for contact by account number (e.g., "7123456789")
2. Parent service returns user info if found
3. User saves contact with custom alias
4. Contact appears in contact list
5. User can block/unblock contacts
6. Blocked status affects message delivery authorization

---

## 🚀 Installation & Setup

### Prerequisites

- **Python 3.9+** - For Flask and Django services
- **Node.js 18+** - For React frontend
- **PostgreSQL 13+** - Main database (SQLite for development)
- **Redis 7+** - For real-time messaging
- **Docker & Docker Compose** (optional) - For containerized deployment

### Parent Service Setup

```bash
cd PARROT/Parent

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your settings (DATABASE_URL, SECRET_KEY, JWT_SECRET_KEY, CLOUDINARY_*)

# Run database migrations
flask db upgrade

# Start development server
python run.py
# Server runs on http://localhost:5000
```

### Messenger Service Setup

```bash
cd PARROT/Messenger

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your settings (DATABASE_URL, REDIS_URL, SECRET_KEY, PARENT_SERVICE_URL)

# Run database migrations
python manage.py migrate

# Start development server
python manage.py runserver 0.0.0.0:8000
# Server runs on http://localhost:8000
```

### React Frontend Setup

```bash
cd PARROT/React

# Install dependencies
npm install

# Configure environment
# Create .env with your API endpoints
# VITE_PARENT_API=http://localhost:5000
# VITE_MESSENGER_API=http://localhost:8000
# VITE_MESSENGER_WS=ws://localhost:8000

# Start development server
npm run dev
# Frontend available at http://localhost:5173

# Build for production
npm run build
# Output in dist/ folder
```

---

## ▶️ Running the Services

### Development - Individual Services

Open three terminals and run each service:

```bash
# Terminal 1: Parent Service (Port 5000)
cd PARROT/Parent
source venv/bin/activate
python run.py

# Terminal 2: Messenger Service (Port 8000)
cd PARROT/Messenger
source venv/bin/activate
python manage.py runserver 0.0.0.0:8000

# Terminal 3: React Frontend (Port 5173)
cd PARROT/React
npm run dev
```

**Access the application**: http://localhost:5173

### Production - Docker Compose

```bash
cd PARROT

# Build and start all services
docker-compose up --build

# Access services:
# Frontend: http://localhost
# Parent API: http://localhost:5000
# Messenger API: http://localhost:8000
```

---

## 🐳 Deployment

### Docker Setup

All services include Dockerfiles. Build individual services:

```bash
# Parent Service
cd PARROT/Parent
docker build -t parrot-parent .
docker run -p 5000:5000 --env-file .env parrot-parent

# Messenger Service
cd PARROT/Messenger
docker build -t parrot-messenger .
docker run -p 8000:8000 --env-file .env parrot-messenger

# React Frontend
cd PARROT/React
docker build -t parrot-react .
docker run -p 80:80 parrot-react
```

### Environment Variables

**Parent Service (.env)**
```
FLASK_ENV=production
DATABASE_URL=postgresql://user:password@db:5432/parent
SECRET_KEY=your-secret-key
JWT_SECRET_KEY=your-jwt-secret
INTERNAL_SERVICE_TOKEN=service-token
CLOUDINARY_CLOUD_NAME=your-cloud
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-secret
```

**Messenger Service (.env)**
```
DEBUG=False
DJANGO_ENV=production
DATABASE_URL=postgresql://user:password@db:5432/messenger
REDIS_URL=redis://redis:6379/0
SECRET_KEY=your-secret-key
PARENT_SERVICE_URL=http://parent:5000
INTERNAL_SERVICE_TOKEN=service-token
ALLOWED_HOSTS=yourdomain.com
```

**React Frontend (.env.production)**
```
VITE_PARENT_API=https://yourdomain.com/api/parent
VITE_MESSENGER_API=https://yourdomain.com/api/messenger
VITE_MESSENGER_WS=wss://yourdomain.com/ws
```

---

## 🔐 Security

### Authentication & Authorization

- **JWT Tokens**: 15-minute access tokens + 7-day refresh tokens
- **Password Hashing**: Argon2 with salt
- **WebSocket Auth**: JWT token validation on connection
- **Service Authorization**: X-Internal-Service-Token for Parent ↔ Messenger communication
- **Contact Blocking**: Prevents message delivery between blocked users
- **Delivery Blocking**: Blocked messages tracked but not delivered

### Best Practices

🔒 **Secrets in Environment** - Never hardcode JWT secrets or API keys  
🔐 **HTTPS in Production** - Always use SSL/TLS for data transmission  
🔑 **Strong Tokens** - Use long, random secrets for JWT signing  
⚙️ **CORS Configuration** - Restrict to trusted origins  
✔️ **Input Validation** - Validate all user inputs server-side  
📋 **Rate Limiting** - Protect endpoints from abuse  
🔐 **Database Security** - Use SSL for database connections  
📊 **Security Logging** - Monitor authentication and authorization events

### Blocking Mechanism

1. User blocks a contact via Parent service
2. Contact status set to `blocked=True` in database
3. When blocked user tries to message: Parent service checks block status
4. If blocked, message marked with `delivery_blocked=True`
5. Messenger service receives delivery-blocked flag
6. Message stored but not delivered to recipient
7. Admin can release blocked messages via `/rooms/<id>/blocked-messages/release/`

---

## 📋 Contributing

We welcome contributions! Here's how to help:

1. **Fork** the repository
2. **Create a feature branch** (`git checkout -b feature/amazing-feature`)
3. **Commit changes** (`git commit -m 'Add amazing feature'`)
4. **Push to branch** (`git push origin feature/amazing-feature`)
5. **Open a Pull Request** with description of changes

### Development Guidelines

- Follow existing code style and patterns
- Write meaningful commit messages
- Test changes before submitting PR
- Update documentation as needed
- Respect existing architecture decisions

---

## 📄 License

This project is open source and available under the MIT License.

---

## 🙋 Support

For questions, issues, or feature requests:

- **Open an Issue** on the respective GitHub repository
- **Check Documentation** for existing solutions
- **Review Code** in each service's README

---

## 🔗 References

- [Flask Documentation](https://flask.palletsprojects.com/)
- [Django Documentation](https://docs.djangoproject.com/)
- [React Documentation](https://react.dev/)
- [Django Channels](https://channels.readthedocs.io/)
- [WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)

---

**Parrot - Unified Communication Platform**  
Built with Flask, Django, React, PostgreSQL, and Redis

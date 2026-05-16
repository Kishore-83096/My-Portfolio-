# <img src="images/favicon.svg" width="40" height="40" alt="Parrot" style="margin-right: 10px; vertical-align: middle;" /> [Parrot - Unified Communication Platform](https://parrot-react.onrender.com/)

> A real-time communication platform with user authentication, contact management, blocking, file attachments, and WebSocket-based messaging.

---

## 📸 Screenshots

| | | |
|:-:|:-:|:-:|
| ![Home Screen](images/Parrot.png)<br>**Landing Page** | ![Login](images/Parrotlogin.png)<br>**User Page** | ![Chat Interface](images/Parrotchat.png)<br>**Chat Interface** |
| ![Chat Room](images/Parrotchatroom.png)<br>**Chat Room** | ![Registration](images/Parrotregisterationmodal.png)<br>**Registration** | ![Login Modal](images/Parrotloginmodal.png)<br>**Login Modal** |

---

## 🔗 Project Links

| Type | Link |
|------|------|
| 🌐 **Live Application** | [Parrot Web App](https://parrot-react.onrender.com/) |
| 🎨 **React Frontend** | [parrot-react](https://github.com/Kishore-83096/parrot-react) |
| 👤 **Parent Service** | [parrot-parent](https://github.com/Kishore-83096/Parent) |
| 💬 **Messenger Service** | [parrot-messenger](https://github.com/Kishore-83096/Parrot-messenger) |

---

## 🎯 Overview

**Parrot** is a unified communication platform built using a microservices approach. The project separates user-related operations from real-time messaging so each service has a clear responsibility.

The platform includes:

- A **React frontend** for the user interface
- A **Parent service** for authentication, profiles, contacts, and blocking
- A **Messenger service** for chat rooms, messages, attachments, and WebSocket communication

---

## ✨ Main Features

- User registration and login
- Profile management with profile picture support
- Contact search and saved contacts
- Contact aliases, blocking, and unblocking
- Direct and group messaging
- Real-time chat using WebSockets
- Typing indicators, delivery status, and read receipts
- Message replies, editing, deletion, and history
- File attachments for images, videos, audio, documents, and PDFs
- Attachment preview, gallery view, and download support

---

<a id="architecture"></a>

## 🏗️ Architecture

```text
React Frontend
      ↓
Parent Service - Flask
Authentication, profiles, contacts, blocking, messaging authorization
      ↓
Messenger Service - Django + Channels
Rooms, messages, attachments, WebSockets, delivery/read status
      ↓
PostgreSQL + Redis + Cloudinary
```

### Architecture Diagram

![Parrot System Architecture](images/productarhietecture.png)

---

## 🧩 Services

| Service | Responsibility |
|---------|----------------|
| **React Frontend** | Handles the user interface for authentication, contacts, profile, and chat screens. |
| **Parent Service** | Manages users, login, profiles, contacts, blocking, and messaging authorization. |
| **Messenger Service** | Manages chat rooms, messages, attachments, WebSocket events, and message status updates. |

---

## 🛠️ Tech Stack

| Area | Technologies |
|------|--------------|
| **Frontend** | React, TypeScript, Vite, Tailwind CSS |
| **Parent Service** | Flask, SQLAlchemy, JWT |
| **Messenger Service** | Django, Django REST Framework, Django Channels |
| **Database** | PostgreSQL |
| **Real-time Layer** | Redis |
| **File Storage** | Cloudinary |

---

## 🔐 Security Highlights

- JWT-based authentication
- Password hashing
- Service-to-service authorization
- WebSocket authentication
- Contact blocking support
- Environment-based secrets and configuration

---

## 📌 Note

This README gives a high-level overview of the complete Parrot platform. For detailed setup instructions, API endpoints, environment variables, and service-specific implementation details, please refer to the individual repositories linked above.

---

**Parrot - Unified Communication Platform**  
Built with React, Flask, Django, PostgreSQL, Redis, and Cloudinary.

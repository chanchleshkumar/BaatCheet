# Talk-A-Tive - Project Architecture Documentation

## Table of Contents
1. [System Overview](#system-overview)
2. [High-Level Architecture](#high-level-architecture)
3. [Frontend Architecture](#frontend-architecture)
4. [Backend Architecture](#backend-architecture)
5. [Database Architecture](#database-architecture)
6. [Real-time Architecture](#real-time-architecture)
7. [API Architecture](#api-architecture)
8. [Data Flow Diagrams](#data-flow-diagrams)
9. [Component Structure](#component-structure)
10. [Deployment Architecture](#deployment-architecture)

---

## System Overview

**Talk-A-Tive** is a full-stack real-time chat application built with the MERN stack.

### Technology Stack

```
Frontend:  React.js + Chakra UI + Socket.io-client
Backend:   Node.js + Express.js + Socket.io
Database:  MongoDB + Mongoose
Auth:      JWT + bcrypt
Real-time: WebSockets (Socket.io)
```

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              React Frontend Application              │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │   │
│  │  │   Homepage   │  │   Chatpage   │  │  Context  │ │   │
│  │  │  (Auth UI)   │  │  (Main App)  │  │  Provider │ │   │
│  │  └──────────────┘  └──────────────┘  └──────────┘ │   │
│  │                                                       │   │
│  │  ┌──────────────────────────────────────────────┐   │   │
│  │  │         Socket.io Client (WebSocket)         │   │   │
│  │  └──────────────────────────────────────────────┘   │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↕ HTTP/REST
                            ↕ WebSocket
┌─────────────────────────────────────────────────────────────┐
│                        SERVER LAYER                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Node.js + Express Server                │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │   │
│  │  │   Routes     │  │ Controllers  │  │Middleware │ │   │
│  │  │  /api/user   │  │   UserCtrl   │  │   Auth    │ │   │
│  │  │  /api/chat   │  │   ChatCtrl   │  │  Error    │ │   │
│  │  │ /api/message │  │ MessageCtrl │  │           │ │   │
│  │  └──────────────┘  └──────────────┘  └──────────┘ │   │
│  │                                                       │   │
│  │  ┌──────────────────────────────────────────────┐   │   │
│  │  │         Socket.io Server (WebSocket)        │   │   │
│  │  │  - Room Management                           │   │   │
│  │  │  - Real-time Events                         │   │   │
│  │  │  - Message Broadcasting                     │   │   │
│  │  └──────────────────────────────────────────────┘   │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↕ Mongoose ODM
┌─────────────────────────────────────────────────────────────┐
│                      DATABASE LAYER                         │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                    MongoDB                           │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────┐     │   │
│  │  │  Users    │  │  Chats   │  │   Messages   │     │   │
│  │  │ Collection│  │Collection│  │  Collection  │     │   │
│  │  └──────────┘  └──────────┘  └──────────────┘     │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## Frontend Architecture

### Component Hierarchy

```
App.js (Root)
│
├─── Homepage (Authentication Page)
│    ├─── Login Component
│    │    ├─── Form Inputs
│    │    └─── Submit Handler
│    │
│    └─── Signup Component
│         ├─── Form Inputs
│         └─── Submit Handler
│
└─── Chatpage (Main Application)
     │
     ├─── ChatProvider (Context API)
     │    ├─── user (Global State)
     │    ├─── selectedChat (Global State)
     │    ├─── chats (Global State)
     │    └─── notification (Global State)
     │
     ├─── SideDrawer Component
     │    ├─── User Search
     │    ├─── User List
     │    └─── Logout
     │
     ├─── MyChats Component
     │    ├─── Chat List
     │    └─── Chat Item
     │         └─── UserListItem
     │
     └─── Chatbox Component
          └─── SingleChat Component
               ├─── Chat Header
               │    ├─── Profile Modal (One-on-One)
               │    └─── Update Group Modal (Group Chat)
               │
               ├─── ScrollableChat Component
               │    └─── Message Items
               │         ├─── Sender Avatar
               │         └─── Message Content
               │
               └─── Message Input
                    ├─── Typing Indicator (Lottie Animation)
                    └─── Input Field
```

### Frontend Folder Structure

```
frontend/
├── public/
│   ├── index.html
│   └── favicon.ico
│
├── src/
│   ├── components/
│   │   ├── Authentication/
│   │   │   ├── Login.js
│   │   │   └── Signup.js
│   │   │
│   │   ├── miscellaneous/
│   │   │   ├── SideDrawer.js
│   │   │   ├── ProfileModal.js
│   │   │   ├── GroupChatModal.js
│   │   │   └── UpdateGroupChatModal.js
│   │   │
│   │   ├── userAvatar/
│   │   │   ├── UserListItem.js
│   │   │   └── UserBadgeItem.js
│   │   │
│   │   ├── Chatbox.js
│   │   ├── MyChats.js
│   │   ├── SingleChat.js
│   │   ├── ScrollableChat.js
│   │   └── ChatLoading.js
│   │
│   ├── Context/
│   │   └── ChatProvider.js (Global State Management)
│   │
│   ├── Pages/
│   │   ├── Homepage.js
│   │   └── Chatpage.js
│   │
│   ├── config/
│   │   └── ChatLogics.js (Helper Functions)
│   │
│   ├── animations/
│   │   └── typing.json (Lottie Animation)
│   │
│   ├── App.js (Router)
│   └── index.js (Entry Point)
│
└── package.json
```

### State Management Architecture

```
┌─────────────────────────────────────────────────────────┐
│              React Context API (Global State)          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ChatProvider Component                                 │
│  ├─── user: { _id, name, email, pic, token }          │
│  ├─── selectedChat: { _id, users, chatName, ... }     │
│  ├─── chats: [ { _id, users, latestMessage, ... } ]    │
│  └─── notification: [ { _id, content, chat, ... } ]    │
│                                                         │
│  Provides state to all child components                │
│  via ChatState() hook                                   │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│            Component Local State (useState)             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  SingleChat Component                                  │
│  ├─── messages: [ Message objects ]                   │
│  ├─── loading: boolean                                 │
│  ├─── newMessage: string                               │
│  ├─── socketConnected: boolean                         │
│  └─── istyping: boolean                                │
│                                                         │
│  Other Components                                       │
│  ├─── Form inputs                                      │
│  ├─── UI state (modals, dropdowns)                     │
│  └─── Loading states                                   │
└─────────────────────────────────────────────────────────┘
```

### Routing Architecture

```
App.js
│
├─── Route: "/" (exact)
│    └─── Homepage Component
│         ├─── Login Component
│         └─── Signup Component
│
└─── Route: "/chats"
     └─── Chatpage Component
          ├─── Protected Route (checks user in localStorage)
          ├─── SideDrawer
          ├─── MyChats
          └─── Chatbox
```

---

## Backend Architecture

### MVC Pattern Implementation

```
┌─────────────────────────────────────────────────────────────┐
│                      BACKEND STRUCTURE                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐      ┌──────────────┐                  │
│  │    ROUTES    │ ───→ │ CONTROLLERS  │                  │
│  │  (Endpoints) │      │ (Business    │                  │
│  └──────────────┘      │  Logic)      │                  │
│         │               └──────────────┘                  │
│         │                       │                          │
│         │                       ↓                          │
│         │              ┌──────────────┐                    │
│         │              │   MODELS     │                    │
│         │              │  (Database   │                    │
│         │              │   Schema)    │                    │
│         │              └──────────────┘                    │
│         │                       │                          │
│         │                       ↓                          │
│         │              ┌──────────────┐                    │
│         └────────────→│  MIDDLEWARE  │                    │
│                        │  (Auth,      │                    │
│                        │   Error)     │                    │
│                        └──────────────┘                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Backend Folder Structure

```
backend/
├── config/
│   ├── db.js (MongoDB Connection)
│   └── generateToken.js (JWT Token Generation)
│
├── controllers/
│   ├── userControllers.js
│   │   ├── registerUser()
│   │   ├── authUser()
│   │   └── allUsers()
│   │
│   ├── chatControllers.js
│   │   ├── accessChat()
│   │   ├── fetchChats()
│   │   ├── createGroupChat()
│   │   ├── renameGroup()
│   │   ├── addToGroup()
│   │   └── removeFromGroup()
│   │
│   └── messageControllers.js
│       ├── allMessages()
│       └── sendMessage()
│
├── middleware/
│   ├── authMiddleware.js
│   │   └── protect() (JWT Verification)
│   │
│   └── errorMiddleware.js
│       ├── notFound()
│       └── errorHandler()
│
├── models/
│   ├── userModel.js
│   ├── chatModel.js
│   └── messageModel.js
│
├── routes/
│   ├── userRoutes.js
│   ├── chatRoutes.js
│   └── messageRoutes.js
│
└── server.js (Entry Point + Socket.io Setup)
```

### Request Flow Architecture

```
HTTP Request
    │
    ↓
┌─────────────────┐
│  Express Server │
└─────────────────┘
    │
    ↓
┌─────────────────┐
│  Body Parser    │ (Parse JSON)
└─────────────────┘
    │
    ↓
┌─────────────────┐
│  Route Handler  │ (Match URL pattern)
└─────────────────┘
    │
    ↓
┌─────────────────┐
│ Auth Middleware │ (Verify JWT Token)
│   (protect)     │
└─────────────────┘
    │
    ↓ (if authenticated)
┌─────────────────┐
│   Controller    │ (Business Logic)
└─────────────────┘
    │
    ↓
┌─────────────────┐
│     Model       │ (Database Operations)
└─────────────────┘
    │
    ↓
┌─────────────────┐
│    MongoDB      │ (Data Storage)
└─────────────────┘
    │
    ↓
┌─────────────────┐
│  HTTP Response  │ (JSON Data)
└─────────────────┘
```

---

## Database Architecture

### Entity Relationship Diagram

```
┌─────────────────┐
│      USER       │
├─────────────────┤
│ _id (PK)        │
│ name            │
│ email (unique)  │
│ password (hash) │
│ pic             │
│ isAdmin         │
│ createdAt       │
│ updatedAt       │
└─────────────────┘
       │
       │ 1
       │
       │ Many
       ↓
┌─────────────────┐         ┌─────────────────┐
│      CHAT       │◄────────│    MESSAGE      │
├─────────────────┤    Many │                 │
│ _id (PK)        │         ├─────────────────┤
│ chatName        │         │ _id (PK)        │
│ isGroupChat     │         │ sender (FK→User)│
│ users [FK]      │         │ content         │
│ latestMessage   │◄────────│ chat (FK→Chat)  │
│ groupAdmin (FK) │   1     │ readBy [FK]     │
│ createdAt       │         │ createdAt       │
│ updatedAt       │         │ updatedAt       │
└─────────────────┘         └─────────────────┘
       │
       │ Many
       │
       ↓
┌─────────────────┐
│      USER       │
└─────────────────┘
```

### Database Schema Details

#### User Collection
```javascript
{
  _id: ObjectId,
  name: String (required),
  email: String (unique, required),
  password: String (hashed, required),
  pic: String (default: avatar URL),
  isAdmin: Boolean (default: false),
  createdAt: Date,
  updatedAt: Date
}
```

#### Chat Collection
```javascript
{
  _id: ObjectId,
  chatName: String (for group chats),
  isGroupChat: Boolean (default: false),
  users: [ObjectId] (references User._id),
  latestMessage: ObjectId (references Message._id),
  groupAdmin: ObjectId (references User._id),
  createdAt: Date,
  updatedAt: Date
}
```

#### Message Collection
```javascript
{
  _id: ObjectId,
  sender: ObjectId (references User._id),
  content: String (required),
  chat: ObjectId (references Chat._id),
  readBy: [ObjectId] (references User._id),
  createdAt: Date,
  updatedAt: Date
}
```

### Relationship Types

```
1. User ↔ Chat: Many-to-Many
   - One user can be in many chats
   - One chat can have many users
   - Implemented via users array in Chat model

2. Chat ↔ Message: One-to-Many
   - One chat can have many messages
   - One message belongs to one chat
   - Implemented via chat field in Message model

3. User ↔ Message: Many-to-One
   - One user can send many messages
   - One message has one sender
   - Implemented via sender field in Message model
```

---

## Real-time Architecture

### Socket.io Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    SOCKET.IO ARCHITECTURE                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              Socket.io Server                        │  │
│  │                                                       │  │
│  │  Connection Events:                                  │  │
│  │  ┌───────────────────────────────────────────────┐  │  │
│  │  │ io.on("connection", (socket) => { ... })      │  │  │
│  │  └───────────────────────────────────────────────┘  │  │
│  │                                                       │  │
│  │  Room Management:                                    │  │
│  │  ┌───────────────────────────────────────────────┐  │  │
│  │  │ socket.join(userId)    → Personal Room        │  │  │
│  │  │ socket.join(chatId)    → Chat Room            │  │  │
│  │  └───────────────────────────────────────────────┘  │  │
│  │                                                       │  │
│  │  Event Handlers:                                     │  │
│  │  ┌───────────────────────────────────────────────┐  │  │
│  │  │ "setup"         → User joins personal room    │  │  │
│  │  │ "join chat"     → User joins chat room        │  │  │
│  │  │ "typing"        → Typing indicator start      │  │  │
│  │  │ "stop typing"   → Typing indicator stop       │  │  │
│  │  │ "new message"   → Broadcast message           │  │  │
│  │  └───────────────────────────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                          ↕ WebSocket
┌─────────────────────────────────────────────────────────────┐
│              SOCKET.IO CLIENT (Frontend)                   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  Connection:                                         │  │
│  │  const socket = io("http://localhost:5000")         │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  Event Emission:                                     │  │
│  │  socket.emit("setup", userData)                      │  │
│  │  socket.emit("join chat", chatId)                    │  │
│  │  socket.emit("typing", chatId)                       │  │
│  │  socket.emit("new message", messageData)             │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  Event Listening:                                    │  │
│  │  socket.on("connected", ...)                        │  │
│  │  socket.on("typing", ...)                            │  │
│  │  socket.on("message received", ...)                  │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Real-time Message Flow

```
User A sends message
    │
    ↓
┌─────────────────┐
│  Frontend (A)   │
│  1. Save via    │
│     REST API    │
│  2. Emit Socket│
│     event       │
└─────────────────┘
    │
    ↓ HTTP POST /api/message
┌─────────────────┐
│  Backend Server  │
│  1. Save to DB   │
│  2. Receive      │
│     Socket event │
└─────────────────┘
    │
    ↓ Socket.io Broadcast
┌─────────────────┐
│  Socket Server  │
│  Broadcast to   │
│  all users in    │
│  chat room       │
└─────────────────┘
    │
    ├───→ User B (Personal Room)
    │     └─── Frontend receives "message received"
    │
    └───→ User C (Personal Room)
          └─── Frontend receives "message received"
```

### Room System

```
┌─────────────────────────────────────────────────────────┐
│                    ROOM ARCHITECTURE                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Personal Rooms (User-specific)                        │
│  ┌───────────────────────────────────────────────────┐ │
│  │ Room Name: user._id                               │ │
│  │ Purpose: Receive notifications, personal messages │ │
│  │ Example: "user123", "user456"                    │ │
│  └───────────────────────────────────────────────────┘ │
│                                                         │
│  Chat Rooms (Chat-specific)                            │
│  ┌───────────────────────────────────────────────────┐ │
│  │ Room Name: chat._id                                │ │
│  │ Purpose: Receive messages for specific chat       │ │
│  │ Example: "chat789", "chat101"                    │ │
│  └───────────────────────────────────────────────────┘ │
│                                                         │
│  Broadcasting:                                          │
│  ┌───────────────────────────────────────────────────┐ │
│  │ socket.in(roomId).emit("event", data)             │ │
│  │ → Sends to all sockets in that room               │ │
│  └───────────────────────────────────────────────────┘ │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## API Architecture

### RESTful API Endpoints

```
┌─────────────────────────────────────────────────────────────┐
│                    API ENDPOINTS                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  USER ENDPOINTS (/api/user)                                │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ GET    /api/user?search=query  → Search users       │  │
│  │ POST   /api/user                → Register user     │  │
│  │ POST   /api/user/login          → Login user        │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  CHAT ENDPOINTS (/api/chat)                                │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ POST   /api/chat                → Access/Create chat│  │
│  │ GET    /api/chat                → Fetch all chats   │  │
│  │ POST   /api/chat/group          → Create group chat │  │
│  │ PUT    /api/chat/rename         → Rename group      │  │
│  │ PUT    /api/chat/groupadd       → Add user to group │  │
│  │ PUT    /api/chat/groupremove    → Remove from group │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  MESSAGE ENDPOINTS (/api/message)                          │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ GET    /api/message/:chatId     → Get all messages  │  │
│  │ POST   /api/message             → Send message      │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### API Request/Response Flow

```
┌─────────────────────────────────────────────────────────┐
│              API REQUEST FLOW                            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Client Request                                      │
│     ┌───────────────────────────────────────────────┐  │
│     │ POST /api/message                              │  │
│     │ Headers: { Authorization: "Bearer <token>" } │  │
│     │ Body: { content: "Hello", chatId: "123" }    │  │
│     └───────────────────────────────────────────────┘  │
│                                                         │
│  2. Route Matching                                      │
│     ┌───────────────────────────────────────────────┐  │
│     │ router.post("/", protect, sendMessage)        │  │
│     └───────────────────────────────────────────────┘  │
│                                                         │
│  3. Authentication                                      │
│     ┌───────────────────────────────────────────────┐  │
│     │ protect middleware verifies JWT token        │  │
│     │ Attaches user to req.user                     │  │
│     └───────────────────────────────────────────────┘  │
│                                                         │
│  4. Controller Processing                               │
│     ┌───────────────────────────────────────────────┐  │
│     │ sendMessage()                                 │  │
│     │ - Validates input                            │  │
│     │ - Creates message in DB                      │  │
│     │ - Populates sender and chat                   │  │
│     │ - Updates chat's latestMessage                │  │
│     └───────────────────────────────────────────────┘  │
│                                                         │
│  5. Response                                            │
│     ┌───────────────────────────────────────────────┐  │
│     │ Status: 200 OK                                 │  │
│     │ Body: { _id, sender, content, chat, ... }     │  │
│     └───────────────────────────────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Data Flow Diagrams

### User Registration Flow

```
┌──────────┐
│  Client  │
└──────────┘
    │
    │ POST /api/user
    │ { name, email, password }
    ↓
┌──────────┐
│  Server  │
└──────────┘
    │
    │ 1. Validate input
    │ 2. Check if user exists
    │ 3. Hash password (bcrypt)
    │ 4. Create user in DB
    │ 5. Generate JWT token
    ↓
┌──────────┐
│ MongoDB  │
└──────────┘
    │
    │ User document created
    ↓
┌──────────┐
│ Response │
│ { user,  │
│   token }│
└──────────┘
    │
    ↓
┌──────────┐
│  Client  │
│ Stores   │
│ token in │
│ localStorage
└──────────┘
```

### Message Sending Flow

```
┌──────────┐
│  Client  │
│ (User A) │
└──────────┘
    │
    │ 1. User types message
    │ 2. Presses Enter
    │
    │ POST /api/message
    │ { content, chatId }
    ↓
┌──────────┐
│  Server  │
└──────────┘
    │
    │ 1. Verify JWT token
    │ 2. Create message in DB
    │ 3. Update chat.latestMessage
    │ 4. Return message data
    ↓
┌──────────┐
│ MongoDB  │
└──────────┘
    │
    │ Message saved
    ↓
┌──────────┐
│ Response │
│ Message  │
│ object   │
└──────────┘
    │
    │ 2. Emit Socket event
    │ socket.emit("new message", data)
    ↓
┌──────────┐
│ Socket.io│
│  Server  │
└──────────┘
    │
    │ Broadcast to chat room
    │ (All users except sender)
    ↓
┌──────────┐     ┌──────────┐
│  Client  │     │  Client  │
│ (User B) │     │ (User C) │
└──────────┘     └──────────┘
    │                 │
    │ Receive         │ Receive
    │ "message        │ "message
    │  received"      │  received"
    │                 │
    │ Update UI       │ Update UI
    │ (if chat        │ (if chat
    │  selected)      │  selected)
    │                 │
    │ OR              │ OR
    │                 │
    │ Add to          │ Add to
    │ notifications   │ notifications
    │ (if chat        │ (if chat
    │  not selected)  │  not selected)
```

### Chat Creation Flow

```
┌──────────┐
│  Client  │
│ (User A) │
└──────────┘
    │
    │ Clicks on User B
    │
    │ POST /api/chat
    │ { userId: "userB_id" }
    ↓
┌──────────┐
│  Server  │
└──────────┘
    │
    │ 1. Verify JWT token
    │ 2. Query: Check if chat exists
    │    between User A and User B
    │
    │ Query MongoDB:
    │ Chat.find({
    │   isGroupChat: false,
    │   $and: [
    │     { users: { $elemMatch: { $eq: userA } } },
    │     { users: { $elemMatch: { $eq: userB } } }
    │   ]
    │ })
    ↓
┌──────────┐
│ MongoDB  │
└──────────┘
    │
    │ Query Result
    ↓
┌──────────┐
│  Server  │
└──────────┘
    │
    │ IF chat exists:
    │   Return existing chat
    │
    │ IF chat doesn't exist:
    │   Create new chat
    │   { users: [userA, userB],
    │     isGroupChat: false }
    │
    ↓
┌──────────┐
│ MongoDB  │
└──────────┘
    │
    │ Chat created/retrieved
    ↓
┌──────────┐
│ Response │
│ Chat     │
│ object   │
└──────────┘
    │
    ↓
┌──────────┐
│  Client  │
│ Updates  │
│ UI with  │
│ chat     │
└──────────┘
```

---

## Component Structure

### Component Communication

```
┌─────────────────────────────────────────────────────────┐
│              COMPONENT COMMUNICATION                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Chatpage (Parent)                                     │
│  ├─── Manages: fetchAgain state                        │
│  │                                                     │
│  ├─── SideDrawer                                       │
│  │    ├─── Uses: ChatState() for user                 │
│  │    ├─── Updates: setSelectedChat()                  │
│  │    └─── Features: Search, Logout                   │
│  │                                                     │
│  ├─── MyChats                                          │
│  │    ├─── Uses: ChatState() for chats, user         │
│  │    ├─── Receives: fetchAgain prop                  │
│  │    ├─── Updates: setSelectedChat()                  │
│  │    └─── Features: Display chat list                │
│  │                                                     │
│  └─── Chatbox                                          │
│       ├─── Receives: fetchAgain, setFetchAgain props  │
│       ├─── Uses: ChatState() for selectedChat         │
│       │                                                     │
│       └─── SingleChat                                   │
│            ├─── Uses: ChatState() for user,            │
│            │            selectedChat, notification      │
│            ├─── Updates: setNotification(),            │
│            │            setFetchAgain()                 │
│            ├─── Manages: messages (local state)       │
│            ├─── Socket.io: Connection, events         │
│            │                                             │
│            ├─── ScrollableChat                          │
│            │    ├─── Receives: messages prop          │
│            │    └─── Uses: ChatState() for user        │
│            │                                             │
│            └─── Message Input                          │
│                 ├─── Manages: newMessage state        │
│                 ├─── Emits: Socket events             │
│                 └─── Shows: Typing indicator           │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Props Flow

```
Chatpage
  │
  ├─── fetchAgain ────────────→ MyChats
  │                              │
  │                              └─── Triggers fetchChats()
  │
  ├─── fetchAgain ────────────→ Chatbox
  │   setFetchAgain              │
  │                              └─── SingleChat
  │                                   │
  │                                   ├─── Uses fetchAgain
  │                                   │    to refresh chat list
  │                                   │
  │                                   └─── Calls setFetchAgain()
  │                                        when notification received
```

---

## Deployment Architecture

### Production Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    PRODUCTION SETUP                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌───────────────────────────────────────────────────┐ │
│  │              CDN / Static Hosting                  │ │
│  │  (Netlify, Vercel, AWS S3 + CloudFront)           │ │
│  │                                                     │ │
│  │  Serves: React build files (HTML, CSS, JS)        │ │
│  └───────────────────────────────────────────────────┘ │
│                          ↕                              │
│  ┌───────────────────────────────────────────────────┐ │
│  │            Load Balancer (Nginx)                    │ │
│  │  - Routes HTTP requests                            │ │
│  │  - Handles WebSocket connections (sticky sessions) │ │
│  │  - SSL/TLS termination                             │ │
│  └───────────────────────────────────────────────────┘ │
│                          ↕                              │
│  ┌───────────────────────────────────────────────────┐ │
│  │         Node.js Server Instances                    │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐         │ │
│  │  │Instance 1│  │Instance 2│  │Instance 3│         │ │
│  │  │Express + │  │Express + │  │Express + │         │ │
│  │  │Socket.io │  │Socket.io │  │Socket.io │         │ │
│  │  └──────────┘  └──────────┘  └──────────┘         │ │
│  │                                                     │ │
│  │  Redis Adapter for Socket.io                      │ │
│  │  (Shared state across instances)                   │ │
│  └───────────────────────────────────────────────────┘ │
│                          ↕                              │
│  ┌───────────────────────────────────────────────────┐ │
│  │              MongoDB Atlas                         │ │
│  │  - Primary Database                                │ │
│  │  - Read Replicas (optional)                       │ │
│  │  - Automated Backups                               │ │
│  └───────────────────────────────────────────────────┘ │
│                          ↕                              │
│  ┌───────────────────────────────────────────────────┐ │
│  │              Redis Cache                           │ │
│  │  - Session storage                                 │ │
│  │  - Socket.io adapter                               │ │
│  │  - Caching frequently accessed data                │ │
│  └───────────────────────────────────────────────────┘ │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Environment Configuration

```
Development:
├── Frontend: http://localhost:3000
├── Backend:  http://localhost:5000
├── Database: MongoDB local or Atlas
└── Socket.io: ws://localhost:5000

Production:
├── Frontend: https://yourdomain.com
├── Backend:  https://api.yourdomain.com
├── Database: MongoDB Atlas (cloud)
├── Socket.io: wss://api.yourdomain.com
└── Redis:    Redis Cloud or AWS ElastiCache
```

---

## Security Architecture

### Authentication Flow

```
┌─────────────────────────────────────────────────────────┐
│              AUTHENTICATION ARCHITECTURE                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. User Registration/Login                            │
│     ┌───────────────────────────────────────────────┐  │
│     │ Client → Server: { email, password }          │  │
│     │ Server: Hash password with bcrypt             │  │
│     │ Server: Generate JWT token                    │  │
│     │ Server → Client: { user, token }              │  │
│     │ Client: Store token in localStorage            │  │
│     └───────────────────────────────────────────────┘  │
│                                                         │
│  2. Protected Route Access                             │
│     ┌───────────────────────────────────────────────┐  │
│     │ Client → Server: Request + Authorization      │  │
│     │              header (Bearer token)             │  │
│     │                                               │  │
│     │ Server: Extract token from header             │  │
│     │ Server: Verify token signature               │  │
│     │ Server: Extract user ID from token            │  │
│     │ Server: Fetch user from database              │  │
│     │ Server: Attach user to req.user               │  │
│     │ Server: Continue to route handler             │  │
│     └───────────────────────────────────────────────┘  │
│                                                         │
│  3. Password Security                                  │
│     ┌───────────────────────────────────────────────┐  │
│     │ Storage: bcrypt hash (never plain text)        │  │
│     │ Salt: Random salt per password                 │  │
│     │ Rounds: 10 (2^10 iterations)                  │  │
│     │ Verification: bcrypt.compare()                │  │
│     └───────────────────────────────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Summary

### Key Architectural Decisions

1. **Separation of Concerns**
   - Frontend: UI and user interaction
   - Backend: Business logic and data management
   - Database: Data persistence

2. **Hybrid Communication**
   - REST API: Reliable data persistence
   - WebSockets: Real-time updates

3. **State Management**
   - Context API: Global state (user, chats, notifications)
   - Local State: Component-specific state

4. **Database Design**
   - MongoDB: Flexible schema for chat data
   - References: Efficient relationships via ObjectIds
   - Populate: Avoid N+1 queries

5. **Real-time Architecture**
   - Socket.io: Bidirectional communication
   - Rooms: Targeted message delivery
   - Events: Typing indicators, notifications

### Scalability Considerations

- **Horizontal Scaling**: Multiple server instances with Redis adapter
- **Database**: MongoDB Atlas with read replicas
- **Caching**: Redis for frequently accessed data
- **Load Balancing**: Nginx with sticky sessions
- **CDN**: Static asset delivery

This architecture provides a solid foundation for a real-time chat application with room for growth and scalability.


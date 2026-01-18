# Talk-A-Tive - Visual Architecture Diagrams

This document contains visual diagrams of the project architecture using Mermaid syntax. These diagrams render beautifully in GitHub, VS Code, and most markdown viewers.

---

## 1. System Architecture Overview

```mermaid
graph TB
    subgraph "Client Layer"
        A[React Frontend]
        A1[Homepage - Auth]
        A2[Chatpage - Main App]
        A3[Context Provider]
        A --> A1
        A --> A2
        A --> A3
        A4[Socket.io Client]
        A --> A4
    end
    
    subgraph "Server Layer"
        B[Node.js + Express]
        B1[Routes]
        B2[Controllers]
        B3[Middleware]
        B --> B1
        B --> B2
        B --> B3
        B4[Socket.io Server]
        B --> B4
    end
    
    subgraph "Database Layer"
        C[MongoDB]
        C1[Users Collection]
        C2[Chats Collection]
        C3[Messages Collection]
        C --> C1
        C --> C2
        C --> C3
    end
    
    A4 <-->|WebSocket| B4
    A <-->|HTTP/REST| B
    B <-->|Mongoose ODM| C
    
    style A fill:#61dafb
    style B fill:#68a063
    style C fill:#4db33d
```

---

## 2. Frontend Component Hierarchy

```mermaid
graph TD
    A[App.js] --> B[Homepage]
    A --> C[Chatpage]
    
    B --> D[Login Component]
    B --> E[Signup Component]
    
    C --> F[ChatProvider Context]
    C --> G[SideDrawer]
    C --> H[MyChats]
    C --> I[Chatbox]
    
    F --> F1[user state]
    F --> F2[selectedChat state]
    F --> F3[chats state]
    F --> F4[notification state]
    
    G --> G1[User Search]
    G --> G2[User List]
    G --> G3[Logout]
    
    H --> H1[Chat List]
    H1 --> H2[Chat Item]
    H2 --> H3[UserListItem]
    
    I --> J[SingleChat]
    J --> J1[Chat Header]
    J --> J2[ScrollableChat]
    J --> J3[Message Input]
    
    J1 --> J4[ProfileModal]
    J1 --> J5[UpdateGroupModal]
    
    J2 --> J6[Message Items]
    J6 --> J7[Avatar]
    J6 --> J8[Message Content]
    
    J3 --> J9[Typing Indicator]
    J3 --> J10[Input Field]
    
    style A fill:#61dafb
    style F fill:#ffd700
    style J fill:#ff6b6b
```

---

## 3. Backend MVC Architecture

```mermaid
graph LR
    subgraph "Request Flow"
        A[HTTP Request] --> B[Express Server]
        B --> C[Route Handler]
        C --> D[Auth Middleware]
        D --> E[Controller]
        E --> F[Model]
        F --> G[MongoDB]
        G --> H[Response]
    end
    
    subgraph "MVC Structure"
        I[Routes Layer]
        J[Controllers Layer]
        K[Models Layer]
        
        I --> I1["/api/user"]
        I --> I2["/api/chat"]
        I --> I3["/api/message"]
        
        J --> J1[userControllers]
        J --> J2[chatControllers]
        J --> J3[messageControllers]
        
        K --> K1[userModel]
        K --> K2[chatModel]
        K --> K3[messageModel]
    end
    
    style B fill:#68a063
    style D fill:#ff9800
    style E fill:#2196f3
    style F fill:#9c27b0
```

---

## 4. Database Schema Relationships

```mermaid
erDiagram
    USER ||--o{ CHAT : "participates in"
    USER ||--o{ MESSAGE : "sends"
    CHAT ||--o{ MESSAGE : "contains"
    USER ||--o{ CHAT : "administers"
    
    USER {
        ObjectId _id PK
        string name
        string email UK
        string password
        string pic
        boolean isAdmin
        date createdAt
        date updatedAt
    }
    
    CHAT {
        ObjectId _id PK
        string chatName
        boolean isGroupChat
        array users FK
        ObjectId latestMessage FK
        ObjectId groupAdmin FK
        date createdAt
        date updatedAt
    }
    
    MESSAGE {
        ObjectId _id PK
        ObjectId sender FK
        string content
        ObjectId chat FK
        array readBy FK
        date createdAt
        date updatedAt
    }
```

---

## 5. Real-time Message Flow

```mermaid
sequenceDiagram
    participant UA as User A (Client)
    participant API as REST API
    participant DB as MongoDB
    participant WS as Socket.io Server
    participant UB as User B (Client)
    participant UC as User C (Client)
    
    UA->>API: POST /api/message<br/>{content, chatId}
    API->>DB: Save message to database
    DB-->>API: Message saved
    API-->>UA: Return message object
    
    UA->>WS: emit("new message", messageData)
    
    WS->>WS: Broadcast to chat room<br/>(all users except sender)
    
    WS->>UB: emit("message received")
    WS->>UC: emit("message received")
    
    alt Chat is selected
        UB->>UB: Add to messages array
        UC->>UC: Add to messages array
    else Chat not selected
        UB->>UB: Add to notifications
        UC->>UC: Add to notifications
    end
```

---

## 6. Authentication Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant API as API Server
    participant DB as MongoDB
    participant JWT as JWT Service
    
    Note over C,JWT: Registration Flow
    C->>API: POST /api/user<br/>{name, email, password}
    API->>DB: Check if user exists
    DB-->>API: User not found
    API->>API: Hash password (bcrypt)
    API->>DB: Create user
    DB-->>API: User created
    API->>JWT: Generate token(userId)
    JWT-->>API: JWT token
    API-->>C: {user, token}
    C->>C: Store token in localStorage
    
    Note over C,JWT: Login Flow
    C->>API: POST /api/user/login<br/>{email, password}
    API->>DB: Find user by email
    DB-->>API: User found
    API->>API: Compare password (bcrypt.compare)
    API->>JWT: Generate token(userId)
    JWT-->>API: JWT token
    API-->>C: {user, token}
    
    Note over C,JWT: Protected Route Access
    C->>API: GET /api/chat<br/>Header: Bearer token
    API->>JWT: Verify token
    JWT-->>API: Token valid, userId
    API->>DB: Find user by userId
    DB-->>API: User data
    API->>API: Attach user to req.user
    API->>API: Execute route handler
    API-->>C: Response with data
```

---

## 7. Socket.io Room Architecture

```mermaid
graph TB
    subgraph "Socket.io Server"
        A[Socket Connection]
        A --> B[User Setup Event]
        A --> C[Join Chat Event]
    end
    
    subgraph "Room System"
        D[Personal Room<br/>user._id]
        E[Chat Room<br/>chat._id]
    end
    
    subgraph "Users"
        F[User A<br/>Room: userA]
        G[User B<br/>Room: userB]
        H[User C<br/>Room: userC]
    end
    
    subgraph "Chat Rooms"
        I[Chat 1<br/>userA, userB]
        J[Chat 2<br/>userA, userC]
    end
    
    B --> D
    C --> E
    
    F --> D
    G --> D
    H --> D
    
    F --> I
    G --> I
    F --> J
    H --> J
    
    style D fill:#ffd700
    style E fill:#61dafb
    style I fill:#4db33d
    style J fill:#4db33d
```

---

## 8. Chat Creation Flow

```mermaid
flowchart TD
    A[User A clicks User B] --> B[POST /api/chat<br/>{userId: userB}]
    B --> C{Auth Middleware}
    C -->|Invalid Token| D[401 Unauthorized]
    C -->|Valid Token| E[Controller: accessChat]
    
    E --> F[Query MongoDB:<br/>Find chat between<br/>User A and User B]
    
    F --> G{Chat Exists?}
    
    G -->|Yes| H[Return Existing Chat]
    G -->|No| I[Create New Chat]
    
    I --> J[Chat Data:<br/>users: [userA, userB]<br/>isGroupChat: false]
    J --> K[Save to MongoDB]
    K --> L[Populate Users]
    L --> M[Return Chat Object]
    
    H --> N[Update UI]
    M --> N
    
    style C fill:#ff9800
    style G fill:#2196f3
    style H fill:#4caf50
    style M fill:#4caf50
```

---

## 9. State Management Flow

```mermaid
graph TB
    subgraph "Global State - Context API"
        A[ChatProvider]
        A --> A1[user state]
        A --> A2[selectedChat state]
        A --> A3[chats state]
        A --> A4[notification state]
    end
    
    subgraph "Component Local State"
        B[SingleChat Component]
        B --> B1[messages state]
        B --> B2[loading state]
        B --> B3[newMessage state]
        B --> B4[socketConnected state]
        B --> B5[istyping state]
    end
    
    subgraph "Other Components"
        C[MyChats Component]
        C --> C1[local loading state]
        
        D[Login Component]
        D --> D1[form state]
    end
    
    A -->|Provides| B
    A -->|Provides| C
    A -->|Provides| D
    
    B -->|Updates| A4
    B -->|Reads| A2
    B -->|Reads| A1
    
    style A fill:#ffd700
    style B fill:#ff6b6b
    style C fill:#4ecdc4
    style D fill:#95e1d3
```

---

## 10. API Endpoints Structure

```mermaid
graph TD
    A[Express Server] --> B[/api/user]
    A --> C[/api/chat]
    A --> D[/api/message]
    
    B --> B1[GET /api/user?search=query<br/>Protected]
    B --> B2[POST /api/user<br/>Public - Register]
    B --> B3[POST /api/user/login<br/>Public - Login]
    
    C --> C1[POST /api/chat<br/>Protected - Access/Create]
    C --> C2[GET /api/chat<br/>Protected - Fetch All]
    C --> C3[POST /api/chat/group<br/>Protected - Create Group]
    C --> C4[PUT /api/chat/rename<br/>Protected - Rename]
    C --> C5[PUT /api/chat/groupadd<br/>Protected - Add User]
    C --> C6[PUT /api/chat/groupremove<br/>Protected - Remove User]
    
    D --> D1[GET /api/message/:chatId<br/>Protected - Get Messages]
    D --> D2[POST /api/message<br/>Protected - Send Message]
    
    style A fill:#68a063
    style B fill:#2196f3
    style C fill:#ff9800
    style D fill:#9c27b0
```

---

## 11. Typing Indicator Flow

```mermaid
sequenceDiagram
    participant UA as User A
    participant WS as Socket.io
    participant UB as User B
    
    Note over UA: User starts typing
    UA->>UA: First keystroke detected
    UA->>WS: emit("typing", chatId)
    WS->>UB: Broadcast "typing" event
    UB->>UB: Show typing indicator
    
    Note over UA: User continues typing
    UA->>UA: More keystrokes
    UA->>UA: Reset 3-second timer
    
    Note over UA: User stops typing
    UA->>UA: No keystroke for 3 seconds
    UA->>WS: emit("stop typing", chatId)
    WS->>UB: Broadcast "stop typing" event
    UB->>UB: Hide typing indicator
```

---

## 12. Notification System Flow

```mermaid
flowchart TD
    A[Message Received via Socket] --> B{Check selectedChat}
    
    B -->|selectedChat === null| C[Add to Notifications]
    B -->|selectedChat._id !== message.chat._id| C
    B -->|selectedChat._id === message.chat._id| D[Add to Messages Array]
    
    C --> E[Update Notification Badge]
    C --> F[Refresh Chat List]
    C --> G[Show Notification Count]
    
    D --> H[Display Message in Chat]
    
    I[User Clicks Notification] --> J[Select Chat]
    J --> K[Clear Notification for Chat]
    K --> L[Load Messages]
    
    style C fill:#ff9800
    style D fill:#4caf50
    style E fill:#ffd700
```

---

## 13. Production Deployment Architecture

```mermaid
graph TB
    subgraph "CDN / Static Hosting"
        A[React Build Files]
    end
    
    subgraph "Load Balancer"
        B[Nginx]
        B --> B1[SSL/TLS Termination]
        B --> B2[Sticky Sessions]
        B --> B3[WebSocket Proxy]
    end
    
    subgraph "Application Servers"
        C1[Node.js Instance 1]
        C2[Node.js Instance 2]
        C3[Node.js Instance 3]
        C1 --> C4[Express + Socket.io]
        C2 --> C4
        C3 --> C4
    end
    
    subgraph "Cache Layer"
        D[Redis]
        D --> D1[Socket.io Adapter]
        D --> D2[Session Storage]
        D --> D3[Data Cache]
    end
    
    subgraph "Database"
        E[MongoDB Atlas]
        E --> E1[Primary Database]
        E --> E2[Read Replicas]
    end
    
    A --> B
    B --> C1
    B --> C2
    B --> C3
    C1 --> D
    C2 --> D
    C3 --> D
    C1 --> E
    C2 --> E
    C3 --> E
    
    style A fill:#61dafb
    style B fill:#ff9800
    style C4 fill:#68a063
    style D fill:#dc382d
    style E fill:#4db33d
```

---

## 14. Complete Data Flow - Message Sending

```mermaid
graph TB
    subgraph "Frontend - User A"
        A1[User Types Message]
        A2[Presses Enter]
        A3[POST /api/message]
        A4[Emit Socket Event]
        A5[Update UI Optimistically]
    end
    
    subgraph "Backend Server"
        B1[Receive HTTP Request]
        B2[Auth Middleware]
        B3[Controller: sendMessage]
        B4[Save to MongoDB]
        B5[Update Chat.latestMessage]
        B6[Return Message Object]
        B7[Receive Socket Event]
        B8[Broadcast to Chat Room]
    end
    
    subgraph "Database"
        C1[(Messages Collection)]
        C2[(Chats Collection)]
    end
    
    subgraph "Socket.io Server"
        D1[Room: chatId]
        D2[Broadcast to All Users]
    end
    
    subgraph "Frontend - User B & C"
        E1[Receive Socket Event]
        E2{Chat Selected?}
        E3[Add to Messages]
        E4[Add to Notifications]
    end
    
    A1 --> A2
    A2 --> A3
    A3 --> B1
    B1 --> B2
    B2 --> B3
    B3 --> B4
    B4 --> C1
    B3 --> B5
    B5 --> C2
    B3 --> B6
    B6 --> A4
    A4 --> A5
    A4 --> B7
    B7 --> B8
    B8 --> D1
    D1 --> D2
    D2 --> E1
    E1 --> E2
    E2 -->|Yes| E3
    E2 -->|No| E4
    
    style A5 fill:#4caf50
    style B2 fill:#ff9800
    style D2 fill:#2196f3
    style E3 fill:#4caf50
    style E4 fill:#ff9800
```

---

## 15. Component Communication Diagram

```mermaid
graph LR
    subgraph "Chatpage Component"
        A[Chatpage]
        A --> A1[fetchAgain State]
    end
    
    subgraph "SideDrawer"
        B[SideDrawer]
        B --> B1[Search Users]
        B --> B2[Select Chat]
        B --> B3[Logout]
    end
    
    subgraph "MyChats"
        C[MyChats]
        C --> C1[Display Chat List]
        C --> C2[Select Chat]
    end
    
    subgraph "Chatbox"
        D[Chatbox]
        D --> E[SingleChat]
    end
    
    subgraph "SingleChat"
        E --> E1[Fetch Messages]
        E --> E2[Send Message]
        E --> E3[Receive Messages]
        E --> E4[Typing Indicator]
    end
    
    A1 -->|prop| C
    A1 -->|prop| D
    B2 -->|setSelectedChat| F[Context]
    C2 -->|setSelectedChat| F
    E3 -->|setNotification| F
    E3 -->|setFetchAgain| A1
    F -->|selectedChat| E
    F -->|user| E
    F -->|notification| B
    
    style A fill:#61dafb
    style F fill:#ffd700
    style E fill:#ff6b6b
```

---

## How to View These Diagrams

### In VS Code:
1. Install "Markdown Preview Mermaid Support" extension
2. Open this file and use Markdown Preview

### In GitHub:
- Diagrams render automatically when you push this file

### Online:
- Copy diagram code to https://mermaid.live/
- View and export as PNG/SVG

### In Other Editors:
- Use Mermaid plugins/extensions for your editor
- Or use online Mermaid editor

---

## Diagram Legend

- **Blue**: Frontend/Client components
- **Green**: Backend/Server components
- **Yellow**: State/Context
- **Orange**: Middleware/Auth
- **Purple**: Database/Models
- **Red**: Real-time/Socket.io

---

These diagrams provide a comprehensive visual representation of your project's architecture. Use them for documentation, presentations, or explaining your project to others!


# HostelHub Architecture Diagram

## Current Architecture (Frontend Only)

```
┌─────────────────────────────────────────────────────────────┐
│                     USER INTERFACE (React)                  │
│  Pages: Login, Dashboard, Rooms, Students, Complaints, etc. │
└──────────────────────┬──────────────────────────────────────┘
                       │ Uses
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              CONTEXT API (State Management)                  │
│  AuthContext, handles login, logout, role-based access      │
└──────────────────────┬──────────────────────────────────────┘
                       │ Calls
                       ▼
┌─────────────────────────────────────────────────────────────┐
│           SERVICE LAYER (lib/services/index.ts)             │
│  - authService.login()                                       │
│  - roomService.getAllRooms()                                 │
│  - complaintService.createComplaint()                        │
│  - paymentService.getPaymentHistory()                        │
│  - etc.                                                       │
└──────────────────────┬──────────────────────────────────────┘
                       │ Currently returns
                       ▼
┌─────────────────────────────────────────────────────────────┐
│           MOCK DATA (lib/services/mockData.ts)              │
│  - mockData.users (students & wardens)                       │
│  - mockData.rooms                                            │
│  - mockData.complaints                                       │
│  - mockData.payments                                         │
└──────────────────────┬──────────────────────────────────────┘
                       │ Stored in
                       ▼
┌─────────────────────────────────────────────────────────────┐
│        BROWSER localStorage (Data Persistence)              │
│  Data saved locally, persists across page reloads            │
└─────────────────────────────────────────────────────────────┘
```

---

## Future Architecture (With Backend & MongoDB)

```
┌─────────────────────────────────────────────────────────────┐
│                     USER INTERFACE (React)                  │
│  Pages: Login, Dashboard, Rooms, Students, Complaints, etc. │
└──────────────────────┬──────────────────────────────────────┘
                       │ Uses (NO CHANGES)
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              CONTEXT API (State Management)                  │
│  AuthContext, handles login, logout, role-based access      │
└──────────────────────┬──────────────────────────────────────┘
                       │ Calls (NO CHANGES)
                       ▼
┌─────────────────────────────────────────────────────────────┐
│           SERVICE LAYER (lib/services/index.ts)             │
│  NOW UPDATED TO CALL REAL API INSTEAD OF MOCK DATA!         │
│  - authService.login() → calls /api/auth/login              │
│  - roomService.getAllRooms() → calls /api/rooms             │
│  - complaintService.createComplaint() → calls /api/complaints
│  - paymentService.getPaymentHistory() → calls /api/payments │
│  - etc.                                                       │
└──────────────────────┬──────────────────────────────────────┘
                       │ Calls via HTTP
                       ▼
        ┌──────────────────────────────────────┐
        │   HTTP/HTTPS (Network Request)       │
        │   JSON data sent over internet        │
        └──────────────┬───────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│          BACKEND SERVER (Node.js/Express)                   │
│                                                              │
│  Routes:                                                     │
│  POST   /api/auth/login          → authController          │
│  POST   /api/auth/register       → authController          │
│  GET    /api/rooms               → roomController          │
│  POST   /api/rooms               → roomController          │
│  PUT    /api/rooms/:id           → roomController          │
│  GET    /api/complaints          → complaintController     │
│  POST   /api/complaints          → complaintController     │
│  PATCH  /api/complaints/:id      → complaintController     │
│  GET    /api/payments            → paymentController       │
│  GET    /api/messages            → messageController       │
│  GET    /api/students            → studentController       │
│                                                              │
│  Middleware:                                                 │
│  - Authentication (JWT verification)                        │
│  - Authorization (Role-based access control)                │
│  - Error handling                                            │
│  - Logging                                                   │
└──────────────────────┬──────────────────────────────────────┘
                       │ Queries
                       ▼
┌─────────────────────────────────────────────────────────────┐
│          DATABASE (MongoDB)                                  │
│                                                              │
│  Collections:                                                │
│  ├── users                                                   │
│  │   ├── _id (ObjectId)                                      │
│  │   ├── name (string)                                       │
│  │   ├── email (string - unique)                             │
│  │   ├── password (hashed string)                            │
│  │   ├── role ("student" | "warden")                         │
│  │   └── timestamps                                          │
│  │                                                            │
│  ├── rooms                                                   │
│  │   ├── _id (ObjectId)                                      │
│  │   ├── roomNumber (string - unique)                        │
│  │   ├── capacity (number)                                   │
│  │   ├── status ("available" | "occupied" | "maintenance")   │
│  │   ├── pricePerNight (number)                              │
│  │   └── timestamps                                          │
│  │                                                            │
│  ├── complaints                                              │
│  │   ├── _id (ObjectId)                                      │
│  │   ├── studentId (reference to users)                      │
│  │   ├── title (string)                                      │
│  │   ├── description (string)                                │
│  │   ├── category (string)                                   │
│  │   ├── status ("open" | "in-progress" | "resolved")        │
│  │   ├── wardenResponse (string)                             │
│  │   └── timestamps                                          │
│  │                                                            │
│  ├── payments                                                │
│  │   ├── _id (ObjectId)                                      │
│  │   ├── studentId (reference to users)                      │
│  │   ├── amount (number)                                     │
│  │   ├── status ("pending" | "completed" | "failed")         │
│  │   └── timestamps                                          │
│  │                                                            │
│  ├── messages                                                │
│  │   ├── _id (ObjectId)                                      │
│  │   ├── senderId (reference to users)                       │
│  │   ├── recipientId (reference to users)                    │
│  │   ├── content (string)                                    │
│  │   ├── read (boolean)                                      │
│  │   └── createdAt (timestamp)                               │
│  │                                                            │
│  └── bookings                                                │
│      ├── _id (ObjectId)                                      │
│      ├── studentId (reference to users)                      │
│      ├── roomId (reference to rooms)                         │
│      ├── checkInDate (date)                                  │
│      ├── checkOutDate (date)                                 │
│      ├── totalPrice (number)                                 │
│      ├── status (string)                                     │
│      └── timestamps                                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Data Flow Examples

### Example 1: Student Login Flow

#### Current (Frontend Only)
```
User enters email/password
    ↓
Click Login button
    ↓
authService.login(email, password)
    ↓
Returns mock user from mockData.users
    ↓
Saved to localStorage
    ↓
Redirect to Dashboard
```

#### Future (With Backend)
```
User enters email/password
    ↓
Click Login button
    ↓
authService.login(email, password)
    ↓
HTTP POST to backend: /api/auth/login
    ↓
Backend queries MongoDB: users.findOne({ email })
    ↓
Backend verifies password with bcrypt
    ↓
Backend generates JWT token
    ↓
Backend sends back: { token, user }
    ↓
Frontend saves token to localStorage
    ↓
Redirect to Dashboard
```

### Example 2: Create Complaint Flow

#### Current (Frontend Only)
```
Warden fills complaint form
    ↓
Clicks Submit
    ↓
complaintService.createComplaint(data)
    ↓
Adds to mockData.complaints
    ↓
Updates localStorage
    ↓
Show success toast
```

#### Future (With Backend)
```
Warden fills complaint form
    ↓
Clicks Submit
    ↓
complaintService.createComplaint(data)
    ↓
HTTP POST to backend: /api/complaints
    ↓
Backend creates document in MongoDB: complaints.insertOne(data)
    ↓
Backend sends back: { id, status, createdAt, ... }
    ↓
Frontend stores in localStorage cache
    ↓
Show success toast
```

### Example 3: Update Complaint Status (Warden)

#### Current (Frontend Only)
```
Warden views complaint detail
    ↓
Clicks "Mark as Resolved"
    ↓
complaintService.updateComplaint(id, { status: 'resolved' })
    ↓
Updates mockData.complaints
    ↓
Updates localStorage
    ↓
Show updated status in UI
```

#### Future (With Backend)
```
Warden views complaint detail
    ↓
Clicks "Mark as Resolved"
    ↓
complaintService.updateComplaint(id, { status: 'resolved' })
    ↓
HTTP PATCH to backend: /api/complaints/:id
    ↓
Backend updates MongoDB: complaints.updateOne({ _id }, { status })
    ↓
Backend sends back: { updated complaint }
    ↓
Frontend updates localStorage cache
    ↓
Show updated status in UI
```

---

## File Organization

```
hostel-management/
│
├── frontend (this project - React/Next.js)
│   │
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── auth/
│   │   │   └── login/
│   │   │       └── page.tsx
│   │   └── dashboard/
│   │       ├── page.tsx
│   │       ├── rooms/
│   │       ├── students/
│   │       ├── complaints/
│   │       ├── analytics/
│   │       └── payments/
│   │
│   ├── components/
│   │   ├── ChatbotWidget.tsx
│   │   ├── layout/
│   │   │   ├── Header.tsx
│   │   │   ├── StudentSidebar.tsx
│   │   │   └── WardenSidebar.tsx
│   │   ├── dashboard/
│   │   │   ├── StudentDashboard.tsx
│   │   │   └── WardenDashboard.tsx
│   │   └── ui/ (shadcn components)
│   │
│   ├── lib/
│   │   ├── api/
│   │   │   └── client.ts          ⬅️ ADD THIS
│   │   ├── services/
│   │   │   ├── index.ts           ⬅️ MODIFY THIS
│   │   │   └── mockData.ts
│   │   └── types/
│   │       └── index.ts
│   │
│   ├── contexts/
│   │   └── AuthContext.tsx
│   │
│   ├── .env.local                 ⬅️ ADD THIS
│   └── package.json
│
└── backend (Node.js/Express) ⬅️ CREATE THIS
    │
    ├── server.js                  ⬅️ MAIN FILE
    │
    ├── routes/
    │   ├── auth.js                (login, register)
    │   ├── rooms.js               (CRUD operations)
    │   ├── complaints.js          (create, update, resolve)
    │   ├── payments.js            (payment history, records)
    │   ├── messages.js            (chat messages)
    │   └── students.js            (student management)
    │
    ├── middleware/
    │   └── auth.js                (JWT verification)
    │
    ├── controllers/ (optional)
    │   ├── authController.js
    │   ├── roomController.js
    │   └── ...
    │
    ├── models/ (if using Mongoose)
    │   ├── User.js
    │   ├── Room.js
    │   ├── Complaint.js
    │   └── ...
    │
    ├── .env                       (MongoDB URI, JWT secret)
    ├── package.json
    └── server.js
```

---

## Component Dependency Graph

```
App (Root)
│
├── AuthContext Provider
│   │
│   ├── ProtectedRoute
│   │   │
│   │   └── Dashboard Layout
│   │       │
│   │       ├── Header
│   │       ├── StudentSidebar or WardenSidebar
│   │       │
│   │       └── Page Components
│   │           ├── StudentDashboard
│   │           ├── WardenDashboard
│   │           ├── RoomsPage
│   │           ├── StudentsPage
│   │           ├── ComplaintsPage
│   │           ├── AnalyticsPage
│   │           └── etc.
│   │
│   └── Login Page
│       └── LoginForm (uses authService)
│
└── ChatbotWidget (Global)
```

---

## Technology Stack

### Frontend
- **Framework**: Next.js 16 (React 19)
- **Language**: TypeScript
- **Styling**: Tailwind CSS 4
- **Components**: ShadCN UI
- **Icons**: Lucide React
- **Animations**: Framer Motion
- **Charts**: Recharts
- **State**: React Context + localStorage
- **Forms**: React Hook Form + Zod

### Backend (When Ready)
- **Runtime**: Node.js
- **Framework**: Express.js (or NestJS)
- **Language**: JavaScript/TypeScript
- **Database**: MongoDB
- **Authentication**: JWT
- **Password Hashing**: bcryptjs
- **Validation**: Joi or Zod
- **ORM**: Mongoose (optional)

---

## API Communication Pattern

All API calls follow this pattern:

```
Frontend Service Layer
    ↓
apiCall() utility function
    ↓
Adds headers (Content-Type, Authorization)
    ↓
HTTP request (GET, POST, PUT, DELETE, PATCH)
    ↓
Backend Express Route
    ↓
Middleware (auth, validation)
    ↓
Controller (business logic)
    ↓
MongoDB Query
    ↓
Response back to Frontend
    ↓
Service updates state/localStorage
    ↓
Component re-renders with new data
```

---

## Security Flow (With Backend)

```
User enters credentials
    ↓
Frontend sends to /api/auth/login
    ↓
Backend verifies password with bcrypt
    ↓
Backend generates JWT token (contains userId, role)
    ↓
Frontend stores JWT in localStorage
    ↓
Subsequent requests include JWT in Authorization header
    ↓
Backend middleware verifies JWT
    ↓
Grants access based on role (student, warden)
    ↓
Backend queries MongoDB with user context
    ↓
Returns only data user has access to
```

---

## Deployment Architecture

### Current (Frontend Only)
```
GitHub Repository
    ↓
Vercel (Frontend Hosting)
    ↓
Browser Caches Data in localStorage
```

### Future (With Backend)
```
Frontend GitHub → Vercel (Frontend)
Backend GitHub → Railway/Render (Backend)
                    ↓
              MongoDB Atlas
                    ↓
Frontend talks to Backend API
Backend reads/writes MongoDB
```

---

## Summary

**The beauty of this architecture:**

1. **Components never change** - They always call services with same signatures
2. **Easy migration** - Swap mock data with real API one service at a time
3. **Scalable** - Start simple with Express, upgrade to NestJS later
4. **Testable** - Each service can be tested independently
5. **Type-safe** - Full TypeScript support throughout

The separation of concerns means you can build and deploy backend independently, and frontend will just work!

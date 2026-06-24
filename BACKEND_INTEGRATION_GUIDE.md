# Backend Integration Guide: Adding API Calls & MongoDB

This guide explains how to add a real backend with MongoDB to replace the current mock data system.

## Current Architecture (Frontend-Only)

```
Frontend (React/Next.js)
    ↓
Context API + localStorage
    ↓
Mock Services (lib/services/index.ts)
    ↓
Mock Data (lib/services/mockData.ts)
```

## Target Architecture (With Backend)

```
Frontend (React/Next.js)
    ↓
Context API
    ↓
Real Services with API calls
    ↓
Backend Server (Node.js/Express)
    ↓
MongoDB Database
```

---

## Step 1: Setup Backend Project

### Option A: Express.js (Recommended for beginners)

```bash
mkdir hostel-backend
cd hostel-backend
npm init -y
npm install express mongodb dotenv cors
npm install -D nodemon
```

Create `server.js`:
```javascript
const express = require('express');
const cors = require('cors');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

Update `package.json`:
```json
"scripts": {
  "start": "node server.js",
  "dev": "nodemon server.js"
}
```

### Option B: NestJS (Enterprise-grade)

```bash
npm install -g @nestjs/cli
nest new hostel-backend
cd hostel-backend
```

---

## Step 2: Setup MongoDB

### Local MongoDB
```bash
# Install MongoDB Community Edition
# macOS: brew install mongodb-community
# Windows: Download from mongodb.com
# Linux: Follow MongoDB official docs

# Start MongoDB
mongod
```

### MongoDB Atlas (Cloud)
1. Create account at https://www.mongodb.com/cloud/atlas
2. Create a cluster (free tier available)
3. Get connection string: `mongodb+srv://username:password@cluster.mongodb.net/dbname`

### Environment Variables

Create `.env` file in backend:
```
MONGODB_URI=mongodb://localhost:27017/hostel_db
# OR for MongoDB Atlas:
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/hostel_db
PORT=5000
NODE_ENV=development
JWT_SECRET=your_secret_key_here
```

---

## Step 3: Database Schema with MongoDB

### 1. Users Collection (Students & Wardens)

```javascript
// models/User.js
const userSchema = {
  _id: ObjectId,
  name: String,
  email: String (unique),
  password: String (hashed with bcryptjs),
  phone: String,
  role: "student" | "warden",
  address: String,
  emergencyContact: String,
  createdAt: Date,
  updatedAt: Date
}

// Example:
{
  "_id": ObjectId("..."),
  "name": "John Doe",
  "email": "john@example.com",
  "password": "$2b$10$...", // bcrypt hashed
  "role": "student",
  "phone": "+1234567890",
  "address": "123 Main St",
  "emergencyContact": "+9876543210",
  "createdAt": ISODate("2024-01-15"),
  "updatedAt": ISODate("2024-01-15")
}
```

### 2. Rooms Collection

```javascript
const roomSchema = {
  _id: ObjectId,
  roomNumber: String (unique),
  capacity: Number,
  currentOccupancy: Number,
  amenities: [String],
  pricePerNight: Number,
  status: "available" | "occupied" | "maintenance",
  floor: Number,
  createdAt: Date,
  updatedAt: Date
}

// Example:
{
  "_id": ObjectId("..."),
  "roomNumber": "101",
  "capacity": 2,
  "currentOccupancy": 1,
  "amenities": ["WiFi", "Attached Bathroom", "AC"],
  "pricePerNight": 500,
  "status": "occupied",
  "floor": 1,
  "createdAt": ISODate("2024-01-15"),
  "updatedAt": ISODate("2024-01-15")
}
```

### 3. Bookings Collection

```javascript
const bookingSchema = {
  _id: ObjectId,
  studentId: ObjectId (reference to User),
  roomId: ObjectId (reference to Room),
  checkInDate: Date,
  checkOutDate: Date,
  totalPrice: Number,
  status: "pending" | "confirmed" | "checked-in" | "checked-out" | "cancelled",
  notes: String,
  createdAt: Date,
  updatedAt: Date
}
```

### 4. Complaints Collection

```javascript
const complaintSchema = {
  _id: ObjectId,
  studentId: ObjectId (reference to User),
  title: String,
  description: String,
  category: "maintenance" | "cleanliness" | "noise" | "other",
  status: "open" | "in-progress" | "resolved",
  priority: "low" | "medium" | "high",
  wardenResponse: String,
  createdAt: Date,
  updatedAt: Date,
  resolvedAt: Date
}
```

### 5. Messages Collection

```javascript
const messageSchema = {
  _id: ObjectId,
  senderId: ObjectId (reference to User),
  recipientId: ObjectId (reference to User),
  content: String,
  read: Boolean,
  createdAt: Date
}
```

---

## Step 4: Create API Routes

### Express.js Example Routes

```javascript
// routes/auth.js
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const router = express.Router();

// Login Route
router.post('/login', async (req, res) => {
  const { email, password } = req.body;
  
  try {
    const user = await db.collection('users').findOne({ email });
    
    if (!user) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }
    
    const isPasswordValid = await bcrypt.compare(password, user.password);
    
    if (!isPasswordValid) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }
    
    const token = jwt.sign(
      { userId: user._id, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '24h' }
    );
    
    res.json({
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role
      }
    });
  } catch (error) {
    res.status(500).json({ message: 'Server error' });
  }
});

// Register Route
router.post('/register', async (req, res) => {
  const { name, email, password, role, phone, address } = req.body;
  
  try {
    const existingUser = await db.collection('users').findOne({ email });
    
    if (existingUser) {
      return res.status(400).json({ message: 'User already exists' });
    }
    
    const hashedPassword = await bcrypt.hash(password, 10);
    
    const result = await db.collection('users').insertOne({
      name,
      email,
      password: hashedPassword,
      role,
      phone,
      address,
      createdAt: new Date(),
      updatedAt: new Date()
    });
    
    res.status(201).json({ message: 'User created', userId: result.insertedId });
  } catch (error) {
    res.status(500).json({ message: 'Server error' });
  }
});

module.exports = router;
```

---

## Step 5: Update Frontend Services

### Current Mock Service (lib/services/index.ts)

```typescript
export const authService = {
  login: async (email: string, password: string) => {
    // Currently returns mock data
    return mockData.users.find(u => u.email === email);
  }
}
```

### Updated with Real API (lib/services/index.ts)

```typescript
const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:5000/api';

export const authService = {
  login: async (email: string, password: string) => {
    const response = await fetch(`${API_BASE_URL}/auth/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password })
    });
    
    if (!response.ok) {
      throw new Error('Login failed');
    }
    
    const data = await response.json();
    // Store token in localStorage
    localStorage.setItem('token', data.token);
    return data.user;
  },
  
  register: async (userData: any) => {
    const response = await fetch(`${API_BASE_URL}/auth/register`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(userData)
    });
    
    if (!response.ok) {
      throw new Error('Registration failed');
    }
    
    return response.json();
  }
}
```

### Add to .env.local

```
NEXT_PUBLIC_API_URL=http://localhost:5000/api
```

---

## Step 6: Create Utility Function for API Calls

Create `lib/api/client.ts`:

```typescript
export async function apiCall<T>(
  endpoint: string,
  options?: RequestInit
): Promise<T> {
  const url = `${process.env.NEXT_PUBLIC_API_URL}${endpoint}`;
  
  const headers = {
    'Content-Type': 'application/json',
    ...options?.headers,
  };
  
  // Add auth token if available
  const token = localStorage.getItem('token');
  if (token) {
    headers['Authorization'] = `Bearer ${token}`;
  }
  
  const response = await fetch(url, {
    ...options,
    headers,
  });
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'API Error');
  }
  
  return response.json();
}

// Usage:
// const user = await apiCall<User>('/auth/login', {
//   method: 'POST',
//   body: JSON.stringify({ email, password })
// });
```

---

## Step 7: Create All API Endpoints

### Authentication Endpoints

```
POST   /api/auth/login              - Login user
POST   /api/auth/register           - Register new user
POST   /api/auth/logout             - Logout user
POST   /api/auth/refresh-token      - Refresh JWT token
GET    /api/auth/me                 - Get current user
```

### Rooms Endpoints

```
GET    /api/rooms                   - Get all rooms
GET    /api/rooms/:id               - Get room details
POST   /api/rooms                   - Create room (Warden only)
PUT    /api/rooms/:id               - Update room (Warden only)
DELETE /api/rooms/:id               - Delete room (Warden only)
GET    /api/rooms/:id/availability  - Check availability
```

### Complaints Endpoints

```
GET    /api/complaints              - Get all complaints
GET    /api/complaints/:id          - Get complaint details
POST   /api/complaints              - Create complaint (Student)
PUT    /api/complaints/:id          - Update complaint (Warden)
PATCH  /api/complaints/:id/resolve  - Resolve complaint (Warden)
```

### Payments Endpoints

```
GET    /api/payments                - Get payment history
POST   /api/payments                - Record payment
GET    /api/payments/:id            - Get payment details
```

### Messages Endpoints

```
GET    /api/messages                - Get messages
POST   /api/messages                - Send message
GET    /api/messages/:conversationId - Get conversation
```

### Students Endpoints (Warden only)

```
GET    /api/students                - Get all students
GET    /api/students/:id            - Get student details
POST   /api/students                - Add student
PUT    /api/students/:id            - Update student
DELETE /api/students/:id            - Delete student
```

---

## Step 8: Implement Authentication Middleware

### Express.js Middleware

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');

const verifyToken = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ message: 'No token provided' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ message: 'Invalid token' });
  }
};

const requireRole = (role) => {
  return (req, res, next) => {
    if (req.user.role !== role && req.user.role !== 'admin') {
      return res.status(403).json({ message: 'Access denied' });
    }
    next();
  };
};

module.exports = { verifyToken, requireRole };
```

---

## Step 9: Migration Checklist

- [ ] Setup backend server (Express/NestJS)
- [ ] Setup MongoDB (Local or Atlas)
- [ ] Create database collections with schema
- [ ] Implement authentication routes
- [ ] Implement all CRUD endpoints
- [ ] Add middleware for auth & role-based access
- [ ] Update frontend API client (`lib/api/client.ts`)
- [ ] Update all services in `lib/services/index.ts`
- [ ] Update environment variables
- [ ] Test API endpoints with Postman/Insomnia
- [ ] Update Context to handle real API responses
- [ ] Add error handling & loading states
- [ ] Deploy backend (Railway, Render, Heroku, AWS)

---

## Step 10: Recommended Tools & Libraries

### Backend Dependencies
```json
{
  "express": "^4.18.2",
  "mongoose": "^7.0.0",
  "bcryptjs": "^2.4.3",
  "jsonwebtoken": "^9.0.0",
  "cors": "^2.8.5",
  "dotenv": "^16.0.3",
  "joi": "^17.9.1"
}
```

### Frontend Dependencies (Already included)
```json
{
  "next": "^16.0.0",
  "react": "^19.0.0",
  "typescript": "^5.0.0"
}
```

---

## Step 11: Quick Example - Adding Rooms API

### Backend (Express)

```javascript
// routes/rooms.js
router.get('/', verifyToken, async (req, res) => {
  try {
    const rooms = await db.collection('rooms').find({}).toArray();
    res.json(rooms);
  } catch (error) {
    res.status(500).json({ message: 'Error fetching rooms' });
  }
});

router.post('/', verifyToken, requireRole('warden'), async (req, res) => {
  try {
    const result = await db.collection('rooms').insertOne({
      ...req.body,
      createdAt: new Date(),
      updatedAt: new Date()
    });
    res.status(201).json(result);
  } catch (error) {
    res.status(500).json({ message: 'Error creating room' });
  }
});
```

### Frontend (Service)

```typescript
// lib/services/index.ts
export const roomService = {
  getAllRooms: async () => {
    return apiCall<Room[]>('/rooms');
  },
  
  createRoom: async (roomData: Partial<Room>) => {
    return apiCall<Room>('/rooms', {
      method: 'POST',
      body: JSON.stringify(roomData)
    });
  },
  
  updateRoom: async (roomId: string, data: Partial<Room>) => {
    return apiCall<Room>(`/rooms/${roomId}`, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }
}
```

---

## Common Issues & Solutions

### CORS Issues
**Problem**: Frontend can't call backend API

**Solution**:
```javascript
const cors = require('cors');
app.use(cors({
  origin: 'http://localhost:3000',
  credentials: true
}));
```

### Token Not Sent
**Problem**: Authorization header missing

**Solution**: Update `lib/api/client.ts` to include token:
```typescript
const token = localStorage.getItem('token');
if (token) {
  headers['Authorization'] = `Bearer ${token}`;
}
```

### Data Not Persisting
**Problem**: Changes are lost on page reload

**Solution**: Ensure MongoDB is running and connection string is correct in `.env`

---

## Deployment

### Backend Deployment (Railway/Render)

1. Push code to GitHub
2. Connect repository to Railway/Render
3. Set environment variables
4. Deploy

### Frontend Deployment (Vercel)

```bash
git push origin main
# Vercel automatically deploys
```

Update `NEXT_PUBLIC_API_URL` in Vercel environment variables to point to production backend.

---

## Summary

1. **Mock → Real**: Replace mock services with API calls
2. **Same Signatures**: Keep function signatures identical for zero component changes
3. **One Step at a Time**: Migrate services one by one
4. **Test Thoroughly**: Verify each endpoint before moving to next
5. **Keep It Simple**: Start with Express, scale to NestJS later if needed

The beauty of this architecture is that **your frontend components never change** - only the service layer implementation changes!

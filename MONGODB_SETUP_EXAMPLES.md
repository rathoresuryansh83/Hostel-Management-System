# MongoDB Setup & Code Examples

Complete code examples for setting up MongoDB and integrating with your frontend.

## Quick Start - MongoDB Atlas (Cloud)

### 1. Create Free MongoDB Atlas Account
1. Go to https://www.mongodb.com/cloud/atlas
2. Click "Try Free"
3. Create account with email
4. Create organization and project
5. Create a free cluster (M0)
6. Wait for cluster to be ready (2-3 minutes)

### 2. Get Connection String
1. Click "Connect" button on cluster
2. Click "Drivers"
3. Copy connection string: `mongodb+srv://username:password@cluster0.xxxx.mongodb.net/database_name`
4. Save to `.env` file

---

## Backend Setup with Express & MongoDB

### 1. Complete Backend Server Code

File: `backend/server.js`

```javascript
const express = require('express');
const { MongoClient, ObjectId } = require('mongodb');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const cors = require('cors');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

// MongoDB Connection
let db;
MongoClient.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
}).then(client => {
  console.log('Connected to MongoDB');
  db = client.db('hostel_db');
}).catch(error => {
  console.error('MongoDB connection error:', error);
  process.exit(1);
});

// Middleware: Verify JWT Token
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

// Middleware: Check User Role
const requireRole = (role) => {
  return (req, res, next) => {
    if (req.user.role !== role && req.user.role !== 'admin') {
      return res.status(403).json({ message: 'Access denied' });
    }
    next();
  };
};

// ==================== AUTH ROUTES ====================

// Login
app.post('/api/auth/login', async (req, res) => {
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
      { 
        userId: user._id.toString(),
        role: user.role,
        name: user.name 
      },
      process.env.JWT_SECRET,
      { expiresIn: '24h' }
    );
    
    res.json({
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role,
        phone: user.phone
      }
    });
  } catch (error) {
    console.error('Login error:', error);
    res.status(500).json({ message: 'Server error' });
  }
});

// Register
app.post('/api/auth/register', async (req, res) => {
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
    
    res.status(201).json({ 
      message: 'User created successfully',
      userId: result.insertedId 
    });
  } catch (error) {
    console.error('Register error:', error);
    res.status(500).json({ message: 'Server error' });
  }
});

// ==================== ROOMS ROUTES ====================

// Get All Rooms
app.get('/api/rooms', verifyToken, async (req, res) => {
  try {
    const rooms = await db.collection('rooms')
      .find({})
      .toArray();
    res.json(rooms);
  } catch (error) {
    res.status(500).json({ message: 'Error fetching rooms' });
  }
});

// Get Room by ID
app.get('/api/rooms/:id', verifyToken, async (req, res) => {
  try {
    const room = await db.collection('rooms').findOne({
      _id: new ObjectId(req.params.id)
    });
    
    if (!room) {
      return res.status(404).json({ message: 'Room not found' });
    }
    
    res.json(room);
  } catch (error) {
    res.status(500).json({ message: 'Error fetching room' });
  }
});

// Create Room (Warden only)
app.post('/api/rooms', verifyToken, requireRole('warden'), async (req, res) => {
  const { roomNumber, floor, capacity, amenities, pricePerNight } = req.body;
  
  try {
    const result = await db.collection('rooms').insertOne({
      roomNumber,
      floor,
      capacity,
      currentOccupancy: 0,
      amenities,
      pricePerNight,
      status: 'available',
      createdAt: new Date(),
      updatedAt: new Date()
    });
    
    res.status(201).json({
      message: 'Room created',
      roomId: result.insertedId
    });
  } catch (error) {
    res.status(500).json({ message: 'Error creating room' });
  }
});

// Update Room (Warden only)
app.put('/api/rooms/:id', verifyToken, requireRole('warden'), async (req, res) => {
  try {
    const result = await db.collection('rooms').findOneAndUpdate(
      { _id: new ObjectId(req.params.id) },
      { 
        $set: {
          ...req.body,
          updatedAt: new Date()
        }
      },
      { returnDocument: 'after' }
    );
    
    if (!result.value) {
      return res.status(404).json({ message: 'Room not found' });
    }
    
    res.json(result.value);
  } catch (error) {
    res.status(500).json({ message: 'Error updating room' });
  }
});

// Delete Room (Warden only)
app.delete('/api/rooms/:id', verifyToken, requireRole('warden'), async (req, res) => {
  try {
    const result = await db.collection('rooms').deleteOne({
      _id: new ObjectId(req.params.id)
    });
    
    if (result.deletedCount === 0) {
      return res.status(404).json({ message: 'Room not found' });
    }
    
    res.json({ message: 'Room deleted' });
  } catch (error) {
    res.status(500).json({ message: 'Error deleting room' });
  }
});

// ==================== COMPLAINTS ROUTES ====================

// Get All Complaints
app.get('/api/complaints', verifyToken, async (req, res) => {
  try {
    const query = req.user.role === 'student' 
      ? { studentId: new ObjectId(req.user.userId) }
      : {};
    
    const complaints = await db.collection('complaints')
      .find(query)
      .sort({ createdAt: -1 })
      .toArray();
    
    res.json(complaints);
  } catch (error) {
    res.status(500).json({ message: 'Error fetching complaints' });
  }
});

// Create Complaint (Student)
app.post('/api/complaints', verifyToken, requireRole('student'), async (req, res) => {
  const { title, description, category } = req.body;
  
  try {
    const result = await db.collection('complaints').insertOne({
      studentId: new ObjectId(req.user.userId),
      title,
      description,
      category,
      status: 'open',
      priority: 'medium',
      wardenResponse: null,
      createdAt: new Date(),
      updatedAt: new Date(),
      resolvedAt: null
    });
    
    res.status(201).json({
      message: 'Complaint submitted',
      complaintId: result.insertedId
    });
  } catch (error) {
    res.status(500).json({ message: 'Error creating complaint' });
  }
});

// Resolve Complaint (Warden only)
app.patch('/api/complaints/:id/resolve', verifyToken, requireRole('warden'), async (req, res) => {
  const { wardenResponse } = req.body;
  
  try {
    const result = await db.collection('complaints').findOneAndUpdate(
      { _id: new ObjectId(req.params.id) },
      { 
        $set: {
          status: 'resolved',
          wardenResponse,
          resolvedAt: new Date(),
          updatedAt: new Date()
        }
      },
      { returnDocument: 'after' }
    );
    
    if (!result.value) {
      return res.status(404).json({ message: 'Complaint not found' });
    }
    
    res.json(result.value);
  } catch (error) {
    res.status(500).json({ message: 'Error resolving complaint' });
  }
});

// ==================== STUDENTS ROUTES (Warden only) ====================

// Get All Students
app.get('/api/students', verifyToken, requireRole('warden'), async (req, res) => {
  try {
    const students = await db.collection('users')
      .find({ role: 'student' })
      .toArray();
    
    res.json(students);
  } catch (error) {
    res.status(500).json({ message: 'Error fetching students' });
  }
});

// Get Student by ID
app.get('/api/students/:id', verifyToken, requireRole('warden'), async (req, res) => {
  try {
    const student = await db.collection('users').findOne({
      _id: new ObjectId(req.params.id),
      role: 'student'
    });
    
    if (!student) {
      return res.status(404).json({ message: 'Student not found' });
    }
    
    res.json(student);
  } catch (error) {
    res.status(500).json({ message: 'Error fetching student' });
  }
});

// ==================== MESSAGES ROUTES ====================

// Get Messages
app.get('/api/messages', verifyToken, async (req, res) => {
  try {
    const messages = await db.collection('messages')
      .find({
        $or: [
          { senderId: new ObjectId(req.user.userId) },
          { recipientId: new ObjectId(req.user.userId) }
        ]
      })
      .sort({ createdAt: -1 })
      .toArray();
    
    res.json(messages);
  } catch (error) {
    res.status(500).json({ message: 'Error fetching messages' });
  }
});

// Send Message
app.post('/api/messages', verifyToken, async (req, res) => {
  const { recipientId, content } = req.body;
  
  try {
    const result = await db.collection('messages').insertOne({
      senderId: new ObjectId(req.user.userId),
      recipientId: new ObjectId(recipientId),
      content,
      read: false,
      createdAt: new Date()
    });
    
    res.status(201).json({
      message: 'Message sent',
      messageId: result.insertedId
    });
  } catch (error) {
    res.status(500).json({ message: 'Error sending message' });
  }
});

// ==================== ANALYTICS ROUTES ====================

// Get Dashboard Analytics
app.get('/api/analytics/dashboard', verifyToken, requireRole('warden'), async (req, res) => {
  try {
    const totalRooms = await db.collection('rooms').countDocuments();
    const occupiedRooms = await db.collection('rooms').countDocuments({ status: 'occupied' });
    const totalStudents = await db.collection('users').countDocuments({ role: 'student' });
    const totalComplaints = await db.collection('complaints').countDocuments();
    const openComplaints = await db.collection('complaints').countDocuments({ status: 'open' });
    
    const totalRevenue = await db.collection('payments')
      .aggregate([
        { $match: { status: 'completed' } },
        { $group: { _id: null, total: { $sum: '$amount' } } }
      ])
      .toArray();
    
    res.json({
      totalRooms,
      occupiedRooms,
      occupancyRate: ((occupiedRooms / totalRooms) * 100).toFixed(2),
      totalStudents,
      totalComplaints,
      openComplaints,
      totalRevenue: totalRevenue[0]?.total || 0
    });
  } catch (error) {
    res.status(500).json({ message: 'Error fetching analytics' });
  }
});

// ==================== SERVER START ====================

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`✅ Server running on http://localhost:${PORT}`);
  console.log(`📚 API base: http://localhost:${PORT}/api`);
});
```

### 2. Backend Package.json

File: `backend/package.json`

```json
{
  "name": "hostel-backend",
  "version": "1.0.0",
  "description": "HostelHub Backend API",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongodb": "^5.8.0",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.1.1",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

### 3. Backend .env File

File: `backend/.env`

```
MONGODB_URI=mongodb+srv://username:password@cluster0.xxxx.mongodb.net/hostel_db
JWT_SECRET=your_super_secret_jwt_key_12345
PORT=5000
NODE_ENV=development
```

---

## Frontend Setup

### 1. Create API Client

File: `lib/api/client.ts`

```typescript
const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:5000/api';

export async function apiCall<T>(
  endpoint: string,
  options?: RequestInit
): Promise<T> {
  const url = `${API_BASE_URL}${endpoint}`;
  
  const headers: HeadersInit = {
    'Content-Type': 'application/json',
    ...options?.headers,
  };
  
  // Add auth token if available
  const token = typeof window !== 'undefined' 
    ? localStorage.getItem('token')
    : null;
  
  if (token) {
    headers['Authorization'] = `Bearer ${token}`;
  }
  
  try {
    const response = await fetch(url, {
      ...options,
      headers,
    });
    
    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message || 'API Error');
    }
    
    return response.json();
  } catch (error) {
    console.error(`API Error [${endpoint}]:`, error);
    throw error;
  }
}
```

### 2. Update Services Layer

File: `lib/services/index.ts` (Replace mock functions)

```typescript
import { apiCall } from '@/lib/api/client';
import { User, Room, Complaint, Payment } from '@/lib/types';

// Auth Service
export const authService = {
  login: async (email: string, password: string): Promise<User> => {
    const response = await apiCall<{ token: string; user: User }>('/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password })
    });
    
    // Store token for subsequent requests
    localStorage.setItem('token', response.token);
    return response.user;
  },

  register: async (userData: any): Promise<{ userId: string }> => {
    return apiCall('/auth/register', {
      method: 'POST',
      body: JSON.stringify(userData)
    });
  },

  logout: () => {
    localStorage.removeItem('token');
  }
};

// Rooms Service
export const roomService = {
  getAllRooms: async (): Promise<Room[]> => {
    return apiCall('/rooms');
  },

  getRoomById: async (roomId: string): Promise<Room> => {
    return apiCall(`/rooms/${roomId}`);
  },

  createRoom: async (roomData: Partial<Room>): Promise<{ roomId: string }> => {
    return apiCall('/rooms', {
      method: 'POST',
      body: JSON.stringify(roomData)
    });
  },

  updateRoom: async (roomId: string, data: Partial<Room>): Promise<Room> => {
    return apiCall(`/rooms/${roomId}`, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
  },

  deleteRoom: async (roomId: string): Promise<void> => {
    return apiCall(`/rooms/${roomId}`, { method: 'DELETE' });
  }
};

// Complaints Service
export const complaintService = {
  getAllComplaints: async (): Promise<Complaint[]> => {
    return apiCall('/complaints');
  },

  createComplaint: async (data: Partial<Complaint>): Promise<{ complaintId: string }> => {
    return apiCall('/complaints', {
      method: 'POST',
      body: JSON.stringify(data)
    });
  },

  resolveComplaint: async (complaintId: string, response: string): Promise<Complaint> => {
    return apiCall(`/complaints/${complaintId}/resolve`, {
      method: 'PATCH',
      body: JSON.stringify({ wardenResponse: response })
    });
  }
};

// Messages Service
export const messageService = {
  getMessages: async (): Promise<any[]> => {
    return apiCall('/messages');
  },

  sendMessage: async (recipientId: string, content: string): Promise<{ messageId: string }> => {
    return apiCall('/messages', {
      method: 'POST',
      body: JSON.stringify({ recipientId, content })
    });
  }
};

// Students Service
export const studentService = {
  getAllStudents: async (): Promise<User[]> => {
    return apiCall('/students');
  },

  getStudentById: async (studentId: string): Promise<User> => {
    return apiCall(`/students/${studentId}`);
  }
};

// Analytics Service
export const analyticsService = {
  getDashboardAnalytics: async (): Promise<any> => {
    return apiCall('/analytics/dashboard');
  }
};
```

### 3. Update .env.local

File: `.env.local`

```
NEXT_PUBLIC_API_URL=http://localhost:5000/api
NEXT_PUBLIC_APP_NAME=HostelHub
```

---

## MongoDB Collections Setup

Run these commands in MongoDB Compass or MongoDB CLI:

```javascript
// Create collections
db.createCollection('users');
db.createCollection('rooms');
db.createCollection('complaints');
db.createCollection('payments');
db.createCollection('messages');
db.createCollection('bookings');

// Create indexes for faster queries
db.users.createIndex({ email: 1 }, { unique: true });
db.rooms.createIndex({ roomNumber: 1 }, { unique: true });
db.complaints.createIndex({ studentId: 1 });
db.messages.createIndex({ senderId: 1, recipientId: 1 });

// Insert sample warden
db.users.insertOne({
  name: "Admin Warden",
  email: "warden@hostel.com",
  password: "$2b$10$...", // bcrypt hash of "password"
  phone: "+1234567890",
  role: "warden",
  address: "Hostel Building",
  createdAt: new Date(),
  updatedAt: new Date()
});

// Insert sample student
db.users.insertOne({
  name: "John Student",
  email: "student@hostel.com",
  password: "$2b$10$...", // bcrypt hash of "password"
  phone: "+9876543210",
  role: "student",
  address: "Dorm Room 101",
  createdAt: new Date(),
  updatedAt: new Date()
});
```

---

## Testing with Postman

### 1. Login
```
POST http://localhost:5000/api/auth/login
Content-Type: application/json

{
  "email": "student@hostel.com",
  "password": "password"
}

Response:
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "...",
    "name": "John Student",
    "email": "student@hostel.com",
    "role": "student"
  }
}
```

### 2. Get All Rooms
```
GET http://localhost:5000/api/rooms
Authorization: Bearer <token_from_login>
Content-Type: application/json
```

### 3. Create Complaint
```
POST http://localhost:5000/api/complaints
Authorization: Bearer <token_from_login>
Content-Type: application/json

{
  "title": "Room AC not working",
  "description": "The AC in my room stopped working",
  "category": "maintenance"
}
```

---

## Running Everything Together

### Terminal 1: Start Backend
```bash
cd backend
npm install
npm run dev

# Should output:
# ✅ Server running on http://localhost:5000
# 📚 API base: http://localhost:5000/api
```

### Terminal 2: Start Frontend
```bash
npm install
npm run dev

# Should output:
# ▲ Next.js 16.0.0
# Local:    http://localhost:3000
```

### Terminal 3: MongoDB
```bash
# MongoDB Atlas - no action needed (cloud)
# OR
# Local MongoDB:
mongod
```

### Browser: Test the App
1. Open http://localhost:3000
2. Login with student@hostel.com / password
3. Check browser console - should see API calls going to http://localhost:5000
4. Create a complaint - should appear in MongoDB
5. Refresh page - data persists from MongoDB!

---

## Troubleshooting

### "Cannot GET /api/rooms"
- Backend not running
- Wrong API_URL in .env.local
- Route not defined in server.js

### "401 Invalid token"
- Token expired
- Token format incorrect
- JWT_SECRET mismatch between backend and login

### "MongoDB Connection Error"
- MongoDB URI wrong
- MongoDB Atlas IP not whitelisted
- Local MongoDB not running

### CORS Errors
Make sure backend has:
```javascript
app.use(cors());
```

---

**You're now ready to connect your frontend to a real backend with MongoDB!** 🎉

# Quick Reference: Where to Add API Calls

## The Three-File System

When you're ready to add a real backend, you'll modify **ONLY 3 types of files**:

### 1. API Client Setup (CREATE THIS)
**File**: `lib/api/client.ts`

```typescript
// Create this file to handle all API calls
export async function apiCall<T>(
  endpoint: string,
  options?: RequestInit
): Promise<T> {
  const url = `${process.env.NEXT_PUBLIC_API_URL}${endpoint}`;
  const response = await fetch(url, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    }
  });
  return response.json();
}
```

**File**: `.env.local` (ADD THIS)
```
NEXT_PUBLIC_API_URL=http://localhost:5000/api
```

### 2. Update Services (MODIFY THESE)
**Location**: `lib/services/index.ts`

**BEFORE (Mock)**:
```typescript
export const authService = {
  login: async (email: string, password: string) => {
    // Simulates API delay
    await new Promise(resolve => setTimeout(resolve, 500));
    return mockData.users.find(u => u.email === email);
  }
}
```

**AFTER (Real API)**:
```typescript
import { apiCall } from '@/lib/api/client';

export const authService = {
  login: async (email: string, password: string) => {
    // Real API call
    return apiCall<User>('/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password })
    });
  }
}
```

### 3. Components & Pages (NO CHANGES NEEDED)
**Location**: All `.tsx` files

**These stay EXACTLY the same!** They call the services, so as long as service signatures don't change, components work automatically.

---

## Service-by-Service Migration Guide

### Step 1: Auth Service
**File to modify**: `lib/services/index.ts`

Replace these functions:
```typescript
authService.login()
authService.logout()
authService.register()
```

**Backend endpoints needed**:
```
POST /auth/login
POST /auth/register
POST /auth/logout
```

**Example**:
```typescript
export const authService = {
  login: async (email: string, password: string) => {
    const response = await apiCall<{ token: string; user: User }>('/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password })
    });
    localStorage.setItem('token', response.token);
    return response.user;
  }
}
```

---

### Step 2: Rooms Service
**File to modify**: `lib/services/index.ts`

Replace these functions:
```typescript
roomService.getAllRooms()
roomService.getRoomById()
roomService.createRoom()
roomService.updateRoom()
roomService.deleteRoom()
```

**Backend endpoints needed**:
```
GET /rooms
GET /rooms/:id
POST /rooms
PUT /rooms/:id
DELETE /rooms/:id
```

---

### Step 3: Complaints Service
**File to modify**: `lib/services/index.ts`

Replace these functions:
```typescript
complaintService.getAllComplaints()
complaintService.getComplaintById()
complaintService.createComplaint()
complaintService.updateComplaint()
complaintService.resolveComplaint()
```

**Backend endpoints needed**:
```
GET /complaints
GET /complaints/:id
POST /complaints
PUT /complaints/:id
PATCH /complaints/:id/resolve
```

---

### Step 4: Payments Service
Replace these functions:
```typescript
paymentService.getPaymentHistory()
paymentService.getPaymentDetails()
paymentService.recordPayment()
```

---

### Step 5: Messages Service
Replace these functions:
```typescript
messageService.getMessages()
messageService.sendMessage()
messageService.getConversation()
```

---

### Step 6: Students Service
Replace these functions:
```typescript
studentService.getAllStudents()
studentService.getStudentById()
studentService.createStudent()
studentService.updateStudent()
studentService.deleteStudent()
```

---

## Folder Structure After Adding Backend

```
hostel-management/
├── frontend/                (This project)
│   ├── app/
│   ├── components/
│   ├── contexts/
│   ├── lib/
│   │   ├── api/
│   │   │   └── client.ts        (NEW - API calls)
│   │   └── services/
│   │       ├── index.ts         (MODIFY - Real API)
│   │       └── mockData.ts      (Keep for reference)
│   └── .env.local              (NEW - API URL)
│
└── backend/                 (NEW - Your server)
    ├── server.js
    ├── routes/
    │   ├── auth.js
    │   ├── rooms.js
    │   ├── complaints.js
    │   ├── payments.js
    │   ├── messages.js
    │   └── students.js
    ├── middleware/
    │   └── auth.js
    ├── models/          (Optional - if using Mongoose)
    └── .env             (MongoDB URI, JWT Secret)
```

---

## Step-by-Step Migration Example

Let's say you want to migrate the `roomService`:

### 1. Check Current Service
```typescript
// lib/services/index.ts - BEFORE
export const roomService = {
  getAllRooms: async () => {
    await new Promise(resolve => setTimeout(resolve, 300));
    return mockData.rooms;
  }
}
```

### 2. Create Backend Endpoint
```javascript
// backend/routes/rooms.js
router.get('/', async (req, res) => {
  const rooms = await db.collection('rooms').find({}).toArray();
  res.json(rooms);
});
```

### 3. Update Frontend Service
```typescript
// lib/services/index.ts - AFTER
import { apiCall } from '@/lib/api/client';

export const roomService = {
  getAllRooms: async () => {
    return apiCall<Room[]>('/rooms');  // Real API call now!
  }
}
```

### 4. Test & Done!
- The component doesn't change
- No re-renders or errors
- Seamless migration!

---

## Testing Your API Integration

### 1. Start Backend Server
```bash
cd backend
npm run dev
# Server running on http://localhost:5000
```

### 2. Start Frontend
```bash
npm run dev
# Frontend running on http://localhost:3000
```

### 3. Test with Postman/Insomnia
```
POST http://localhost:5000/api/auth/login
Content-Type: application/json

{
  "email": "student@hostel.com",
  "password": "password"
}
```

### 4. Check Browser Console
Make sure no errors when API is called. Token should be saved to localStorage.

---

## Common Patterns

### Pattern 1: GET with Error Handling
```typescript
export const userService = {
  getProfile: async () => {
    try {
      return await apiCall<User>('/auth/me');
    } catch (error) {
      console.error('Failed to fetch profile:', error);
      throw error;
    }
  }
}
```

### Pattern 2: POST with Token
```typescript
export const complaintService = {
  createComplaint: async (data: ComplaintData) => {
    return apiCall<Complaint>('/complaints', {
      method: 'POST',
      body: JSON.stringify(data)
      // Token is automatically added by apiCall()
    });
  }
}
```

### Pattern 3: PUT with ID
```typescript
export const roomService = {
  updateRoom: async (roomId: string, data: Partial<Room>) => {
    return apiCall<Room>(`/rooms/${roomId}`, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }
}
```

### Pattern 4: DELETE
```typescript
export const studentService = {
  deleteStudent: async (studentId: string) => {
    return apiCall<void>(`/students/${studentId}`, {
      method: 'DELETE'
    });
  }
}
```

---

## Important: Keep the Same Function Signatures!

This is the KEY to zero-change component migration:

```typescript
// These functions must ALWAYS accept the same parameters
// and return the same types - whether mock or real!

// ✅ GOOD - Same signature
export const authService = {
  login: async (email: string, password: string): Promise<User> => {
    // Mock version
    return mockData.users.find(u => u.email === email)!;
  }
}

// Later...
export const authService = {
  login: async (email: string, password: string): Promise<User> => {
    // Real API version - SAME SIGNATURE!
    return apiCall<User>('/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password })
    });
  }
}

// ❌ BAD - Different signature will break components!
export const authService = {
  login: async (email: string, password: string, options?: LoginOptions): Promise<{ user: User; token: string }> => {
    // Oops! Function signature changed!
  }
}
```

---

## MongoDB Collections Quick Setup

Run this in MongoDB to create collections:

```javascript
db.createCollection('users', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['name', 'email', 'password', 'role'],
      properties: {
        _id: { bsonType: 'objectId' },
        name: { bsonType: 'string' },
        email: { bsonType: 'string' },
        password: { bsonType: 'string' },
        phone: { bsonType: 'string' },
        role: { enum: ['student', 'warden'] },
        createdAt: { bsonType: 'date' },
        updatedAt: { bsonType: 'date' }
      }
    }
  }
});

db.createCollection('rooms');
db.createCollection('complaints');
db.createCollection('payments');
db.createCollection('messages');
```

---

## Checklist for Each Migration

- [ ] Create backend endpoint
- [ ] Update service function in `lib/services/index.ts`
- [ ] Keep same function signature
- [ ] Test in browser (should work instantly!)
- [ ] Check localStorage for tokens
- [ ] Verify no console errors
- [ ] Move to next service

---

## Need Help?

Refer to `BACKEND_INTEGRATION_GUIDE.md` for detailed examples and full implementation guide.

The key principle: **Frontend components stay EXACTLY the same, only the services change!**

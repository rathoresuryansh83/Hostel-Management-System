# Notices Feature Implementation Guide

## What Changed

The "Messages" feature has been replaced with a "Notices" system that allows wardens to manage announcements and notices for the hostel.

## Overview

### Notice Features

**For Students:**
- View all hostel notices
- Search notices by title/content
- See notice details (title, content, category, priority, author, date)
- Read-only access (cannot create, edit, or delete)

**For Wardens:**
- Full CRUD operations on notices:
  - **Create** - Create new notices with title, content, category, and priority
  - **Read** - View all notices
  - **Update** - Edit existing notices
  - **Delete** - Delete notices
- Search notices
- Categorize notices (General, Maintenance, Event, Important, Emergency)
- Set priority levels (Low, Medium, High)
- Track author and creation date

## Files Changed/Created

### Created Files
1. **`/app/dashboard/notices/page.tsx`** - Main notices page with CRUD UI
2. **`/lib/services/mockData.ts`** - Added mock notices data
3. **`/NOTICES_IMPLEMENTATION.md`** - This file

### Modified Files
1. **`/lib/types/index.ts`** - Added Notice interface
2. **`/lib/services/index.ts`** - Added noticeService with CRUD methods
3. **`/components/layout/StudentSidebar.tsx`** - Changed "Messages" to "Notices"
4. **`/components/layout/WardenSidebar.tsx`** - Changed "Messages" to "Notices"
5. **`/lib/services/mockData.ts`** - Added mockNotices array with 5 sample notices

### Deleted Files
1. **`/app/dashboard/messages/page.tsx`** - Removed old messages page

## Notice Structure

```typescript
interface Notice {
  id: string;
  title: string;
  content: string;
  authorId: string;
  authorName: string;
  createdAt: Date;
  updatedAt: Date;
  expiresAt?: Date;
  priority: 'low' | 'medium' | 'high';
  category: 'general' | 'maintenance' | 'event' | 'important' | 'emergency';
}
```

## API Service Methods

The `noticeService` provides these methods:

```typescript
// Get all notices (sorted by newest first)
noticeService.getAllNotices(): Promise<Notice[]>

// Get a specific notice
noticeService.getNoticeById(id: string): Promise<Notice | undefined>

// Create a new notice (Warden only)
noticeService.createNotice(data: Omit<Notice, 'id' | 'createdAt' | 'updatedAt'>): Promise<Notice>

// Update a notice (Warden only)
noticeService.updateNotice(id: string, data: Partial<Notice>): Promise<Notice | undefined>

// Delete a notice (Warden only)
noticeService.deleteNotice(id: string): Promise<boolean>
```

## How to Use

### For Students
1. Navigate to "Notices" from the sidebar
2. View all available notices
3. Use the search bar to filter by title or content
4. Click on any notice to see full details

### For Wardens
1. Navigate to "Notices" from the sidebar
2. Click "New Notice" button
3. Fill in the form:
   - **Title** - Notice heading
   - **Content** - Full notice text
   - **Priority** - Select Low/Medium/High
   - **Category** - Select category from dropdown
4. Click "Create Notice"
5. To edit: Hover over notice and click the edit icon
6. To delete: Hover over notice and click the delete icon

## Frontend to API Mapping

When you connect to a real backend, replace these service calls:

```typescript
// In lib/services/index.ts

// Replace this:
export const noticeService = {
  getAllNotices: async (): Promise<Notice[]> => {
    // Currently returns mock data
  },
  // ... other methods
}

// With this:
export const noticeService = {
  getAllNotices: async (): Promise<Notice[]> => {
    const response = await fetch('/api/notices', {
      headers: { Authorization: `Bearer ${token}` }
    });
    return response.json();
  },
  
  createNotice: async (data) => {
    const response = await fetch('/api/notices', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${token}`
      },
      body: JSON.stringify(data),
    });
    return response.json();
  },
  
  updateNotice: async (id, data) => {
    const response = await fetch(`/api/notices/${id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${token}`
      },
      body: JSON.stringify(data),
    });
    return response.json();
  },
  
  deleteNotice: async (id) => {
    const response = await fetch(`/api/notices/${id}`, {
      method: 'DELETE',
      headers: { Authorization: `Bearer ${token}` }
    });
    return response.ok;
  }
}
```

## Backend API Endpoints to Create

When building your backend, create these endpoints:

### GET /api/notices
- Get all notices
- Returns: `Notice[]`
- Access: Both students and wardens

### POST /api/notices
- Create a new notice
- Requires: `{ title, content, priority, category }`
- Returns: Created `Notice`
- Access: Wardens only

### PUT /api/notices/:id
- Update a notice
- Requires: Partial notice fields to update
- Returns: Updated `Notice`
- Access: Wardens only

### DELETE /api/notices/:id
- Delete a notice
- Returns: Success boolean
- Access: Wardens only

## MongoDB Schema Example

```javascript
// notices.model.js
const noticeSchema = {
  title: String,
  content: String,
  authorId: String,
  authorName: String,
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now },
  expiresAt: Date,
  priority: {
    type: String,
    enum: ['low', 'medium', 'high'],
    default: 'medium'
  },
  category: {
    type: String,
    enum: ['general', 'maintenance', 'event', 'important', 'emergency'],
    default: 'general'
  }
}
```

## Role-Based Access Control

The page automatically shows different UIs based on user role:

- **Students**: Read-only view of all notices
- **Wardens**: Full CRUD interface with create/edit/delete buttons

Access is controlled in the component via:
```typescript
const isWarden = user?.role === 'warden';
```

## Sample Data Included

5 mock notices are pre-loaded:
1. Hostel Maintenance Schedule (High priority, Maintenance)
2. Annual Hostel Sports Day (Medium priority, Event)
3. Curfew Reminder (Medium priority, General)
4. WiFi Network Upgrade (High priority, General)
5. Hostel Fees Due (High priority, Important)

## Next Steps for Backend Integration

1. Create MongoDB Notice schema
2. Build Express routes for `/api/notices`
3. Add authentication middleware
4. Replace mock service calls with real API calls
5. Test CRUD operations from frontend

See `BACKEND_INTEGRATION_GUIDE.md` for detailed backend setup instructions.

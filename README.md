# HostelHub - Hostel Management System

A professional, fully functional hostel management dashboard built with Next.js, React, TypeScript, and modern UI components. The application provides separate dashboards for students and wardens with complete booking, room, complaint, and payment management features.

## Features

### Student Portal
- **Dashboard**: Quick overview of active bookings, complaints, and payment status
- **My Bookings**: View and manage room bookings with check-in/check-out status
- **Complaints**: Submit, track, and view responses to hostel-related complaints
- **Messages**: Direct communication with the warden
- **Payments**: Track payment history, pending dues, and download receipts
- **Profile**: Manage personal information and emergency contacts

### Warden Portal
- **Dashboard**: Complete hostel overview with KPIs, recent bookings, and room status
- **Bookings Management**: Manage all student bookings with check-in/out operations
- **Rooms Management**: View all rooms with occupancy status, amenities, and pricing
- **Students Management**: Access student profiles and details
- **Complaints Management**: Review and resolve student complaints
- **Analytics**: Detailed charts showing revenue trends, occupancy rates, and booking patterns
- **Reports**: Generate and download various management reports
- **Messages**: Communicate with students via integrated messaging

### Additional Features
- **AI-like Chatbot**: Floating assistant widget available throughout the application
- **Role-based Authentication**: Separate login for students and wardens
- **Mock Data**: Realistic demo data for testing all features
- **Responsive Design**: Fully mobile-responsive interface
- **Real-time Animations**: Smooth transitions powered by Framer Motion
- **Professional Styling**: Modern dark theme with Tailwind CSS

## Tech Stack

- **Frontend Framework**: Next.js 16 with App Router
- **Language**: TypeScript
- **Styling**: Tailwind CSS 4
- **UI Components**: ShadCN UI
- **Icons**: Lucide React
- **Animations**: Framer Motion
- **Data Visualization**: Recharts
- **Form Handling**: React Hook Form + Zod
- **Notifications**: Sonner Toast
- **State Management**: React Context API
- **Data Persistence**: MongoDB Atlas (Students/Rooms) + localStorage (session)

## Project Structure

```
app/
├── auth/
│   ├── login/
│   │   └── page.tsx              # Login page with role selection
│   └── layout.tsx                # Auth layout
├── dashboard/
│   ├── layout.tsx                # Dashboard layout with sidebars
│   ├── page.tsx                  # Dashboard home (role-based)
│   ├── bookings/
│   │   └── page.tsx              # Bookings management (both roles)
│   ├── rooms/
│   │   └── page.tsx              # Room management (Warden only)
│   ├── students/
│   │   └── page.tsx              # Student management (Warden only)
│   ├── complaints/
│   │   └── page.tsx              # Complaint management (both roles)
│   ├── messages/
│   │   └── page.tsx              # Messaging (both roles)
│   ├── analytics/
│   │   └── page.tsx              # Analytics dashboard (Warden only)
│   ├── payments/
│   │   └── page.tsx              # Payment tracking (both roles)
│   ├── profile/
│   │   └── page.tsx              # Profile management (Student only)
│   └── reports/
│       └── page.tsx              # Reports (Warden only)
├── layout.tsx                    # Root layout with AuthProvider
├── page.tsx                      # Root page (redirects to login)
└── globals.css                   # Global styles

components/
├── ChatbotWidget.tsx             # Floating chatbot assistant
├── ProtectedRoute.tsx            # Route protection wrapper
├── layout/
│   ├── Header.tsx                # Top navigation bar
│   ├── StudentSidebar.tsx        # Student navigation sidebar
│   └── WardenSidebar.tsx         # Warden navigation sidebar
├── dashboard/
│   ├── StudentDashboard.tsx      # Student home page
│   └── WardenDashboard.tsx       # Warden home page
└── ui/                           # ShadCN UI components

contexts/
└── AuthContext.tsx               # Global authentication state

lib/
├── types/
│   └── index.ts                  # TypeScript type definitions
├── services/
│   ├── index.ts                  # All mock services
│   └── mockData.ts               # Mock data for demo
└── utils.ts                      # Utility functions
```

## Getting Started

### Prerequisites
- Node.js 18+ 
- pnpm (or npm/yarn)

### Installation

1. **Install dependencies**:
```bash
pnpm install
```

2. **Create environment files**:

- **Backend**: create `backend/.env` (MongoDB Atlas SRV)

  Example:

  ```bash
  MONGO_URI="mongodb+srv://<username>:<password>@<cluster>.mongodb.net/?retryWrites=true&w=majority"
  MONGO_DB_NAME="hostelhub"
  PORT=5000
  ```

- **Frontend**: create `.env.local`

  ```bash
  NEXT_PUBLIC_API_URL="http://localhost:5000"
  ```

3. **Run both backend + frontend (recommended)**:

```bash
npm run dev:all
```

Or run them in separate terminals:

- Backend:

```bash
cd backend
npm start
```

- Frontend:

```bash
npm run dev
```

4. **Health check (validate Atlas connection)**:

Open `http://localhost:5000/health` and confirm:
- `ok: true`
- `mongo.dbName` matches `MONGO_DB_NAME`

### Atlas validation checklist (must-do)

- **IP allowlist**: In Atlas → Network Access → IP Access List → add `0.0.0.0/0` (allow all) temporarily.
- **Database name**: Set `MONGO_DB_NAME` in `backend/.env` (recommended) so the backend always uses the correct DB.
- **Connection string**: Use SRV format: `mongodb+srv://...`

5. **Open your browser** and navigate to `http://localhost:3000`

---

### Legacy note: one-time migration from localStorage

If you previously used localStorage for Students/Rooms, the app performs a **one-time migration** on first dashboard load:
- Reads old keys: `hostel.students.v1`, `hostel.rooms.v1`
- Bulk upserts into MongoDB (no duplicates)
- Clears old keys and marks migrated

If you want to re-run migration during testing, remove the flag:
- `localStorage["hostel.mongoMigrated.v1"]`

---

### Demo Credentials

#### Student Login
- **Email**: student@hostel.com
- **Password**: password

#### Warden Login
- **Email**: warden@hostel.com
- **Password**: password

Or use the **Demo Login** buttons on the login page for quick access.

---

### (Old) Run the development server (frontend only)
```bash
pnpm dev
```

## Key Components

### Authentication (`contexts/AuthContext.tsx`)
- Global auth state management using React Context
- localStorage persistence
- Role-based access control
- Demo user credentials

### Mock Services (`lib/services/index.ts`)
Fully functional mock services that simulate API behavior:
- `authService`: Login/logout
- `studentService`: Student profile management
- `roomService`: Room management
- `bookingService`: Booking CRUD operations
- `complaintService`: Complaint management
- `messageService`: Messaging
- `analyticsService`: Analytics data
- `paymentService`: Payment tracking

### Protected Routes
Automatically redirect unauthenticated users to login page and enforce role-based access.

### Chatbot Widget
Interactive floating assistant with:
- Pre-defined helpful responses
- Quick action buttons
- Message history
- Smooth animations

## Backend Integration Guide

This project comes with **3 comprehensive guides** to help you add a real backend:

### 📖 Documentation Files
1. **`API_QUICK_REFERENCE.md`** - Quick reference for where to add API calls (START HERE!)
2. **`BACKEND_INTEGRATION_GUIDE.md`** - Complete step-by-step backend setup with MongoDB
3. **`ARCHITECTURE.md`** - Visual diagrams and data flow explanations

### Migrating to a Real Backend

The project is designed for easy backend integration. Follow these steps:

1. **Update Service Layer** (`lib/services/index.ts`):
   - Replace mock functions with actual API calls
   - Keep the same function signatures for zero frontend changes
   
   Example:
   ```typescript
   // Before (mock)
   export const authService = {
     login: async (email: string, password: string) => {
       await delay();
       // mock logic
     }
   }

   // After (real API)
   export const authService = {
     login: async (email: string, password: string) => {
       const response = await fetch('/api/auth/login', {
         method: 'POST',
         body: JSON.stringify({ email, password })
       });
       return response.json();
     }
   }
   ```

2. **Create API Endpoints** (Node.js/Express):
   ```
   POST   /api/auth/login
   POST   /api/auth/logout
   GET    /api/bookings
   POST   /api/bookings
   PUT    /api/bookings/:id
   GET    /api/rooms
   GET    /api/students
   POST   /api/complaints
   GET    /api/analytics
   ```

3. **Update Authentication**:
   - Replace localStorage simulation with JWT tokens
   - Implement httpOnly cookies for security
   - Add token refresh logic

4. **Database Schema**:
   ```sql
   users (id, name, email, phone, role, created_at)
   students (id, user_id, roll_number, department, semester, ...)
   wardens (id, user_id, department, join_date)
   bookings (id, student_id, room_id, check_in, check_out, status, ...)
   rooms (id, room_number, floor, capacity, type, status, ...)
   complaints (id, student_id, category, title, description, status, ...)
   messages (id, sender_id, recipient_id, content, timestamp)
   payments (id, booking_id, student_id, amount, status, ...)
   ```

5. **Recommended Backend Stack**:
   - **Runtime**: Node.js
   - **Framework**: Express.js, NestJS, or Fastify
   - **Database**: PostgreSQL + Prisma ORM
   - **Authentication**: JWT + bcryptjs
   - **Hosting**: Vercel, Railway, Render, or AWS

## Customization

### Colors & Styling
Edit Tailwind classes in components. Current color scheme:
- Primary: Blue (#0ea5e9)
- Secondary: Teal (#06b6d4)
- Background: Slate-900
- Accent: Green, Yellow, Red

### Mock Data
Update `lib/services/mockData.ts` to change:
- Student/warden profiles
- Room configurations
- Booking data
- Complaint examples

### Animations
Adjust animation timings in Framer Motion components (`motion.div`, `transition` props)

### Chatbot Responses
Customize bot responses in `components/ChatbotWidget.tsx` - update the `botResponses` object with your own Q&A pairs.

## Performance Optimization

- ✅ Image optimization with Next.js Image
- ✅ Code splitting with dynamic imports
- ✅ Optimized animations with Framer Motion
- ✅ Component lazy loading
- ✅ Efficient state management with Context API

## Security Considerations

- ✅ Protected routes with authentication checks
- ✅ Role-based access control
- ✅ Input validation with Zod
- ✅ XSS protection via React's built-in escaping
- ✅ CSRF protection ready (implement when adding backend)

When deploying with a real backend, add:
- HTTPS only
- httpOnly cookies for tokens
- CORS configuration
- Rate limiting
- Input sanitization
- Database query parameterization

## Deployment

### Deploy to Vercel
```bash
# Push to GitHub first
git init
git add .
git commit -m "Initial commit"
git push -u origin main

# Then deploy via Vercel dashboard or CLI
vercel
```

### Environment Variables
No environment variables required for frontend-only mode. When adding backend, create `.env.local`:
```
NEXT_PUBLIC_API_URL=http://localhost:3001
NEXT_PUBLIC_APP_NAME=HostelHub
```

## Troubleshooting

### Port Already in Use
```bash
# Change port
pnpm dev -- -p 3001
```

### Module Not Found
```bash
# Clear node_modules and reinstall
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

### Styling Issues
```bash
# Rebuild Tailwind CSS
pnpm build
```

## Future Enhancements

- Real-time notifications with WebSockets
- Email notifications for bookings/complaints
- SMS alerts for important updates
- Advanced analytics and reporting
- Calendar view for bookings
- Guest check-in QR codes
- Payment gateway integration (Stripe, Razorpay)
- File upload for complaint attachments
- Bulk operations and import/export
- Multi-language support
- Dark/light theme toggle

## License

This project is open source and available under the MIT License.

## Support

For issues, feature requests, or questions, please refer to the documentation or contact support.

---

**Built with ❤️ for hostel management excellence**

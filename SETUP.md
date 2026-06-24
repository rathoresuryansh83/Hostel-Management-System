# Quick Setup Guide

## Installation & Running

1. **Install dependencies:**
   ```bash
   pnpm install
   ```

2. **Start development server:**
   ```bash
   pnpm dev
   ```

3. **Open in browser:**
   Navigate to `http://localhost:3000`

## First Time Users

### Quick Login
Click the **Demo Login** buttons on the login page to instantly access either:
- **Student Dashboard** (student@hostel.com)
- **Warden Dashboard** (warden@hostel.com)

Password for both: `password`

## What's Included

### Complete Feature Set
✅ Role-based Authentication (Student/Warden)
✅ Student Bookings Management
✅ Complaint System
✅ Payment Tracking
✅ Room Inventory
✅ Student Profiles
✅ Analytics & Charts
✅ Messaging System
✅ AI Chatbot Widget
✅ Reports & Analytics
✅ Responsive Design
✅ Dark Theme UI

### Tech Stack Highlights
- Next.js 16 with App Router
- TypeScript for type safety
- Tailwind CSS for styling
- ShadCN UI components
- Recharts for visualizations
- Framer Motion for animations
- React Context for state
- localStorage for persistence

## Project Structure at a Glance

```
app/                           # Next.js App Router pages
├── auth/login/                # Login page
├── dashboard/                 # All dashboard pages
│   ├── page.tsx              # Home (role-based)
│   ├── bookings/             # Booking management
│   ├── rooms/                # Room management (Warden)
│   ├── students/             # Student list (Warden)
│   ├── complaints/           # Complaint handling
│   ├── messages/             # Messaging
│   ├── analytics/            # Charts & Analytics (Warden)
│   ├── payments/             # Payment tracking
│   ├── profile/              # Profile (Student)
│   └── reports/              # Reports (Warden)
└── layout.tsx                # Root layout with providers

components/                    # React components
├── ChatbotWidget.tsx         # Floating chatbot
├── ProtectedRoute.tsx        # Auth protection
├── layout/                   # Navigation components
├── dashboard/                # Dashboard components
└── ui/                       # ShadCN UI components

contexts/                      # React Context providers
└── AuthContext.tsx           # Authentication state

lib/                          # Utilities & services
├── types/                    # TypeScript definitions
├── services/                 # Mock API services
└── utils.ts                  # Helper functions
```

## Key Features Explained

### Authentication
- Unified login page for students and wardens
- Role-based dashboard routing
- localStorage session persistence
- Demo credentials for quick access

### Student Features
- View personal bookings
- Submit and track complaints
- Check payment status
- Direct messaging with warden
- Update profile information

### Warden Features
- Manage all bookings (check-in/out)
- Room availability tracking
- Student directory and profiles
- Complaint resolution system
- Revenue and occupancy analytics
- Complaint management dashboard

### Chatbot
- Available on all pages
- Context-aware helpful responses
- Quick action buttons
- Smooth animations

## Demo Data

All data is pre-populated for testing:
- 4 sample students
- 1 warden account
- 5 rooms with different types
- Multiple bookings in different statuses
- Sample complaints
- Payment records

## Customization Tips

### Change Colors
Edit `bg-blue-600`, `text-teal-400`, etc. in components. Currently using:
- Blue for student portal
- Teal for warden portal
- Slate for backgrounds

### Modify Mock Data
Edit `lib/services/mockData.ts` to:
- Add more students/wardens
- Change room configurations
- Update booking scenarios
- Customize complaint examples

### Adjust Animations
Look for `motion.div`, `transition`, `variants` in components. Modify timing with:
```typescript
transition={{ duration: 0.3 }} // Change duration
initial={{ opacity: 0 }}        // Change initial state
animate={{ opacity: 1 }}        // Change final state
```

## Adding Backend

### When Ready to Connect Real API

1. **Update Services** (`lib/services/index.ts`):
   ```typescript
   // Replace mock calls with fetch
   export const authService = {
     login: async (email: string, password: string) => {
       const res = await fetch('YOUR_API/auth/login', {
         method: 'POST',
         body: JSON.stringify({ email, password })
       });
       return res.json();
     }
   }
   ```

2. **Create Backend Endpoints**:
   - POST `/auth/login`
   - GET `/bookings`
   - POST `/bookings`
   - GET `/students`
   - POST `/complaints`
   - etc.

3. **Database Tables**:
   - users, students, wardens
   - bookings, rooms, payments
   - complaints, messages

See README.md for detailed backend integration guide.

## File Sizes & Performance

- Total bundle size: ~200KB (optimized)
- No external image CDNs (self-contained)
- Lazy-loaded components
- Optimized Recharts charts

## Deployment

### Vercel (Recommended)
1. Push code to GitHub
2. Import project in Vercel
3. Click Deploy
4. Done! Auto-deploys on git push

### Docker
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN pnpm install
RUN pnpm build
CMD ["pnpm", "start"]
```

## Troubleshooting

### Page Won't Load
- Check browser console for errors
- Ensure you're logged in
- Try clearing localStorage: `localStorage.clear()`

### Styles Look Wrong
- Clear Tailwind cache: Delete `.next` folder
- Reinstall: `rm -rf node_modules && pnpm install`

### Need Help?
- Check README.md for detailed documentation
- Review component code for examples
- Check console.log outputs for data flow

## Browser Support

- Chrome/Chromium (recommended)
- Firefox
- Safari
- Edge

## Performance Tips

For best experience:
- Use Chrome DevTools to inspect network
- Check Performance tab for bottlenecks
- Unused code is automatically removed by Next.js
- All animations are GPU-accelerated

## What to Try First

1. **Login** - Use demo credentials or student/warden buttons
2. **Explore Dashboard** - See the role-specific home page
3. **Create Complaint** - Try the complaint submission flow
4. **Check Analytics** (Warden) - View charts and trends
5. **Open Chatbot** - Click the message icon (bottom right)

## Next Steps

After exploring:
1. Read full README.md for detailed features
2. Explore component code to understand architecture
3. Plan your backend integration
4. Customize colors, data, and content
5. Deploy to production

---

**Happy Hosting! 🚀**

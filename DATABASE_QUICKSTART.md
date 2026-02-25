# Database Quick Start Guide

## 🚀 Get Started in 5 Steps

### Step 1: Update Environment Variables
Create `.env.local` in the project root:
```env
MONGODB_URI=mongodb://localhost:27017/finsight
MONGODB_DB_NAME=finsight
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production
DEMO_EMAIL=demo@finsight.app
DEMO_PASSWORD=demo12345
```

### Step 2: Start MongoDB
**Option A - Docker (Easiest):**
```bash
docker-compose up -d
```

**Option B - Local MongoDB:**
```bash
# Windows: MongoDB should auto-start
# Or manually start MongoDB service
mongod
```

### Step 3: Seed Demo Data
```bash
node scripts/db-init.js seed
```

Output:
```
✓ Connected to MongoDB
📊 Seeding demo data...
✓ Created demo user: demo@finsight.app
✓ Created budget
✓ Created 3 goals
✓ Created 46 transactions

✅ Demo data seeded successfully!

Login credentials:
  Email: demo@finsight.app
  Password: demo12345
```

### Step 4: Start Development Server
```bash
npm run dev
```

### Step 5: Test the Database

**Login with Demo Account:**
```bash
curl -X POST http://localhost:3000/api/demo/login
```

**Or Signup New User:**
```bash
curl -X POST http://localhost:3000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Your Name",
    "email": "your@email.com",
    "password": "securepass123"
  }'
```

---

## 📝 Database Management Commands

### List All Users
```bash
node scripts/db-init.js list-users
```

### Create New User
```bash
node scripts/db-init.js create-user john@example.com password123 "John Doe"
```

### Delete User & Data
```bash
node scripts/db-init.js delete-user john@example.com
```

### Reset Database (Clear All)
```bash
node scripts/db-init.js reset
```

### Re-seed Demo Data
```bash
node scripts/db-init.js reset
node scripts/db-init.js seed
```

---

## 🔍 Test API Endpoints

### Authentication APIs

**Signup:**
```bash
curl -X POST http://localhost:3000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Jane Smith",
    "email": "jane@example.com",
    "password": "securepass123"
  }'
```

**Login:**
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "jane@example.com",
    "password": "securepass123"
  }'
```

**Get Current User (requires auth cookie):**
```bash
curl http://localhost:3000/api/auth/me
```

### Transaction APIs

**List Transactions:**
```bash
curl http://localhost:3000/api/transactions
```

**Create Transaction:**
```bash
curl -X POST http://localhost:3000/api/transactions \
  -H "Content-Type: application/json" \
  -d '{
    "type": "expense",
    "amount": 45.50,
    "category": "Food",
    "merchant": "Cafe Coffee",
    "note": "Morning coffee",
    "occurredAt": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
  }'
```

### Dashboard APIs

**Get Dashboard Summary:**
```bash
curl http://localhost:3000/api/dashboard/summary
```

**Get Financial Insights:**
```bash
curl http://localhost:3000/api/insights
```

---

## 🗄️ MongoDB Management

### Using MongoDB Compass (GUI - Easiest)
1. Download: https://www.mongodb.com/products/tools/compass
2. Connect to: `mongodb://localhost:27017`
3. Browse collections visually

### Using MongoDB Shell
```bash
# Connect
mongosh

# View databases
show dbs

# Use finsight database
use finsight

# View collections
show collections

# Find users
db.users.find()

# Find transactions for a user
db.transactions.findOne()
db.transactions.find({ userId: ObjectId("...") })

# Count records
db.users.countDocuments()
db.transactions.countDocuments()

# Export data
mongoexport --db finsight --collection users --out users.json
```

---

## 🔐 Security Checklist

- [x] Password hashing with bcryptjs (12 rounds)
- [x] JWT tokens with 7-day expiration
- [x] HttpOnly cookies to prevent XSS
- [x] SameSite cookies for CSRF protection
- [ ] Change `JWT_SECRET` in production (IMPORTANT!)
- [ ] Use HTTPS in production
- [ ] Enable MongoDB authentication
- [ ] Use MongoDB Atlas with VPC for cloud

---

## 📊 Database Size & Performance

### Current Collections
- **users**: ~10s-100s of documents
- **transactions**: ~1000s per active user
- **budgets**: ~12 per user per year
- **goals**: ~5-10 per user
- **notifications**: ~100s per user

### Optimization Tips
1. Indexes are auto-created on startup
2. Use pagination for large result sets
3. Lean queries for read-only operations
4. Archive old transactions periodically

---

## ⚠️ Common Issues & Solutions

### "MongoDB connection refused"
```bash
# Check if MongoDB is running
mongosh --host localhost:27017

# If not, start MongoDB
docker-compose up -d
# or
mongod
```

### "MONGODB_URI is not set"
```bash
# Create .env.local
MONGODB_URI=mongodb://localhost:27017/finsight
```

### "Demo user not seeded"
```bash
node scripts/db-init.js seed
```

### "Email already in use"
```bash
# Either use different email or delete existing user
node scripts/db-init.js delete-user old@email.com
```

### "Duplicate key error"
```bash
# Drop all collections
node scripts/db-init.js reset
node scripts/db-init.js seed
```

---

## 🎯 Next Features to Add

1. **Audit Logging** - Track all user actions
2. **Data Export** - CSV/PDF export functionality
3. **Backup Automation** - Daily/weekly backups
4. **User Preferences** - Language, timezone, theme
5. **Recurring Transactions** - Auto-bill functionality
6. **Alerts** - Budget exceeded, goal progress notifications
7. **Multi-currency** - Convert between currencies
8. **Sharing** - Share financial reports with advisors
9. **Photo Receipts** - Upload transaction receipts
10. **OAuth Integration** - Google/GitHub login

---

## 📚 Resources

- [MongoDB Documentation](https://docs.mongodb.com/)
- [Mongoose Documentation](https://mongoosejs.com/)
- [JWT.io](https://jwt.io/)
- [bcryptjs Reference](https://github.com/dcodeIO/bcrypt.js)

---

## 💡 Architecture Overview

```
┌─────────────────────────────────────────┐
│         Next.js Frontend                │
│   (React Components + Styling)          │
└─────────────────┬───────────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────┐
│      Next.js API Routes                 │
│   (Authentication, Data Operations)     │
└─────────────────┬───────────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────┐
│      Mongoose ODM Layer                 │
│   (User, Transaction, Budget, etc.)     │
└─────────────────┬───────────────────────┘
                  │
                  ↓
┌─────────────────────────────────────────┐
│         MongoDB Database                │
│      (Multi-user Data Storage)          │
└─────────────────────────────────────────┘
```

---

**Happy Coding! 🎉** Your database is ready for production use.

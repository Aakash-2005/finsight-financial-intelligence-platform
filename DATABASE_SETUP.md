# Database Setup Guide for HackMor (FinSight)

## Overview
Your application uses **MongoDB** with **Mongoose** ODM for storing user data and related financial information. The database supports multiple users with complete data isolation.

## Database Architecture

### Models Implemented

1. **User** (`models/User.ts`)
   - Stores user authentication and profile data
   - Fields: name, email, passwordHash, currency, cashBalance, creditLimit, creditBalance
   - Unique index on email

2. **Transaction** (`models/Transaction.ts`)
   - Records all financial transactions (income/expense)
   - Fields: userId, type, amount, category, merchant, note, occurredAt, tags
   - Indexes: userId + occurredAt (for efficient queries)

3. **Budget** (`models/Budget.ts`)
   - Monthly budget allocations per category
   - Fields: userId, month, totalBudget, categoryBudgets
   - Unique index on userId + month

4. **Goal** (`models/Goal.ts`)
   - User financial goals tracking
   - Fields: userId, title, targetAmount, currentAmount, deadline, status
   - Index: userId + status

5. **Notification** (`models/Notification.ts`)
   - System notifications for users
   - Fields: userId, severity, title, message, kind, readAt
   - Index: userId + createdAt

## Setup Instructions

### 1. Environment Configuration

Create or update `.env.local` file:

```env
# MongoDB Connection
MONGODB_URI=mongodb://localhost:27017/finsight
MONGODB_DB_NAME=finsight

# JWT Authentication
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production

# Demo Account (optional)
DEMO_EMAIL=demo@finsight.app
DEMO_PASSWORD=demo12345
```

### 2. MongoDB Installation Options

#### Option A: Local MongoDB (Recommended for Development)
```bash
# Windows with Chocolatey
choco install mongodb-community

# Or download from: https://www.mongodb.com/try/download/community
```

#### Option B: Docker (Recommended)
```bash
# Using docker-compose.yml in your project
docker-compose up -d

# Or manual Docker command
docker run -d -p 27017:27017 --name mongodb mongo:latest
```

#### Option C: MongoDB Atlas (Cloud)
1. Go to https://www.mongodb.com/cloud/atlas
2. Create account and cluster
3. Update `MONGODB_URI` with connection string
4. Example: `mongodb+srv://username:password@cluster.mongodb.net/finsight`

### 3. Initialize Database

#### Seed Demo Data
```bash
# With demo user, sample transactions, budgets, and goals
curl -X POST http://localhost:3000/api/demo/seed

# Reset demo data (delete existing)
curl -X POST http://localhost:3000/api/demo/seed?reset=1
```

#### Login with Demo Account
```bash
# After seeding
curl -X POST http://localhost:3000/api/demo/login
```

## API Routes Using Database

### Authentication
- `POST /api/auth/signup` - Create new user account
- `POST /api/auth/login` - Login with email/password
- `POST /api/auth/logout` - Clear auth cookie
- `GET /api/auth/me` - Get current user profile

### Transactions
- `GET /api/transactions` - List user transactions
- `POST /api/transactions` - Create transaction
- `PUT /api/transactions/[id]` - Update transaction
- `DELETE /api/transactions/[id]` - Delete transaction

### Budgets
- `GET /api/budgets` - Get budgets
- `POST /api/budgets` - Create/update budget

### Goals
- `GET /api/goals` - List goals
- `POST /api/goals` - Create goal
- `PUT /api/goals/[id]` - Update goal

### Other Features
- `GET /api/dashboard/summary` - Get dashboard data
- `GET /api/insights` - Generate financial insights
- `GET /api/notifications` - Get notifications

## Database Relationships

```
User (1) ──── (Many) Transactions
User (1) ──── (Many) Budgets
User (1) ──── (Many) Goals
User (1) ──── (Many) Notifications
```

All financial data is properly isolated per user through the `userId` field.

## Common Tasks

### Add New User via API
```bash
curl -X POST http://localhost:3000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "securepassword123"
  }'
```

### Query Users (MongoDB)
```javascript
# Using MongoDB shell or compass
db.users.find({})
db.users.findOne({ email: "demo@finsight.app" })
db.transactions.find({ userId: ObjectId("...") })
```

### Monitor Database
```bash
# Using MongoDB Compass (GUI)
# Connection URI: mongodb://localhost:27017
# Or use shell
mongosh --host localhost --port 27017
```

## Database Backup

### Local Backup
```bash
mongodump --uri="mongodb://localhost:27017/finsight" --out=./backup
mongorestore --uri="mongodb://localhost:27017/finsight" ./backup
```

### Docker Backup
```bash
docker exec mongodb mongodump --out=/dump
docker cp mongodb:/dump ./local_backup
```

## Troubleshooting

### MongoDB Connection Error
```
Error: Missing env var: MONGODB_URI
```
**Solution**: Ensure `.env.local` has valid MONGODB_URI

### Demo User Not Found
```
Error: Demo user not seeded yet. Call /api/demo/seed first.
```
**Solution**: 
```bash
curl -X POST http://localhost:3000/api/demo/seed
```

### Duplicate Key Error
```
E11000 duplicate key error collection: finsight.users index: email_1
```
**Solution**: Email already exists. Use different email or drop collection:
```bash
db.users.deleteMany({})
```

## Performance Optimization

### Indexes Created Automatically
- User: email (unique)
- Transaction: userId + occurredAt
- Budget: userId + month (unique)
- Goal: userId + status
- Notification: userId + createdAt

### Pagination Example
```typescript
// In API routes
const pageSize = 10
const page = req.query.page || 1
const skip = (page - 1) * pageSize

const transactions = await Transaction.find({ userId })
  .sort({ occurredAt: -1 })
  .skip(skip)
  .limit(pageSize)
```

## Security Notes

1. **JWT Secret**: Change `JWT_SECRET` in production
2. **Password Hashing**: Uses bcryptjs (12 salt rounds)
3. **HTTPS**: Enable in production (`secure: true` in cookie)
4. **CSRF**: Implemented via SameSite cookies
5. **Data Validation**: Zod schemas validate all inputs

## Next Steps

1. ✅ Database configured with MongoDB
2. ✅ Authentication integrated with database
3. ✅ All models properly structured for scalability
4. ✅ Demo data seeding available
5. 📌 Consider: Add database migrations tool (e.g., migration scripts)
6. 📌 Consider: Add data export/import features
7. 📌 Consider: Implement audit logging for security

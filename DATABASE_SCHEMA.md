# Database Schema & Architecture

## 📊 Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                          USER                                  │
│  ┌───────────────────────────────────────────────────────┐   │
│  │ _id: ObjectId (Primary Key)                           │   │
│  │ name: String                                          │   │
│  │ email: String (Unique Index)                          │   │
│  │ passwordHash: String (Hashed with bcryptjs)           │   │
│  │ currency: String (Default: "INR")                     │   │
│  │ cashBalance: Number (Default: 12000)                 │   │
│  │ creditLimit: Number (Default: 5000)                  │   │
│  │ creditBalance: Number (Default: 1200)                │   │
│  │ createdAt: Date                                       │   │
│  │ updatedAt: Date                                       │   │
│  └───────────────────────────────────────────────────────┘   │
│           │           │           │         │                │
│           │           │           │         └─────────┐      │
│           ↓           ↓           ↓                   ↓      │
│    TRANSACTION   BUDGET      GOALS           NOTIFICATION   │
│         (1:N)     (1:N)      (1:N)                 (1:N)     │
│           │         │         │                     │       │
└─────────────────────────────────────────────────────────────────┘
```

## 📋 Detailed Schema

### 1. **User Collection**

```typescript
{
  _id: ObjectId,
  name: string,                    // User's full name
  email: string,                   // Unique email (indexed)
  passwordHash: string,            // bcryptjs hashed password
  currency: string,                // "INR", "USD", etc.
  cashBalance: number,             // Cash/checking balance
  creditLimit: number,             // Credit card limit
  creditBalance: number,           // Credit card balance
  createdAt: Date,                 // Auto-generated
  updatedAt: Date                  // Auto-generated
}

// Indexes
{ email: 1 }  // UNIQUE INDEX
```

**Example Document:**
```json
{
  "_id": ObjectId("65a1234567890abcdef12345"),
  "name": "John Doe",
  "email": "john@example.com",
  "passwordHash": "$2a$12$...",
  "currency": "INR",
  "cashBalance": 50000,
  "creditLimit": 10000,
  "creditBalance": 3000,
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-01-20T15:45:30Z"
}
```

---

### 2. **Transaction Collection**

```typescript
{
  _id: ObjectId,
  userId: ObjectId,                // Reference to User
  type: "income" | "expense",      // Transaction type
  amount: number,                  // Amount in cents/paise
  category: string,                // "Food", "Transport", etc.
  merchant: string,                // Where transaction occurred
  note: string,                    // Optional user note
  occurredAt: Date,                // When transaction happened
  tags: string[],                  // Optional tags for filtering
  createdAt: Date,                 // Auto-generated
  updatedAt: Date                  // Auto-generated
}

// Indexes
{ userId: 1, occurredAt: -1 }        // For queries
{ userId: 1, type: 1, occurredAt: -1 } // For filtered queries
```

**Example Document:**
```json
{
  "_id": ObjectId("65a1234567890abcdef12346"),
  "userId": ObjectId("65a1234567890abcdef12345"),
  "type": "expense",
  "amount": 4500,
  "category": "Food",
  "merchant": "Starbucks Coffee",
  "note": "Coffee with team",
  "occurredAt": "2024-01-20T09:30:00Z",
  "tags": ["work", "coffee"],
  "createdAt": "2024-01-20T09:31:00Z",
  "updatedAt": "2024-01-20T09:31:00Z"
}
```

---

### 3. **Budget Collection**

```typescript
{
  _id: ObjectId,
  userId: ObjectId,                // Reference to User
  month: string,                   // "2024-01" format (YYYY-MM)
  totalBudget: number,             // Total monthly budget
  categoryBudgets: {               // Per-category budgets
    "Food": number,
    "Transport": number,
    "Shopping": number,
    // ... more categories
  },
  createdAt: Date,                 // Auto-generated
  updatedAt: Date                  // Auto-generated
}

// Indexes (UNIQUE)
{ userId: 1, month: 1 }
```

**Example Document:**
```json
{
  "_id": ObjectId("65a1234567890abcdef12347"),
  "userId": ObjectId("65a1234567890abcdef12345"),
  "month": "2024-01",
  "totalBudget": 50000,
  "categoryBudgets": {
    "Food": 8000,
    "Transport": 5000,
    "Shopping": 10000,
    "Bills": 15000,
    "Entertainment": 5000,
    "Utilities": 2000,
    "Groceries": 5000
  },
  "createdAt": "2024-01-01T00:00:00Z",
  "updatedAt": "2024-01-20T10:30:00Z"
}
```

---

### 4. **Goal Collection**

```typescript
{
  _id: ObjectId,
  userId: ObjectId,                // Reference to User
  title: string,                   // Goal name
  targetAmount: number,            // Goal target amount
  currentAmount: number,           // Current progress
  deadline: Date,                  // Target completion date
  status: "active" | "completed" | "paused",
  createdAt: Date,                 // Auto-generated
  updatedAt: Date                  // Auto-generated
}

// Indexes
{ userId: 1, status: 1 }
```

**Example Document:**
```json
{
  "_id": ObjectId("65a1234567890abcdef12348"),
  "userId": ObjectId("65a1234567890abcdef12345"),
  "title": "Emergency Fund",
  "targetAmount": 200000,
  "currentAmount": 150000,
  "deadline": "2024-12-31T23:59:59Z",
  "status": "active",
  "createdAt": "2024-01-01T10:30:00Z",
  "updatedAt": "2024-01-20T15:45:30Z"
}
```

---

### 5. **Notification Collection**

```typescript
{
  _id: ObjectId,
  userId: ObjectId,                // Reference to User
  severity: "info" | "success" | "warning" | "danger",
  title: string,                   // Notification title
  message: string,                 // Notification message
  kind: string,                    // "system", "alert", etc.
  readAt: Date | null,             // When user read it (null if unread)
  createdAt: Date,                 // Auto-generated
  updatedAt: Date                  // Auto-generated
}

// Indexes
{ userId: 1, createdAt: -1 }
```

**Example Document:**
```json
{
  "_id": ObjectId("65a1234567890abcdef12349"),
  "userId": ObjectId("65a1234567890abcdef12345"),
  "severity": "warning",
  "title": "Budget Limit Reached",
  "message": "You've reached 95% of your Food budget for January",
  "kind": "system",
  "readAt": null,
  "createdAt": "2024-01-20T08:15:00Z",
  "updatedAt": "2024-01-20T08:15:00Z"
}
```

---

## 🔗 Relationships & Foreign Keys

### User → Transaction (1:N)
```sql
-- In SQL terms:
SELECT * FROM transactions 
WHERE userId = user._id
```

### User → Budget (1:N)
```sql
-- One user can have multiple budgets (one per month)
SELECT * FROM budgets 
WHERE userId = user._id AND month = "2024-01"
```

### User → Goal (1:N)
```sql
-- One user can have multiple active goals
SELECT * FROM goals 
WHERE userId = user._id AND status = "active"
```

### User → Notification (1:N)
```sql
-- One user can have many notifications
SELECT * FROM notifications 
WHERE userId = user._id 
ORDER BY createdAt DESC
```

---

## 📈 Indexes & Query Performance

| Collection       | Index                          | Purpose                      |
|-----------------|--------------------------------|------------------------------|
| **users**       | `{ email: 1 }`                 | Fast email lookup (UNIQUE)   |
| **transactions**| `{ userId: 1, occurredAt: -1 }` | Get user's transactions      |
| **transactions**| `{ userId: 1, type: 1, occurredAt: -1 }` | Filter by type |
| **budgets**     | `{ userId: 1, month: 1 }`     | Get monthly budget (UNIQUE)  |
| **goals**       | `{ userId: 1, status: 1 }`    | Get active goals             |
| **notifications** | `{ userId: 1, createdAt: -1 }` | Get recent notifications     |

---

## 🔐 Data Isolation

Each document has a `userId` field pointing to the User collection:

```
User A (id: 123)
├── Transactions (userId: 123)
├── Budgets (userId: 123)
├── Goals (userId: 123)
└── Notifications (userId: 123)

User B (id: 456)
├── Transactions (userId: 456)
├── Budgets (userId: 456)
├── Goals (userId: 456)
└── Notifications (userId: 456)
```

**Complete data isolation** - Users cannot access each other's data.

---

## 📊 Sample Size Estimates

### For 1,000 Active Users:
- **Users**: 1,000 documents (~1 KB each) = 1 MB
- **Transactions**: 100,000 documents (~500 B each) = 50 MB
- **Budgets**: 12,000 documents (~500 B each) = 6 MB
- **Goals**: 5,000 documents (~300 B each) = 1.5 MB
- **Notifications**: 50,000 documents (~300 B each) = 15 MB

**Total**: ~73.5 MB (very compact!)

### Scaling Considerations:
- At 1M users: ~73 GB (still manageable)
- Create sharding strategy by userId
- Archive old transactions (older than 2 years)
- Use TTL indexes for auto-deleting old notifications

---

## 🔄 Migration Planning

### Future Schema Changes:
```javascript
// Add new field to all users
db.users.updateMany({}, { $set: { phoneVerified: false } })

// Rename field
db.transactions.updateMany({}, { $rename: { occuredAt: occurredAt } })

// Add array field
db.users.updateMany({}, { $set: { tags: [] } })

// Create new index
db.transactions.createIndex({ userId: 1, category: 1 })
```

---

## 🛡️ Backup Strategy

### Full Backup
```bash
mongodump --uri="mongodb://localhost:27017/finsight" --archive=backup.archive
```

### Restore Backup
```bash
mongorestore --uri="mongodb://localhost:27017/finsight" --archive=backup.archive
```

### Automated Daily Backup (Cron Job)
```bash
# 2 AM daily backup
0 2 * * * mongodump --uri="mongodb://localhost:27017/finsight" --archive=/backups/finsight-$(date +\%Y-\%m-\%d).archive
```

---

## 📝 Documentation References

- [MongoDB BSON Limits](https://docs.mongodb.com/manual/reference/limits/)
- [Mongoose Schema Types](https://mongoosejs.com/docs/schematypes.html)
- [Index Best Practices](https://docs.mongodb.com/manual/applications/indexes/)

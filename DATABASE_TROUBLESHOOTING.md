# Database Troubleshooting Guide

## 🔧 Common Issues & Solutions

### 1. MongoDB Connection Issues

#### Problem: `Error: connect ECONNREFUSED 127.0.0.1:27017`

**Cause**: MongoDB service is not running

**Solutions**:

**If using Docker**:
```bash
# Check if container is running
docker ps | grep mongodb

# If not, start it
docker-compose up -d mongodb

# If doesn't exist, create it
docker run -d -p 27017:27017 --name mongodb mongo:latest
```

**If using local MongoDB**:
```bash
# Windows - Start MongoDB service
# Method 1: Services UI
# Search "Services" → Find "MongoDB Server" → Start

# Method 2: Command line
net start MongoDB

# Method 3: Manually run mongod
mongod
```

**If using MongoDB Atlas**:
```
Check your connection string:
- Correct username/password
- IP address whitelisted
- Database name correct
```

---

#### Problem: `Error: Missing env var: MONGODB_URI`

**Solution**: Add to `.env.local`
```env
MONGODB_URI=mongodb://localhost:27017/finsight
MONGODB_DB_NAME=finsight
```

---

### 2. Authentication Issues

#### Problem: `Error: Missing env var: JWT_SECRET`

**Solution**: Add to `.env.local`
```env
JWT_SECRET=your-super-secret-key-change-this-in-production
```

Generate a secure secret:
```bash
# Using OpenSSL
openssl rand -base64 32

# Using Node
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

---

#### Problem: `JsonWebTokenError: invalid token`

**Cause**: Token is malformed or tampered

**Solutions**:
1. Clear browser cookies and re-login
2. Check JWT_SECRET is same in `.env.local`
3. Verify cookie is being sent in requests

```bash
# Test token manually
curl -X GET http://localhost:3000/api/auth/me \
  -H "Cookie: finsight_token=YOUR_TOKEN"
```

---

### 3. User & Authentication

#### Problem: `Error: Email already in use`

**Cause**: User with this email already exists

**Solutions**:
```bash
# Option 1: Use different email
curl -X POST http://localhost:3000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John",
    "email": "different@email.com",
    "password": "secure123"
  }'

# Option 2: Delete existing user
node scripts/db-init.js delete-user old@email.com

# Option 3: Check existing users
node scripts/db-init.js list-users
```

---

#### Problem: `Invalid credentials` on login

**Cause**: Wrong email or password

**Solutions**:
```bash
# Check if user exists
node scripts/db-init.js list-users

# Try demo login
curl -X POST http://localhost:3000/api/demo/login

# Reset password (not implemented - need to implement)
# For now, delete and recreate user
node scripts/db-init.js delete-user user@email.com
node scripts/db-init.js create-user user@email.com newpass "User Name"
```

---

#### Problem: `Demo user not seeded yet. Call /api/demo/seed first`

**Solution**: Seed demo data
```bash
node scripts/db-init.js seed

# Or via HTTP
curl -X POST http://localhost:3000/api/demo/seed
```

---

### 4. Database Data Issues

#### Problem: `E11000 duplicate key error`

**Cause**: Unique index violation (usually on email)

**Example**:
```
MongoServerError: E11000 duplicate key error collection: finsight.users index: email_1 dup key: { email: "test@example.com" }
```

**Solutions**:
```bash
# Check for duplicate emails
mongosh
> use finsight
> db.users.find({ email: "test@example.com" })

# Delete duplicate
> db.users.deleteOne({ email: "test@example.com" })

# Or reset everything
node scripts/db-init.js reset
node scripts/db-init.js seed
```

---

#### Problem: Data not persisting between server restarts

**Cause**: Using Docker with no volume mapping

**Solution**: Update `docker-compose.yml`
```yaml
services:
  mongodb:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db  # Add this

volumes:
  mongodb_data:  # Add this
```

Then restart:
```bash
docker-compose down -v  # Remove old volume
docker-compose up -d
```

---

### 5. Performance Issues

#### Problem: Slow queries / API endpoint timeout

**Solution 1: Add missing indexes**
```javascript
// In MongoDB shell
db.transactions.createIndex({ userId: 1, occurredAt: -1 })
db.users.createIndex({ email: 1 })
```

**Solution 2: Use pagination for large result sets**
```typescript
// In API route
const pageSize = 50
const page = parseInt(req.query.page as string) || 1
const skip = (page - 1) * pageSize

const transactions = await Transaction.find({ userId })
  .sort({ occurredAt: -1 })
  .skip(skip)
  .limit(pageSize)
```

**Solution 3: Use lean() for read-only queries**
```typescript
// Instead of:
const user = await User.findById(userId)

// Use:
const user = await User.findById(userId).lean()
```

---

#### Problem: MongoDB using too much RAM

**Solution**:
```bash
# Limit MongoDB memory in docker-compose.yml
services:
  mongodb:
    image: mongo:latest
    mem_limit: 512m
    memswap_limit: 512m
```

---

### 6. Seed/Import Issues

#### Problem: `node scripts/db-init.js seed` fails

**Check MongoDB connection**:
```bash
# Test connection
mongosh --host localhost:27017 --eval "db.version()"
```

**Check Node dependencies**:
```bash
npm install mongoose bcryptjs dotenv
```

**Manually run seed**:
```bash
node -e "
const mongoose = require('mongoose');
require('dotenv').config({ path: '.env.local' });
mongoose.connect(process.env.MONGODB_URI).then(() => {
  console.log('Connected!');
  process.exit(0);
}).catch(err => {
  console.error('Failed:', err.message);
  process.exit(1);
});
"
```

---

#### Problem: Seed script creates duplicate data

**Solution**: Reset before seeding
```bash
node scripts/db-init.js reset
node scripts/db-init.js seed
```

Or use reset flag:
```bash
curl -X POST "http://localhost:3000/api/demo/seed?reset=1"
```

---

### 7. Data Export/Import

#### Export all data to JSON
```bash
mongodump --uri="mongodb://localhost:27017/finsight" \
  --out=./mongo-backup

# Convert to JSON
mongorestore --uri="mongodb://localhost:27017/finsight" \
  --dump=./mongo-backup --archive --archive=backup.archive
```

#### Export specific collection
```bash
mongoexport --uri="mongodb://localhost:27017/finsight" \
  --collection=users --out=users.json

mongoexport --uri="mongodb://localhost:27017/finsight" \
  --collection=transactions --out=transactions.json
```

#### Import from JSON
```bash
mongoimport --uri="mongodb://localhost:27017/finsight" \
  --collection=users --file=users.json --jsonArray
```

---

### 8. Development Tools

#### MongoDB Compass (GUI)
- **Download**: https://www.mongodb.com/products/tools/compass
- **Connect**: `mongodb://localhost:27017`
- **Benefits**: Visual query builder, data exploration, index management

#### VS Code Extension
- Install: `MongoDB for VS Code`
- Connect to localhost:27017
- Browse collections directly in editor

#### Command Line Tools
```bash
# Install MongoDB tools
brew install mongodb-community-tools  # macOS
choco install mongodb  # Windows

# Common commands
mongosh                               # Interactive shell
mongoexport --help                   # Export data
mongoimport --help                   # Import data
mongodump --help                     # Full backup
mongorestore --help                  # Full restore
```

---

### 9. Environment-Specific Issues

#### Development
```env
MONGODB_URI=mongodb://localhost:27017/finsight
NODE_ENV=development
```

#### Production (MongoDB Atlas)
```env
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/finsight
NODE_ENV=production
# Add IP whitelist in MongoDB Atlas console
```

#### Testing
```env
MONGODB_URI=mongodb://mongodb:27017/finsight_test
NODE_ENV=test
```

---

### 10. Advanced Debugging

#### Enable MongoDB debug logging
```bash
# In .env.local
DEBUG=mongoose:*

# Then run
npm run dev
```

#### Check indexes
```bash
mongosh
> use finsight
> db.users.getIndexes()
> db.transactions.getIndexes()
```

#### Validate data integrity
```javascript
// Remove documents with null userId
db.transactions.deleteMany({ userId: null })

// Find orphaned transactions (user deleted but transactions exist)
db.transactions.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user"
    }
  },
  { $match: { user: { $size: 0 } } },
  { $project: { _id: 1, userId: 1 } }
])
```

---

## 📞 Getting Help

### Check Logs
```bash
# Next.js dev server logs
npm run dev  # Logs to console

# MongoDB logs (if running locally)
tail -f /var/log/mongodb/mongod.log  # macOS/Linux
# Windows: Check MongoDB Server installer logs
```

### Verify Setup
```bash
# Test Node.js
node --version

# Test MongoDB
mongosh --version

# Test npm packages
npm ls mongoose bcryptjs

# Test connection
node scripts/db-init.js list-users
```

### Reset Everything
```bash
# Nuclear option - start fresh
node scripts/db-init.js reset
node scripts/db-init.js seed
npm run dev:fresh
```

---

**Remember**: Most issues are related to MongoDB not running or missing environment variables. Always check those first! 🔍

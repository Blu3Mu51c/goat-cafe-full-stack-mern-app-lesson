# 03. JWT Authentication Implementation

## Overview
This file covers implementing JWT (JSON Web Token) authentication for user signup, login, and protected route access.

## Table of Contents
1. [JWT Authentication Flow](#jwt-authentication-flow)
2. [User Model with Password Hashing](#user-model-with-password-hashing)
3. [Authentication Middleware](#authentication-middleware)
4. [User Controller](#user-controller)
5. [User Routes](#user-routes)
6. [Testing Authentication](#testing-authentication)

---

## JWT Authentication Flow

### How JWT Works
```
1. User Login/Signup → Backend validates credentials
2. Token Generation → Backend creates JWT with user data
3. Token Storage → Frontend stores token in localStorage
4. API Requests → Frontend sends token in Authorization header
5. Token Verification → Backend middleware validates token
6. User Access → Backend provides access to protected resources
```

### JWT Structure
```
Header.Payload.Signature
```
- **Header**: Algorithm and token type
- **Payload**: User data (user ID, permissions, expiration)
- **Signature**: Encrypted hash for verification

---

## User Model with Password Hashing

### Create models/user.js
```javascript
import mongoose from 'mongoose';
import bcrypt from 'bcrypt';
const Schema = mongoose.Schema;

const SALT_ROUNDS = 6;

const userSchema = new Schema({
  name: { type: String, required: true },
  email: {
    type: String,
    unique: true,
    trim: true,
    lowercase: true,
    required: true
  },
  password: {
    type: String,
    trim: true,
    minlength: 3,
    required: true
  }
}, {
  timestamps: true,
  toJSON: {
    transform: function(doc, ret) {
      delete ret.password;
      return ret;
    }
  }
});

userSchema.pre('save', async function(next) {
  // 'this' is the use document
  if (!this.isModified('password')) return next();
  // update the password with the computed hash
  this.password = await bcrypt.hash(this.password, SALT_ROUNDS);
  return next();
});

export default mongoose.model('User', userSchema);
```

### Model Features Explained

#### 1. Schema Definition
```javascript
const userSchema = new mongoose.Schema({
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true }
}, {
    timestamps: true
});
```
- **`required: true`**: Field must be provided
- **`unique: true`**: Email must be unique across all users
- **`timestamps: true`**: Automatically adds `createdAt` and `updatedAt`

#### 2. Instance Methods
```javascript
userSchema.methods.comparePassword = async function(password) {
    return bcrypt.compare(password, this.password);
};
```
**Usage**: `const isMatch = await user.comparePassword('password123');`

#### 3. Static Methods
```javascript
userSchema.statics.findByEmail = function(email) {
    return this.findOne({ email });
};
```
**Usage**: `const user = await User.findByEmail('user@example.com');`

#### 4. Pre-save Middleware
```javascript
userSchema.pre('save', async function(next) {
    if (!this.isModified('password')) return next();
    
    try {
        const hashedPassword = await bcrypt.hash(this.password, 6);
        this.password = hashedPassword;
        next();
    } catch (error) {
        next(error);
    }
});
```
**Purpose**: Automatically hash passwords before saving to database
**Security**: Uses bcrypt with salt rounds of 6

---

## Authentication Middleware

### Create config/checkToken.js
```javascript
import jwt from 'jsonwebtoken';

export default (req, res, next) => {
    let token = req.get('Authorization')
    if(token){
        token = token.split(' ')[1]
        console.log('token', token)
        jwt.verify(token, process.env.SECRET, (err, decoded) => {
            req.user = err ? null : decoded.user
            req.exp = err ? null : new Date(decoded.exp * 1000)
        })
        return next()
    } else {
        req.user = null 
        return next()
    }
}
```

### Create config/ensureLoggedIn.js
```javascript
export default (req, res, next ) => {
    if(req.user) return next()
    res.status('401').json({ msg: 'Unauthorized You Shall Not Pass'})
}
```

### Middleware Flow Explained

#### 1. checkToken Middleware
**Purpose**: Extract and verify JWT token from Authorization header

**Process**:
1. Extract token from `Authorization: Bearer <token>` header
2. Verify token using `process.env.SECRET`
3. Set `req.user` and `req.exp` on successful verification
4. Set both to `null` on failure or missing token
5. Always call `next()` to continue

**Why always call next()**: Even if token verification fails, we want the request to continue so `ensureLoggedIn` can handle the authentication check.

#### 2. ensureLoggedIn Middleware
**Purpose**: Ensure user is authenticated before accessing protected routes

**Process**:
1. Check if `req.user` exists (set by `checkToken`)
2. If user exists, allow access by calling `next()`
3. If no user, return 401 Unauthorized response

---

## User Controller

### Create controllers/api/users.js
```javascript
import User from '../../models/user.js';
import jwt from 'jsonwebtoken';
import bcrypt from 'bcrypt';

const checkToken = (req, res) => {
  console.log('req.user', req.user)
  res.json(req.exp)
}

const dataController = {
  async create (req, res, next) {
    try {
      const user = await User.create(req.body)
      console.log(req.body)
      // token will be a string
      const token = createJWT(user)
      // send back the token as a string
      // which we need to account for
      // in the client
      res.locals.data.user = user
      res.locals.data.token = token
      next()
    } catch (e) {
      console.log('you got a database problem')
      res.status(400).json(e)
    }
  },
  async login (req, res, next) {
    try {
      const user = await User.findOne({ email: req.body.email })
      if (!user) throw new Error()
      const match = await bcrypt.compare(req.body.password, user.password)
      if (!match) throw new Error()
      res.locals.data.user = user
      res.locals.data.token = createJWT(user)
      next()
    } catch {
      res.status(400).json('Bad Credentials')
    }
  }
}

const apiController = {
  auth (req, res) {
    res.json({
      token: res.locals.data.token,
      user: res.locals.data.user
    })
  }
}

function createJWT (user) {
  return jwt.sign(
    // data payload
    {  user },
    process.env.SECRET,
    { expiresIn: '24h' }
  )
}

export {
  checkToken,
  dataController,
  apiController
};
```

### Controller Pattern Explained

#### 1. Data Controllers
**Purpose**: Handle business logic and data operations

**`signup` Process**:
1. Create new user with `User.create(req.body)`
2. Generate JWT token with `createJWT(user)`
3. Store data in `res.locals.data` for next middleware
4. Call `next()` to continue to API controller

**`login` Process**:
1. Find user by email
2. Compare password using bcrypt
3. Generate JWT token if credentials match
4. Store data in `res.locals.data`
5. Call `next()` to continue

#### 2. API Controllers
**Purpose**: Format and send responses

**`auth` Process**:
1. Extract data from `res.locals.data`
2. Send JSON response with token and user data

#### 3. JWT Creation
```javascript
function createJWT(user) {
    return jwt.sign(
        { user },                    // Payload
        process.env.SECRET,          // Secret key
        { expiresIn: '24h' }        // Options
    );
}
```
**Payload**: Contains user data (will be decoded on verification)
**Secret**: Used to sign and verify token
**Expiration**: Token expires in 24 hours

---

## User Routes

### Create routes/api/users.js
```javascript
import express from 'express';
import { checkToken, dataController, apiController } from '../../controllers/api/users.js';
import ensureLoggedIn from '../../config/ensureLoggedIn.js';

const router = express.Router();

// POST /api/users
router.post('/', dataController.create, apiController.auth)
// POST /api/users/login
router.post('/login', dataController.login, apiController.auth)

// GET /api/users/check-token
router.get('/check-token', ensureLoggedIn, checkToken)

export default router;
```

### Route Flow Explained

#### 1. Signup Route
```
POST /api/users/signup
↓
dataController.create (create user, generate token)
↓
apiController.auth (send response)
```

#### 2. Login Route
```
POST /api/users/login
↓
dataController.login (verify credentials, generate token)
↓
apiController.auth (send response)
```

---

## Testing Authentication

### Start Your Server
```bash
npm run server
```

### Test User Signup
```bash
curl -X POST http://localhost:3000/api/users/signup \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test User",
    "email": "test@example.com",
    "password": "password123"
  }'
```

**Expected Response**:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "60f7b3b3b3b3b3b3b3b3b3b3",
    "name": "Test User",
    "email": "test@example.com",
    "createdAt": "2023-07-20T10:00:00.000Z",
    "updatedAt": "2023-07-20T10:00:00.000Z"
  }
}
```

### Test User Login
```bash
curl -X POST http://localhost:3000/api/users/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123"
  }'
```

**Expected Response**: Same format as signup

### Test Protected Route (Should Fail)
```bash
curl http://localhost:3000/api/items
```

**Expected Response**: `{"msg":"Unauthorized You Shall Not Pass"}`

### Test Protected Route with Token (Should Succeed)
```bash
curl -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  http://localhost:3000/api/items
```

**Expected Response**: Items data or empty array

---

## Common Issues and Solutions

### Issue: JWT Verification Fails
**Error**: `{"msg":"Unauthorized You Shall Not Pass"}`

**Solutions**:
1. Check `SECRET` environment variable is set
2. Verify `checkToken` middleware is included in protected routes
3. Ensure token format: `Authorization: Bearer <token>`

### Issue: Password Hashing Not Working
**Error**: `Password mismatch` on login

**Solutions**:
1. Check bcrypt is properly imported
2. Verify pre-save middleware is working
3. Ensure password field is being modified

### Issue: User Creation Fails
**Error**: `ValidationError` or duplicate email

**Solutions**:
1. Check required fields are provided
2. Ensure email is unique
3. Verify password meets requirements

---

## Security Considerations

### Password Security
- **Never store plain text passwords**
- **Use bcrypt with salt rounds ≥ 10**
- **Validate password strength** (minimum length, complexity)

### JWT Security
- **Use strong, unique secrets**
- **Set appropriate expiration times**
- **Store secrets in environment variables**
- **Rotate secrets regularly**

### Input Validation
- **Validate email format**
- **Check password requirements**
- **Sanitize user inputs**
- **Handle validation errors gracefully**

---

## Next Steps

After completing this setup:

1. **Test Authentication**: Verify signup and login work
2. **Test Protected Routes**: Ensure middleware is working
3. **Check Security**: Verify passwords are hashed
4. **Move to Next File**: Continue with [04_DATABASE_MODELS.md](./04_DATABASE_MODELS.md)

## Verification Checklist

- [ ] User model created with password hashing
- [ ] Authentication middleware working
- [ ] User controller handles signup and login
- [ ] User routes configured correctly
- [ ] JWT tokens generated successfully
- [ ] Protected routes require authentication
- [ ] Passwords hashed before saving
- [ ] No authentication bypass possible
- [ ] Error handling implemented
- [ ] Security best practices followed

Once all items are checked, you're ready to proceed to the next file!

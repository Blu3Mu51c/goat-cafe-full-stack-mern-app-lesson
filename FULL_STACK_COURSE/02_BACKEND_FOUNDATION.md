# 02. Backend Foundation and Server Setup

## Overview
This file covers setting up the Express backend server, database connection, and basic middleware configuration.

## Table of Contents
1. [Server Entry Point](#server-entry-point)
2. [Express App Configuration](#express-app-configuration)
3. [Database Connection](#database-connection)
4. [Middleware Setup](#middleware-setup)
5. [Static File Serving](#static-file-serving)
6. [React Router Support](#react-router-support)

---

## Server Entry Point

### Create server.js
Create the main server entry point:
```javascript
import dotenv from 'dotenv';

// Load environment variables FIRST
dotenv.config();

// Then import other modules that need environment variables
import './config/database.js';
import app from './app-server.js';

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
	console.log('We in the building on ' + PORT)
})
```

### Critical Import Order
**Why this order matters:**
1. **`dotenv.config()`** must run first to load environment variables
2. **Database import** needs `MONGO_URI` to be available
3. **App import** comes last after all dependencies are loaded

**Common Mistake**: Importing database before `dotenv.config()` will result in `MONGO_URI undefined` errors.

---

## Express App Configuration

### Create app-server.js
Create the Express application configuration:
```javascript
import express from 'express';
import path from 'path';
import { fileURLToPath } from 'url';
import cors from 'cors';
import checkToken from './config/checkToken.js';
import ensureLoggedIn from './config/ensureLoggedIn.js';
import userRoutes from './routes/api/users.js';
import itemRoutes from './routes/api/items.js';
import orderRoutes from './routes/api/orders.js';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

const app = express();

app.use(cors());
app.use(express.json());
app.use((req, res, next) => {
    res.locals.data = {}
    next()
})

// API Routes - these must come before the static file serving
app.use('/api/users', userRoutes);
app.use('/api/items', checkToken, ensureLoggedIn, itemRoutes);
app.use('/api/orders', checkToken, ensureLoggedIn, orderRoutes);

// Determine which directory to serve static files from
const staticDir = process.env.NODE_ENV === 'production' ? 'dist' : 'public';
const indexPath = process.env.NODE_ENV === 'production' ? 'dist/index.html' : 'index.html';

// Serve static files from the appropriate directory
app.use(express.static(staticDir));

// For React Router - serve index.html for all non-API routes
app.get('*', (req, res) => {
    // Don't serve index.html for API routes
    if (req.path.startsWith('/api/')) {
        return res.status(404).json({ error: 'API endpoint not found' });
    }
    
    // Serve the React app for all other routes
    res.sendFile(path.resolve(path.join(__dirname, indexPath)));
});

export default app;
```

### Key Configuration Points

#### 1. ES Module Path Resolution
```javascript
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
```
**Why needed**: ES modules don't have `__dirname` by default, so we create it manually.

#### 2. Middleware Order
```javascript
app.use(cors());           // 1. Enable CORS
app.use(express.json());   // 2. Parse JSON bodies
app.use((req, res, next) => {  // 3. Initialize response locals
    res.locals.data = {}
    next()
})
```

#### 3. API Routes with Authentication
```javascript
app.use('/api/users', userRoutes);                    // Public routes
app.use('/api/items', checkToken, ensureLoggedIn, itemRoutes);    // Protected
app.use('/api/orders', checkToken, ensureLoggedIn, orderRoutes);  // Protected
```

**Middleware Chain**: `checkToken` → `ensureLoggedIn` → route handlers

---

## Database Connection

### Create config/database.js
Create the MongoDB connection configuration:
```javascript
import mongoose from 'mongoose';
import dotenv from 'dotenv';

dotenv.config();    

mongoose.connect(process.env.MONGO_URI)

mongoose.connection.once('open', () => {
    console.log('Mongo is showing us love')
})
```

### Connection Events Explained
- **`connected`**: Fired when successfully connected to MongoDB
- **`error`**: Fired when connection fails
- **`disconnected`**: Fired when connection is lost

### Advanced Connection Options
```javascript
mongoose.connect(process.env.MONGO_URI, {
    serverSelectionTimeoutMS: 5000,  // Timeout for server selection
    socketTimeoutMS: 45000,          // Socket timeout
    bufferCommands: false,           // Disable mongoose buffering
    bufferMaxEntries: 0              // Disable model buffering
});
```

---

## Middleware Setup

### CORS Configuration
```javascript
app.use(cors());
```
**Purpose**: Allows frontend (port 5173) to make requests to backend (port 3000).

**Custom CORS Configuration** (if needed):
```javascript
app.use(cors({
    origin: ['http://localhost:5173', 'https://yourdomain.com'],
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization']
}));
```

### JSON Body Parser
```javascript
app.use(express.json());
```
**Purpose**: Parses incoming JSON request bodies into `req.body`.

**Alternative Options**:
```javascript
app.use(express.json({ limit: '10mb' }));        // Limit body size
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded bodies
```

### Response Locals Initialization
```javascript
app.use((req, res, next) => {
    res.locals.data = {}
    next()
})
```
**Purpose**: Initializes `res.locals.data` for sharing data between middleware functions.

---

## Static File Serving

### Development vs Production
```javascript
const staticDir = process.env.NODE_ENV === 'production' ? 'dist' : 'public';
const indexPath = process.env.NODE_ENV === 'production' ? 'dist/index.html' : 'index.html';
```

**Development**: Serves from `public/` directory (Vite dev server)
**Production**: Serves from `dist/` directory (built files)

### Static File Middleware
```javascript
app.use(express.static(staticDir));
```
**Purpose**: Serves static files (CSS, JS, images) directly from the specified directory.

---

## React Router Support

### Catch-All Route
```javascript
app.get('*', (req, res) => {
    // Don't serve index.html for API routes
    if (req.path.startsWith('/api/')) {
        return res.status(404).json({ error: 'API endpoint not found' });
    }
    
    // Serve the React app for all other routes
    res.sendFile(path.resolve(path.join(__dirname, indexPath)));
});
```

**Purpose**: Enables client-side routing by serving `index.html` for all non-API routes.

**How it works**:
1. **API routes** (`/api/*`) return 404 if not found
2. **Frontend routes** (`/orders`, `/cart`, etc.) serve `index.html`
3. **React Router** takes over and renders the appropriate component

---

## Testing Your Backend

### Start the Server
```bash
npm run server
```

### Expected Output
```
Connected to MongoDB at cluster.mongodb.net:27017
We in the building on 3000
```

### Test Basic Functionality
```bash
# Test if server is running
curl http://localhost:3000

# Test CORS (should work from frontend)
curl -H "Origin: http://localhost:5173" http://localhost:3000/api/users
```

---

## Common Issues and Solutions

### Issue: MongoDB Connection Failed
**Error**: `MongooseError: The uri parameter to openUri() must be a string, got "undefined"`

**Solution**: Check import order in `server.js` - `dotenv.config()` must come before database import.

### Issue: Port Already in Use
**Error**: `EADDRINUSE: address already in use :::3000`

**Solution**: 
```bash
lsof -i :3000
kill -9 <PID>
```

### Issue: CORS Errors
**Error**: `Access to fetch at 'http://localhost:3000/api/users' from origin 'http://localhost:5173' has been blocked by CORS policy`

**Solution**: Ensure `app.use(cors())` is configured before routes.

---

## Next Steps

After completing this setup:

1. **Verify Server**: Ensure backend starts without errors
2. **Test Database**: Confirm MongoDB connection is successful
3. **Check Middleware**: Verify CORS and JSON parsing work
4. **Move to Next File**: Continue with [03_JWT_AUTHENTICATION.md](./03_JWT_AUTHENTICATION.md)

## Verification Checklist

- [ ] `server.js` created with correct import order
- [ ] `app-server.js` configured with all middleware
- [ ] Database connection established successfully
- [ ] Server starts on correct port
- [ ] CORS middleware enabled
- [ ] Static file serving configured
- [ ] React Router catch-all route working
- [ ] No MongoDB connection errors
- [ ] No import/export errors

Once all items are checked, you're ready to proceed to the next file!

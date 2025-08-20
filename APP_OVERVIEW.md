# Goat Cafe Vite - Full Stack App Build Guide

<center>

# The Flow of the code files

![mern architecture](https://i.imgur.com/uoJvBRK.jpg)

# User does action (click button, fill out form, or other trigger [onClick, onSubmit etc], or useEffect action (component mounts, data changes...))
# Event Handler / Callback Function is called in REACT COMPONENT
## <--> users-service.js <--> users-api.js <---> (We don't do non-react logic in react components we do them in services/utilities, not because we can't but we do it to be organized, and make it easier to let react specialists focus on react and others worry about non-react logic)
# <-FETCH Request Over Internet->
## server.js <--> Routes <--> Controllers
# <-Send Response Over Internet->
## users-api.js <--> users-service.js <--->
# Back to REACT COMPONENT

</center>

<hr/>

## A Full Stack Developer knows how to connect the frontend and backend together and understands the flow of data from FRONT to BACK and Back to Front

## Team Scenario
1. Mike is a developer a backend specialist, has almost no frontend knowledge
1. Jenny is a developer and a REACT & CSS Master
1. LaQuiesha is a developer bad at styling, but knows the entire MERN Stack is a true Full Stack Developer

### Mike would...
1. Work on setting up server and all routes, controllers, and Models

### Jenny would ...
1. Build all Components and Pages

### LaQuiesha would ...
1. Build utilities that Jenny can call in Components and Pages

#### Issue Jenny receives a change from UX team and now needs more data to render page,
1. Jenny thinks this could effect the backend
1. Mike is confused by the request, because the data is on the backend recommends the frontend make additional api requests
1. LaQueshia who is a Full Stack Developer understands how to fix the issue and makes small updates in backend by adding extra controllers `think of Josh's table method` and makes changes in utility functions so that now Jenny gets the updated data needed.

<center>

# JWT

![](https://i.imgur.com/IXByEPP.png)

# Browser & Server

![](https://i.imgur.com/3quZxs4.png)

</center>

---

## Table of Contents

1. [Overview of how the Full Stack App works](#overview)
2. [Setup](#setup)
3. [Creating the Models](#models)
4. [Config files](#config)
5. [Controllers](#controllers)
6. [Routes](#routes)
7. [Server files](#server)
8. [Testing Routes in Postman](#testing)
9. [Vite Config changes](#vite)
10. [Frontend utilities](#utilities)
11. [App Router and Pages (Stubs only at first)](#router)
12. [Components and integrating them into the pages](#components)
13. [Explanation of SCSS and SCSS Modules](#scss-explanation)
14. [SCSS integration](#scss-integration)
15. [Testing the full app in the browser](#testing-browser)
16. [Refactoring to a Dynamic App Router](#refactor-router)

---

## 1. Overview of how the Full Stack App works

This is a full-stack MERN (MongoDB, Express, React, Node.js) application that demonstrates user authentication, order management, and a restaurant menu system. The app follows a clear separation of concerns where:

- **Backend**: Handles data persistence, business logic, and API endpoints
- **Frontend**: Provides user interface and manages application state
- **Database**: Stores users, items, categories, and orders
- **Authentication**: Uses JWT tokens for secure user sessions

The data flow follows this pattern:
1. User interacts with React component
2. Component calls utility/service function
3. Service makes API call to backend
4. Backend processes request through routes → controllers → models
5. Response flows back through the same path
6. Component updates with new data

---

## 2. Setup

### 2a. Using npm create vite

First, create a new Vite project:

```bash
npm create vite@latest goat-cafe-vite -- --template react
cd goat-cafe-vite
npm install
```

### 2b. Making the config, models, controllers and routes folders

Create the following folder structure:

```bash
mkdir config
mkdir models
mkdir controllers
mkdir controllers/api
mkdir routes
mkdir routes/api
```

### 2c. Making app-server and server.js

Create `app-server.js` in the root directory:

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

Create `server.js` in the root directory:

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

### 2d. Installing packages, and updating package.json

Install the required packages:

```bash
npm install express mongoose dotenv bcrypt jsonwebtoken cors react-router-dom
npm install --save-dev nodemon concurrently sass
```

Update your `package.json` to include these scripts:

```json
{
  "name": "goat-cafe-vite",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "server": "nodemon server.js",
    "dev:full": "concurrently \"npm run server\" \"npm run dev\"",
    "build": "vite build",
    "start": "NODE_ENV=production node server.js",
    "start:dev": "NODE_ENV=development node server.js",
    "lint": "eslint .",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^19.1.1",
    "react-dom": "^19.1.1",
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1",
    "mongoose": "^8.0.3",
    "bcrypt": "^5.1.1",
    "jsonwebtoken": "^9.0.2",
    "react-router-dom": "^6.8.0"
  },
  "devDependencies": {
    "@eslint/js": "^9.33.0",
    "@types/react": "^19.1.10",
    "@types/react-dom": "^19.1.7",
    "@vitejs/plugin-react": "^5.0.0",
    "eslint": "^9.33.0",
    "eslint-plugin-react-hooks": "^5.2.0",
    "eslint-plugin-react-refresh": "^0.4.20",
    "globals": "^16.3.0",
    "vite": "^7.1.2",
    "sass": "^1.69.5",
    "concurrently": "^8.2.2"
  }
}
```

Create a `.env` file in the root directory:

```env
MONGO_URI=mongodb://127.0.0.1:27017/goat-cafe
SECRET=your-secret-key-here
PORT=3000
```

---

## 3. Creating the Models

### 3a. User Model

Create `models/user.js`:

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

**What this does**: Defines a user schema with name, email, and password fields. Uses bcrypt to hash passwords before saving and removes password from JSON responses for security.

### 3b. Category Model

Create `models/category.js`:

```javascript
import mongoose from 'mongoose';
const Schema = mongoose.Schema;

const categorySchema = new Schema({
  name: { type: String, required: true },
  sortOrder: Number
}, {
  timestamps: true
});

export default mongoose.model('Category', categorySchema);
```

**What this does**: Simple schema for food categories with a name and sort order for organizing menu items.

### 3c. Item Schema

Create `models/itemSchema.js`:

```javascript
import item from './item.js';

const Schema = (await import('mongoose')).Schema;

const itemSchema = new Schema({
  name: { type: String, required: true },
  emoji: String,
  category: { type: Schema.Types.ObjectId, ref: 'Category' },
  price: { type: Number, required: true, default: 0 }
}, {
  timestamps: true
});

export default itemSchema;
```

**What this does**: Defines the structure for individual menu items with name, emoji, category reference, and price.

### 3d. Item Model

Create `models/item.js`:

```javascript
import mongoose from 'mongoose';
// Ensure the Category model is processed by Mongoose
import './category.js';

import itemSchema from './itemSchema.js';

export default mongoose.model('Item', itemSchema);
```

**What this does**: Creates the Item model from the schema and ensures the Category model is loaded for population.

### 3e. Order Model

Create `models/order.js`:

```javascript
import mongoose from 'mongoose';
const Schema = mongoose.Schema;
import itemSchema from './itemSchema.js';

const lineItemSchema = new Schema({
  qty: { type: Number, default: 1 },
  item: itemSchema
}, {
  timestamps: true,
  toJSON: { virtuals: true }
});

lineItemSchema.virtual('extPrice').get(function() {
  // 'this' is bound to the lineItem subdoc
  return this.qty * this.item.price;
});

const orderSchema = new Schema({
  user: { type: Schema.Types.ObjectId, ref: 'User' },
  lineItems: [lineItemSchema],
  isPaid: { type: Boolean, default: false }
}, {
  timestamps: true,
  toJSON: { virtuals: true }
});

orderSchema.virtual('orderTotal').get(function() {
  return this.lineItems.reduce((total, item) => total + item.extPrice, 0);
});

orderSchema.virtual('totalQty').get(function() {
  return this.lineItems.reduce((total, item) => total + item.qty, 0);
});

orderSchema.virtual('orderId').get(function() {
  return this.id.slice(-6).toUpperCase();
});

orderSchema.statics.getCart = function(userId) {
  // 'this' is the Order model
  return this.findOneAndUpdate(
    // query
    { user: userId, isPaid: false },
    // update
    { user: userId },
    // upsert option will create the doc if
    // it doesn't exist
    { upsert: true, new: true }
  );
};

orderSchema.methods.addItemToCart = async function(itemId) {
  const cart = this;
  // Check if item already in cart
  const lineItem = cart.lineItems.find(lineItem => lineItem.item._id.equals(itemId));
  if (lineItem) {
    lineItem.qty += 1;
  } else {
    const item = await mongoose.model('Item').findById(itemId);
    cart.lineItems.push({ item });
  }
  return cart.save();
};

// Instance method to set an item's qty in the cart (will add item if does not exist)
orderSchema.methods.setItemQty = function(itemId, newQty) {
  // this keyword is bound to the cart (order doc)
  const cart = this;
  // Find the line item in the cart for the menu item
  const lineItem = cart.lineItems.find(lineItem => lineItem.item._id.equals(itemId));
  if (lineItem && newQty <= 0) {
    // Calling remove, removes itself from the cart.lineItems array
    lineItem.deleteOne();
  } else if (lineItem) {
    // Set the new qty - positive value is assured thanks to prev if
    lineItem.qty = newQty;
  }
  // return the save() method's promise
  return cart.save();
};

export default mongoose.model('Order', orderSchema);
```

**What this does**: Complex order schema with line items, virtuals for calculated fields, static methods for cart operations, and instance methods for cart manipulation.

### Understanding Virtuals, Statics, and Methods

**Virtuals**: Document properties that don't persist in MongoDB but are computed on-the-fly. They're like getter methods that calculate values from existing data.

**Statics**: Methods defined on the Model itself (not instances). They're like class methods that can be called without creating an instance.

**Methods**: Instance methods that operate on individual documents. They're like object methods that can manipulate the current document.

**Example Usage**:
- `Order.getCart(userId)` - Static method called on the Order model
- `cart.addItemToCart(itemId)` - Instance method called on a specific order document
- `order.orderTotal` - Virtual property that calculates total from line items

---

## 4. Config files

### 4a. Database Configuration

Create `config/database.js`:

```javascript
import mongoose from 'mongoose';
import dotenv from 'dotenv';

dotenv.config();    

mongoose.connect(process.env.MONGO_URI)

mongoose.connection.once('open', () => {
    console.log('Mongo is showing us love')
})
```

**What this does**: Establishes connection to MongoDB using the MONGO_URI from environment variables and logs connection status.

### 4b. JWT Token Checking

Create `config/checkToken.js`:

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

**What this does**: Middleware that checks for JWT tokens in headers, verifies them, and attaches the decoded user to req.user for use in subsequent middleware.

### 4c. Authentication Middleware

Create `config/ensureLoggedIn.js`:

```javascript
export default (req, res, next ) => {
    if(req.user) return next()
    res.status('401').json({ msg: 'Unauthorized You Shall Not Pass'})
}
```

**What this does**: Simple middleware that checks if req.user exists (set by checkToken) and returns unauthorized error if no user is logged in.

### 4d. Seed File

Create `config/seed.js`:

```javascript
import dotenv from 'dotenv';
import './database.js';
import Category from '../models/category.js';
import Item from '../models/item.js';

dotenv.config();

(async function() {

  await Category.deleteMany({});
  const categories = await Category.create([
    {name: 'Sandwiches', sortOrder: 10},
    {name: 'Seafood', sortOrder: 20},
    {name: 'Mexican', sortOrder: 30},
    {name: 'Italian', sortOrder: 40},
    {name: 'Sides', sortOrder: 50},
    {name: 'Desserts', sortOrder: 60},
    {name: 'Drinks', sortOrder: 70},
  ]);

  await Item.deleteMany({});
  const items = await Item.create([
    {name: 'Hamburger', emoji: '🍔', category: categories[0], price: 5.95},
    {name: 'Turkey Sandwich', emoji: '🥪', category: categories[0], price: 6.95},
    {name: 'Hot Dog', emoji: '🌭', category: categories[0], price: 3.95},
    {name: 'Crab Plate', emoji: '🦀', category: categories[1], price: 14.95},
    {name: 'Fried Shrimp', emoji: '🍤', category: categories[1], price: 13.95},
    {name: 'Whole Lobster', emoji: '🦞', category: categories[1], price: 25.95},
    {name: 'Taco', emoji: '🌮', category: categories[2], price: 1.95},
    {name: 'Burrito', emoji: '🌯', category: categories[2], price: 4.95},
    {name: 'Pizza Slice', emoji: '🍕', category: categories[3], price: 3.95},
    {name: 'Spaghetti', emoji: '🍝', category: categories[3], price: 7.95},
    {name: 'Garlic Bread', emoji: '🍞', category: categories[3], price: 1.95},
    {name: 'French Fries', emoji: '🍟', category: categories[4], price: 2.95},
    {name: 'Green Salad', emoji: '🥗', category: categories[4], price: 3.95},
    {name: 'Ice Cream', emoji: '🍨', category: categories[5], price: 1.95},
    {name: 'Cup Cake', emoji: '🧁', category: categories[5], price: 0.95},
    {name: 'Custard', emoji: '🍮', category: categories[5], price: 2.95},
    {name: 'Strawberry Shortcake', emoji: '🍰', category: categories[5], price: 3.95},
    {name: 'Milk', emoji: '🥛', category: categories[6], price: 0.95},
    {name: 'Coffee', emoji: '☕', category: categories[6], price: 0.95},
    {name: 'Mai Tai', emoji: '🍹', category: categories[6], price: 8.95},
    {name: 'Beer', emoji: '🍺', category: categories[6], price: 3.95},
    {name: 'Wine', emoji: '🍷', category: categories[6], price: 7.95},
  ]);

  console.log(items)

  process.exit();

})();
```

**What this does**: Populates the database with sample categories and menu items for testing. Run with `node config/seed.js` to populate your database.

---

## 5. Controllers

### 5a. Users Controller

Create `controllers/api/users.js`:

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

export {
  checkToken,
  dataController,
  apiController
}

/* -- Helper Functions -- */

function createJWT (user) {
  return jwt.sign(
    // data payload
    {  user },
    process.env.SECRET,
    { expiresIn: '24h' }
  )
}
```

**What this does**: Handles user registration and login. Creates JWT tokens for authentication and uses bcrypt to verify passwords securely.

### 5b. Orders Controller

Create `controllers/api/orders.js`:

```javascript
import Order from '../../models/order.js';

export {
  cart,
  addToCart,
  setItemQtyInCart,
  checkout,
  history
};

// A cart is the unpaid order for a user
async function cart(req, res) {
  try{
    const cart = await Order.getCart(req.user._id);
    res.status(200).json(cart);
  }catch(e){
    res.status(400).json({ msg: e.message });
  }
}

// Add an item to the cart
async function addToCart(req, res) {
  try{
    const cart = await Order.getCart(req.user._id);
    await cart.addItemToCart(req.params.id);
    res.status(200).json(cart);
  }catch(e){
    res.status(400).json({ msg: e.message });
  }  
}

// Updates an item's qty in the cart
async function setItemQtyInCart(req, res) {
  try{
    const cart = await Order.getCart(req.user._id);
    await cart.setItemQty(req.body.itemId, req.body.newQty);
    res.status(200).json(cart);
  }catch(e){
    res.status(400).json({ msg: e.message });
  }
}

// Update the cart's isPaid property to true
async function checkout(req, res) {
  try{
    const cart = await Order.getCart(req.user._id);
    cart.isPaid = true;
    await cart.save();
    res.status(200).json(cart);
  }catch(e){
    res.status(400).json({ msg: e.message });
  }  
}

// Return the logged in user's paid order history
async function history(req, res) {
  // Sort most recent orders first
  try{
    const orders = await Order
      .find({ user: req.user._id, isPaid: true })
      .sort('-updatedAt').exec();
    res.status(200).json(orders);
  }catch(e){
    res.status(400).json({ msg: e.message });
  }
}
```

**What this does**: Manages all order-related operations including cart management, adding items, updating quantities, checkout, and retrieving order history.

### 5c. Items Controller

Create `controllers/api/items.js`:

```javascript
import Item from '../../models/item.js';

export default {
  index,
  show
};
export {
  index,
  show
}

// GET /api/items
async function index(req, res) {
  try{
    const items = await Item.find({}).sort('name').populate('category').exec();
    // re-sort based upon the sortOrder of the categories
    items.sort((a, b) => a.category.sortOrder - b.category.sortOrder);
    res.status(200).json(items);
  }catch(e){
    res.status(400).json({ msg: e.message });
  }
}

async function show(req, res) {
  try{
    const item = await Item.findById(req.params.id);
    res.status(200).json(item);
  }catch(e){
    res.status(400).json({ msg: e.message });
  }  
}
```

**What this does**: Provides endpoints to retrieve all menu items (sorted by category) and individual item details with category population.

---

## 6. Routes

### 6a. Users Routes

Create `routes/api/users.js`:

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

**What this does**: Defines user authentication routes for signup, login, and token verification with proper middleware protection.

### 6b. Orders Routes

Create `routes/api/orders.js`:

```javascript
import express from 'express';
import { cart, addToCart, setItemQtyInCart, checkout, history } from '../../controllers/api/orders.js';

const router = express.Router();

// GET /api/orders/cart
router.get('/cart', cart);
// GET /api/orders/history
router.get('/history', history);
// POST /api/orders/cart/items/:id
router.post('/cart/items/:id', addToCart);
// POST /api/orders/cart/checkout
router.post('/cart/checkout', checkout);
// POST /api/orders/cart/qty
router.put('/cart/qty', setItemQtyInCart);

export default router;
```

**What this does**: Defines all order management routes including cart operations, checkout, and order history retrieval.

### 6c. Items Routes

Create `routes/api/items.js`:

```javascript
import express from 'express';
import itemsCtrl from '../../controllers/api/items.js';

const router = express.Router();

// GET /api/items
router.get('/', itemsCtrl.index);
// GET /api/items/:id
router.get('/:id', itemsCtrl.show);

export default router;
```

**What this does**: Provides routes for retrieving menu items, either all items or individual items by ID.

---

## 7. Server files

The server files (`app-server.js` and `server.js`) are already created in section 2c. These files:

1. **Load environment variables** and database connection
2. **Set up Express middleware** for logging, JSON parsing, and static file serving
3. **Configure authentication** with JWT token checking
4. **Mount API routes** for users, items, and orders
5. **Serve the React app** for client-side routing
6. **Start the server** on the specified port

**Key Features**:
- Uses `checkToken` middleware to verify JWT tokens
- Protects item and order routes with `ensureLoggedIn` middleware
- Serves static files from the `dist` folder (built React app)
- Includes a catch-all route for SPA client-side routing

---

## 8. Testing Routes in Postman

### 8a. Setup Postman

1. **Create a new collection** called "SEI CAFE API"
2. **Set base URL** to `http://localhost:3000`
3. **Create environment variables**:
   - `baseUrl`: `http://localhost:3000`
   - `token`: (leave empty, will be set after login)

### 8b. Test User Routes

#### POST /api/users (Sign Up)
- **Method**: POST
- **URL**: `{{baseUrl}}/api/users`
- **Body** (raw JSON):
```json
{
  "name": "Test User",
  "email": "test@example.com",
  "password": "password123"
}
```
- **Expected Response**: JWT token string
- **Save token**: Copy the response and set it as the `token` environment variable

#### POST /api/users/login (Login)
- **Method**: POST
- **URL**: `{{baseUrl}}/api/users/login`
- **Body** (raw JSON):
```json
{
  "email": "test@example.com",
  "password": "password123"
}
```
- **Expected Response**: JWT token string
- **Update token**: Set this as the new `token` environment variable

#### GET /api/users/check-token (Verify Token)
- **Method**: GET
- **URL**: `{{baseUrl}}/api/users/check-token`
- **Headers**: `Authorization: Bearer {{token}}`
- **Expected Response**: Token expiration info

### 8c. Test Items Routes

#### GET /api/items (Get All Items)
- **Method**: GET
- **URL**: `{{baseUrl}}/api/items`
- **Headers**: `Authorization: Bearer {{token}}`
- **Expected Response**: Array of menu items with categories

#### GET /api/items/:id (Get Single Item)
- **Method**: GET
- **URL**: `{{baseUrl}}/api/items/[item-id]` (replace with actual item ID)
- **Headers**: `Authorization: Bearer {{token}}`
- **Expected Response**: Single item object

### 8d. Test Orders Routes

#### GET /api/orders/cart (Get Cart)
- **Method**: GET
- **URL**: `{{baseUrl}}/api/orders/cart`
- **Headers**: `Authorization: Bearer {{token}}`
- **Expected Response**: Current user's cart (unpaid order)

#### POST /api/orders/cart/items/:id (Add to Cart)
- **Method**: POST
- **URL**: `{{baseUrl}}/api/orders/cart/items/[item-id]`
- **Headers**: `Authorization: Bearer {{token}}`
- **Expected Response**: Updated cart with new item

#### PUT /api/orders/cart/qty (Update Quantity)
- **Method**: PUT
- **URL**: `{{baseUrl}}/api/orders/cart/qty`
- **Headers**: `Authorization: Bearer {{token}}`
- **Body** (raw JSON):
```json
{
  "itemId": "[item-id]",
  "newQty": 2
}
```
- **Expected Response**: Updated cart with modified quantity

#### POST /api/orders/cart/checkout (Checkout)
- **Method**: POST
- **URL**: `{{baseUrl}}/api/orders/cart/checkout`
- **Headers**: `Authorization: Bearer {{token}}`
- **Expected Response**: Completed order with `isPaid: true`

#### GET /api/orders/history (Order History)
- **Method**: GET
- **URL**: `{{baseUrl}}/api/orders/history`
- **Headers**: `Authorization: Bearer {{token}}`
- **Expected Response**: Array of paid orders for the user

### 8e. Testing Flow

1. **Start the server**: `npm run server`
2. **Seed the database**: `node config/seed.js`
3. **Test signup** to create a user
4. **Test login** to get a token
5. **Test protected routes** using the token
6. **Test cart operations** by adding items and checking out
7. **Verify order history** shows completed orders

**Common Issues**:
- **401 Unauthorized**: Check that the token is valid and included in Authorization header
- **400 Bad Request**: Verify request body format and required fields
- **500 Server Error**: Check server logs for database connection or model errors

---

## 9. Vite Config changes

Update your `vite.config.js` to include a proxy for development:

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  css: {
    preprocessorOptions: {
      scss: {
        // Remove the problematic import - we'll handle imports in individual files
        // additionalData: `@import "src/index.scss";`
      }
    }
  },
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true
      }
    }
  }
})
```

**What this does**: Sets up a development proxy so that API calls to `/api/*` from the frontend are forwarded to the backend server running on port 3000.

---

## 10. Frontend utilities

### 10a. Send Request Utility

Create `src/utilities/send-request.js`:

```javascript
import { getToken } from './users-service';

export default async function sendRequest(url, method = 'GET', payload = null) {
  // Fetch takes an optional options object as the 2nd argument
  // used to include a data payload, set headers, etc.
  const options = { method };
  if (payload) {
    options.headers = { 'Content-Type': 'application/json' };
    options.body = JSON.stringify(payload);
  }
  const token = getToken();
  if (token) {
    // Ensure headers object exists
    options.headers = options.headers || {};
    // Add token to an Authorization header
    // Prefacing with 'Bearer' is recommended in the HTTP specification
    options.headers.Authorization = `Bearer ${token}`;
  }
  const res = await fetch(url, options);
  // res.ok will be false if the status code set to 4xx in the controller action
  if (res.ok) return res.json();
  throw new Error('Bad Request');
}
```

**What this does**: Centralized utility for making HTTP requests with automatic JWT token inclusion and error handling.

### 10b. Users API

Create `src/utilities/users-api.js`:

```javascript
import sendRequest from './send-request';

const BASE_URL = '/api/users';

export function signUp(userData) {
  return sendRequest(BASE_URL, 'POST', userData);
}

export function login(credentials) {
  return sendRequest(`${BASE_URL}/login`, 'POST', credentials);
}
```

**What this does**: API functions for user authentication that use the send-request utility.

### 10c. Users Service

Create `src/utilities/users-service.js`:

```javascript
import * as usersAPI  from './users-api';

export async function signUp(userData) {
  // The backend now returns { token: "...", user: {...} }
  const response = await usersAPI.signUp(userData);
  // Persist the token to localStorage
  localStorage.setItem('token', response.token);
  // Return the user object directly
  return response.user;
}

export async function login(credentials) {
  // The backend now returns { token: "...", user: {...} }
  const response = await usersAPI.login(credentials);
  // Persist the token to localStorage
  localStorage.setItem('token', response.token);
  // Return the user object directly
  return response.user;
}

export function getToken() {
  const token = localStorage.getItem('token');
  // getItem will return null if no key
  if (!token) return null;
  
  try {
    const payload = JSON.parse(atob(token.split('.')[1]));
    // A JWT's expiration is expressed in seconds, not miliseconds
    if (payload.exp < Date.now() / 1000) {
      // Token has expired
      localStorage.removeItem('token');
      return null;
    }
    return token;
  } catch (error) {
    // Token is malformed or invalid
    console.log('Invalid token, removing from localStorage');
    localStorage.removeItem('token');
    return null;
  }
}

export function getUser() {
  const token = getToken();
  if (!token) return null;
  
  try {
    return JSON.parse(atob(token.split('.')[1])).user;
  } catch (error) {
    // Token is malformed, remove it
    console.log('Invalid token format, removing from localStorage');
    localStorage.removeItem('token');
    return null;
  }
}

export function logOut() {
  localStorage.removeItem('token');
}
```

**What this does**: Service layer that handles JWT token storage, retrieval, and user state management with automatic token expiration checking.

### 10d. Items API

Create `src/utilities/items-api.js`:

```javascript
import sendRequest from './send-request';

const BASE_URL = '/api/items';

export function getAll() {
  return sendRequest(BASE_URL);
}

export function getById(id) {
  return sendRequest(`${BASE_URL}/${id}`);
}
```

**What this does**: API functions for retrieving menu items with automatic authentication.

### 10e. Orders API

Create `src/utilities/orders-api.js`:

```javascript
import sendRequest from './send-request';

const BASE_URL = '/api/orders';

// Retrieve an unpaid order for the logged in user
export function getCart() {
  return sendRequest(`${BASE_URL}/cart`);
}

// Add an item to the cart
export function addItemToCart(itemId) {
  // Just send itemId for best security (no pricing)
  return sendRequest(`${BASE_URL}/cart/items/${itemId}`, 'POST');
}

// Update the item's qty in the cart
// Will add the item to the order if not currently in the cart
// Sending info via the data payload instead of a long URL
export function setItemQtyInCart(itemId, newQty) {
  return sendRequest(`${BASE_URL}/cart/qty`, 'PUT', { itemId, newQty });
}

// Updates the order's (cart's) isPaid property to true
export function checkout() {
  // Changing data on the server, so make it a POST request
  return sendRequest(`${BASE_URL}/cart/checkout`, 'POST');
}

// Return all paid orders for the logged in user
export function getOrderHistory() {
  return sendRequest(`${BASE_URL}/history`);
}
```

**What this does**: Complete API interface for all order operations including cart management, checkout, and order history retrieval.

---

## 11. App Router and Pages (Stubs only at first)

### 11a. Main Entry Point

Update `src/main.jsx`:

```javascript
import './index.scss';
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './pages/App/App';

import { BrowserRouter as Router } from 'react-router-dom';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <Router>
      <App />
    </Router>
  </React.StrictMode>
);
```

**What this does**: Sets up React with React Router and renders the main App component.

### 11b. Main App Component

Create `src/pages/App/App.jsx`:

```javascript
import React, { useState } from 'react';
import { Routes, Route, Navigate } from 'react-router-dom';
import styles from './App.module.scss';
import { getUser } from '../../utilities/users-service';
import AuthPage from '../AuthPage/AuthPage';
import NewOrderPage from '../NewOrderPage/NewOrderPage';
import OrderHistoryPage from '../OrderHistoryPage/OrderHistoryPage';

export default function App() {
  const [user, setUser] = useState(getUser());
  return (
    <main className={styles.App}>
      { user ?
        <>
          <Routes>
            {/* client-side route that renders the component instance if the path matches the url in the address bar */}
            <Route path="/orders/new" element={<NewOrderPage user={user} setUser={setUser} />} />
            <Route path="/orders" element={<OrderHistoryPage user={user} setUser={setUser} />} />
            {/* redirect to /orders/new if path in address bar hasn't matched a <Route> above */}
            <Route path="/*" element={<Navigate to="/orders/new" />} />
          </Routes>
        </>
        :
        <AuthPage setUser={setUser} />
      }
    </main>
  );
}
```

**What this does**: Main application component that handles routing and user authentication state. Shows AuthPage when not logged in, and routes to order pages when authenticated.

### 11c. App Styles

Create `src/pages/App/App.module.scss`:

```scss
.App {
  height: 100%;
}
```

**What this does**: Basic styling for the main app container to take full height.

### 11d. Global Styles

Create `src/index.scss`:

```scss
// CSS Custom Properties (Variables) - these are what the components actually use
:root {
  --white: ghostwhite;
  --tan-1: #FBF9F6;
  --tan-2: #E7E2DD;
  --tan-3: #E2D9D1;
  --tan-4: #D3C1AE;
  --orange: orangered;
  --text-light: #968c84;
  --text-dark: black;
}

// Global SCSS Variables and Mixins (for future use)
$primary-color: #3498db;
$secondary-color: #2ecc71;
$text-color: #2c3e50;
$border-radius: 8px;
$shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

// Mixins
@mixin button-style($bg-color, $text-color: white) {
  background-color: $bg-color;
  color: $text-color;
  border: none;
  border-radius: $border-radius;
  padding: 12px 24px;
  cursor: pointer;
  transition: all 0.3s ease;
  
  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  }
}

@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin card-style {
  background: white;
  border-radius: $border-radius;
  box-shadow: $shadow;
  padding: 20px;
  transition: transform 0.3s ease;
  
  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  }
}

// Global styles
*, *:before, *:after {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
  'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
  sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: var(--tan-4);
  padding: 2vmin;
  height: 100vh;
}

code {
  font-family: source-code-pro, Menlo, Monaco, Consolas, 'Courier New',
    monospace;
}

#root {
  height: 100%;
}

// Utility classes from the original CSS
.align-ctr {
  text-align: center;
}

.align-rt {
  text-align: right;
}

.smaller {
  font-size: smaller;
}

.flex-ctr-ctr {
  display: flex;
  justify-content: center;
  align-items: center;
}

.flex-col {
  flex-direction: column;
}

.flex-j-end {
  justify-content: flex-end;
}

.scroll-y {
  overflow-y: scroll;
}

.section-heading {
  display: flex;
  justify-content: space-around;
  align-items: center;
  background-color: var(--tan-1);
  color: var(--text-dark);
  border: .1vmin solid var(--tan-3);
  border-radius: 1vmin;
  padding: .6vmin;
  text-align: center;
  font-size: 2vmin;
}

.form-container {
  padding: 3vmin;
  background-color: var(--tan-1);
  border: .1vmin solid var(--tan-3);
  border-radius: 1vmin;
}

p.error-message {
  color: var(--orange);
  text-align: center;
}

form {
  display: grid;
  grid-template-columns: 1fr 3fr;
  gap: 1.25vmin;
  color: var(--text-light);
}

label {
  font-size: 2vmin;
  display: flex;
  align-items: center;
}

input {
  padding: 1vmin;
  font-size: 2vmin;
  border: .1vmin solid var(--tan-3);
  border-radius: .5vmin;
  color: var(--text-dark);
  background-image: none !important; /* prevent lastpass */
  outline: none;
}

input:focus {
  border-color: var(--orange);
}

button, a.button {
  margin: 1vmin;
  padding: 1vmin;
  color: var(--white);
  background-color: var(--orange);
  font-size: 2vmin;
  font-weight: bold;
  text-decoration: none;
  text-align: center;
  border: .1vmin solid var(--tan-2);
  border-radius: .5vmin;
  outline: none;
  cursor: pointer;
}

button.btn-sm {
  font-size: 1.5vmin;
  padding: .6vmin .8vmin;
}

button.btn-xs {
  font-size: 1vmin;
  padding: .4vmin .5vmin;
}

button:disabled, form:invalid button[type="submit"] {
  cursor: not-allowed;
  background-color: var(--tan-4);
}

button[type="submit"] {
  grid-column: span 2;
  margin: 1vmin 0 0;
}

// Additional utility classes
.btn-primary {
  @include button-style($primary-color);
}

.btn-secondary {
  @include button-style($secondary-color);
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

.flex-center {
  @include flex-center;
}

.card {
  @include card-style;
}
```

**What this does**: Global CSS variables and utility classes for consistent styling across the entire application.

---

## 12. Components and integrating them into the pages

### 12a. Logo Component

Create `src/components/Logo/Logo.jsx`:

```javascript
import styles from './Logo.module.scss';

export default function Logo() {
  return (
    <div className={styles.Logo}>
      <div>GOAT</div>
      <div>CAFE</div>
    </div>
  );
}
```

Create `src/components/Logo/Logo.module.scss`:

```scss
.Logo {
  height: 12vmin;
  width: 12vmin;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 50%;
  background-color: var(--orange);
  color: var(--tan-1);
  font-size: 2.7vmin;
  border: .6vmin solid var(--tan-2);
}
```

**What this does**: Displays the GOAT CAFE logo as a circular orange badge with white text, used consistently across all pages.

### 12b. LoginForm Component

Create `src/components/LoginForm/LoginForm.jsx`:

```javascript
import { useState } from 'react';
import * as usersService from '../../utilities/users-service';

export default function LoginForm({ setUser }) {
  const [credentials, setCredentials] = useState({
    email: '',
    password: ''
  });
  const [error, setError] = useState('');

  function handleChange(evt) {
    setCredentials({ ...credentials, [evt.target.name]: evt.target.value });
    setError('');
  }

  async function handleSubmit(evt) {
    // Prevent form from being submitted to the server
    evt.preventDefault();
    try {
      // The promise returned by the login service method
      // will resolve to the user object included in the
      // payload of the JSON Web Token (JWT)
      const user = await usersService.login(credentials);
      setUser(user);
    } catch {
      setError('Log In Failed - Try Again');
    }
  }

  return (
    <div>
      <div className="form-container">
        <form autoComplete="off" onSubmit={handleSubmit}>
          <label>Email</label>
          <input type="text" name="email" value={credentials.email} onChange={handleChange} required />
          <label>Password</label>
          <input type="password" name="password" value={credentials.password} onChange={handleChange} required />
          <button type="submit">LOG IN</button>
        </form>
      </div>
      <p className="error-message">&nbsp;{error}</p>
    </div>
  );
}
```

Create `src/components/LoginForm/LoginForm.module.scss`:

```scss
/* This component uses global styles from index.scss */
```

**What this does**: Form component for user login with email and password fields, error handling, and integration with the users service.

### 12c. SignUpForm Component

Create `src/components/SignUpForm/SignUpForm.jsx`:

```javascript
import { Component } from "react";
import { signUp } from '../../utilities/users-service';

export default class SignUpForm extends Component {
  state = {
    name: '',
    email: '',
    password: '',
    confirm: '',
    error: ''
  };

  handleChange = (evt) => {
    this.setState({
      [evt.target.name]: evt.target.value,
      error: ''
    });
  };

  handleSubmit = async (evt) => {
    evt.preventDefault();
    try {
      const formData = {...this.state};
      delete formData.confirm;
      delete formData.error;
      // The promise returned by the signUp service method
      // will resolve to the user object included in the
      // payload of the JSON Web Token (JWT)
      const user = await signUp(formData);
      // Baby step
      this.props.setUser(user);
    } catch {
      // An error happened on the server
      this.setState({ error: 'Sign Up Failed - Try Again' });
    }
  };

  // We must override the render method
  // The render method is the equivalent to a function-based component
  // (its job is to return the UI)
  render() {
    const disable = this.state.password !== this.state.confirm;
    return (
      <div>
        <div className="form-container">
          <form autoComplete="off" onSubmit={this.handleSubmit}>
            <label>Name</label>
            <input type="text" name="name" value={this.state.name} onChange={this.handleChange} required />
            <label>Email</label>
            <input type="email" name="email" value={this.state.email} onChange={this.handleChange} required />
            <label>Password</label>
            <input type="password" name="password" value={this.state.password} onChange={this.handleChange} required />
            <label>Confirm</label>
            <input type="password" name="confirm" value={this.state.confirm} onChange={this.handleChange} required />
            <button type="submit" disabled={disable}>SIGN UP</button>
          </form>
        </div>
        <p className="error-message">&nbsp;{this.state.error}</p>
      </div>
    );
  }
}
```

Create `src/components/SignUpForm/SignUpForm.module.scss`:

```scss
/* This component uses global styles from index.scss */
```

**What this does**: Class-based component for user registration with name, email, password, and confirmation fields, including password matching validation.

### 12d. UserLogOut Component

Create `src/components/UserLogOut/UserLogOut.jsx`:

```javascript
import styles from './UserLogOut.module.scss';
import { logOut } from '../../utilities/users-service';

export default function UserLogOut({ user, setUser }) {
  function handleLogOut() {
    logOut();
    setUser(null);
  }

  return (
    <div className={styles.UserLogOut}>
      <div>{user.name}</div>
      <div className={styles.email}>{user.email}</div>
      <button className="btn-sm" onClick={handleLogOut}>LOG OUT</button>
    </div>
  );
}
```

Create `src/components/UserLogOut/UserLogOut.module.scss`:

```scss
.UserLogOut {
  font-size: 1.5vmin;
  color: var(--text-light);
  text-align: center;
}

.UserLogOut .email {
  font-size: smaller;
}
```

**What this does**: Displays current user information and provides a logout button that clears the user state and removes the JWT token.

### 12e. CategoryList Component

Create `src/components/CategoryList/CategoryList.jsx`:

```javascript
import styles from './CategoryList.module.scss';

export default function CategoryList({ categories, activeCat, setActiveCat }) {
  const cats = categories.map(cat =>
    <li
      key={cat}
      className={cat === activeCat ? styles.active : ''}
      onClick={() => setActiveCat(cat)}
    >
      {cat}
    </li>
  );
  return (
    <ul className={styles.CategoryList}>
      {cats}
    </ul>
  );
}
```

Create `src/components/CategoryList/CategoryList.module.scss`:

```scss
.CategoryList {
  color: var(--text-light);
  list-style: none;
  padding: 0;
  font-size: 1.7vw;
}

.CategoryList li {
  padding: .6vmin;
  text-align: center;
  border-radius: .5vmin;
  margin-bottom: .5vmin;
}

.CategoryList li:hover:not(.active) {
  cursor: pointer;
  background-color: var(--orange);
  color: var(--white);
}

.CategoryList li.active {
  color: var(--text-dark);
  background-color: var(--tan-1);
  border: .1vmin solid var(--tan-3);
}
```

**What this does**: Renders a list of food categories with clickable items that highlight the active category and allow users to filter menu items.

### 12f. MenuList Component

Create `src/components/MenuList/MenuList.jsx`:

```javascript
import styles from './MenuList.module.scss';
import MenuListItem from '../MenuListItem/MenuListItem';

export default function MenuList({ menuItems, handleAddToOrder }) {
  const items = menuItems.map(item =>
    <MenuListItem
      key={item._id}
      handleAddToOrder={handleAddToOrder}
      menuItem={item}
    />
  );
  return (
    <main className={styles.MenuList}>
      {items}
    </main>
  );
}
```

Create `src/components/MenuList/MenuList.module.scss`:

```scss
.MenuList {
  background-color: var(--tan-1);
  border: .1vmin solid var(--tan-3);
  border-radius: 2vmin;
  margin: 3vmin 0;
  padding: 3vmin;
  overflow-y: scroll;
}
```

**What this does**: Container component that renders a list of menu items filtered by the selected category, with scrolling for overflow content.

### 12g. MenuListItem Component

Create `src/components/MenuListItem/MenuListItem.jsx`:

```javascript
import styles from './MenuListItem.module.scss';

export default function MenuListItem({ menuItem, handleAddToOrder }) {
  return (
    <div className={styles.MenuListItem}>
      <div className={styles.emoji + ' ' + 'flex-ctr-ctr'}>{menuItem.emoji}</div>
      <div className={styles.name}>{menuItem.name}</div>
      <div className={styles.buy}>
        <span>${menuItem.price.toFixed(2)}</span>
        <button className="btn-sm" onClick={() => handleAddToOrder(menuItem._id)}>
          ADD
        </button>
      </div>
    </div>
  );
}
```

Create `src/components/MenuListItem/MenuListItem.module.scss`:

```scss
.MenuListItem {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 3vmin;
  padding: 2vmin;
  color: var(--text-light);
  background-color: var(--white);
  border: .1vmin solid var(--tan-3);
  border-radius: 1vmin;
  font-size: 4vmin;
}

.MenuListItem .emoji {
  height: 8vw;
  width: 8vw;
  font-size: 4vw;
  background-color: var(--tan-1);
  border: .1vmin solid var(--tan-3);
  border-radius: 1vmin;
}

.MenuListItem .buy {
  display: flex;
  flex-direction: column;
}

.MenuListItem .buy span {
  font-size: 1.7vw;
  text-align: center;
  color: var(--text-light);
}

.MenuListItem .name {
  font-size: 2vw;
  text-align: center;
  color: var(--text-light);
}
```

**What this does**: Individual menu item display with emoji, name, price, and an add button that triggers the add to cart functionality.

### 12h. LineItem Component

Create `src/components/LineItem/LineItem.jsx`:

```javascript
import styles from './LineItem.module.scss';

export default function LineItem({ lineItem, isPaid, handleChangeQty }) {
  return (
    <div className={styles.LineItem}>
      <div className="flex-ctr-ctr">{lineItem.item.emoji}</div>
      <div className="flex-ctr-ctr flex-col">
        <span className="align-ctr">{lineItem.item.name}</span>
        <span>{lineItem.item.price.toFixed(2)}</span>
      </div>
      <div className={styles.qty} style={{ justifyContent: isPaid && 'center' }}>
        {!isPaid &&
          <button
            className="btn-xs"
            onClick={() => handleChangeQty(lineItem.item._id, lineItem.qty - 1)}
          >−</button>
        }
        <span>{lineItem.qty}</span>
        {!isPaid &&
          <button
            className="btn-xs"
            onClick={() => handleChangeQty(lineItem.item._id, lineItem.qty + 1)}
          >+</button>
        }
      </div>
      <div className={styles.extPrice}>${lineItem.extPrice.toFixed(2)}</div>
    </div>
  );
}
```

Create `src/components/LineItem/LineItem.module.scss`:

```scss
.LineItem {
  width: 100%;
  display: grid;
  grid-template-columns: 3vw 15.35vw 5.75vw 5.25vw;
  padding: 1vmin 0;
  color: var(--text-light);
  background-color: var(--white);
  border-top: .1vmin solid var(--tan-3);
  font-size: 1.5vw;
}

.LineItem:last-child {
  border-bottom: .1vmin solid var(--tan-3);
}

.LineItem .qty {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 1.3vw;
}

.LineItem .extPrice {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  font-size: 1.3vw;
}

.LineItem button {
  margin: 0;
}
```

**What this does**: Displays individual line items in orders/cart with quantity controls (for unpaid orders) and calculated extended prices.

### 12i. OrderDetail Component

Create `src/components/OrderDetail/OrderDetail.jsx`:

```javascript
import styles from './OrderDetail.module.scss';
import LineItem from '../LineItem/LineItem';

// Used to display the details of any order, including the cart (unpaid order)
export default function OrderDetail({ order, handleChangeQty, handleCheckout }) {
  if (!order) return null;

  const lineItems = order.lineItems.map(item =>
    <LineItem
      lineItem={item}
      isPaid={order.isPaid}
      handleChangeQty={handleChangeQty}
      key={item._id}
    />
  );

  return (
    <div className={styles.OrderDetail}>
      <div className={styles.sectionHeading}>
        {order.isPaid ?
          <span>ORDER <span className="smaller">{order.orderId}</span></span>
          :
          <span>NEW ORDER</span>
        }
        <span>{new Date(order.updatedAt).toLocaleDateString()}</span>
      </div>
      <div className={`${styles.lineItemContainer} flex-ctr-ctr flex-col scroll-y`}>
        {lineItems.length ?
          <>
            {lineItems}
            <section className={styles.total}>
              {order.isPaid ?
                <span className={styles.right}>TOTAL&nbsp;&nbsp;</span>
                :
                <button
                  className="btn-sm"
                  onClick={handleCheckout}
                  disabled={!lineItems.length}
                >CHECKOUT</button>
              }
              <span>{order.totalQty}</span>
              <span className={styles.right}>${order.orderTotal.toFixed(2)}</span>
            </section>
          </>
          :
          <div className={styles.hungry}>Hungry?</div>
        }
      </div>
    </div>
  );
}
```

Create `src/components/OrderDetail/OrderDetail.module.scss`:

```scss
.OrderDetail {
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  padding: 3vmin;
  font-size: 2vmin;
  color: var(--text-light);
}

.OrderDetail .sectionHeading {
  width: 100%
}

.OrderDetail .lineItemContainer {
  margin-top: 3vmin;
  justify-content: flex-start;
  height: calc(100vh - 18vmin);
  width: 100%;
}

.OrderDetail .total {
  width: 100%;
  display: grid;
  grid-template-columns: 18.35vw 5.75vw 5.25vw;
  padding: 1vmin 0;
  color: var(--text-light);
  border-top: .1vmin solid var(--tan-3);
}

.OrderDetail .total span {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 1.5vw;
  color: var(--text-dark);
}

.OrderDetail .total span.right {
  display: flex;
  justify-content: flex-end;
}

.OrderDetail .hungry {
  position: absolute;
  top: 50vh;
  font-size: 2vmin;
}
```

**What this does**: Comprehensive order display component that shows order details, line items, totals, and checkout functionality for unpaid orders.

### 12j. OrderList Component

Create `src/components/OrderList/OrderList.jsx`:

```javascript
import OrderListItem from '../OrderListItem/OrderListItem';
import styles from './OrderList.module.scss';

export default function OrderList({ orders, activeOrder, handleSelectOrder }) {
  const orderItems = orders.map(o =>
    <OrderListItem
      order={o}
      isSelected={o === activeOrder}
      handleSelectOrder={handleSelectOrder}
      key={o._id}
    />
  );

  return (
    <main className={styles.OrderList}>
      {orderItems.length ?
        orderItems
        :
        <span className={styles.noOrders}>No Previous Orders</span>
      }
    </main>
  );
}
```

Create `src/components/OrderList/OrderList.module.scss`:

```scss
.OrderList {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: var(--tan-1);
  border: .1vmin solid var(--tan-3);
  border-radius: 2vmin;
  margin: 3vmin 0;
  padding: 3vmin;
  overflow-y: scroll;
}

.OrderList .noOrders {
  color: var(--text-light);
  font-size: 2vmin;
  position: absolute;
  top: calc(50vh);
}
```

**What this does**: Container component that displays a list of order summary items with scrolling and handles empty state messaging.

### 12k. OrderListItem Component

Create `src/components/OrderListItem/OrderListItem.jsx`:

```javascript
import styles from './OrderListItem.module.scss';

export default function OrderListItem({ order, isSelected, handleSelectOrder }) {
  return (
    <div className={`${styles.OrderListItem} ${isSelected ? styles.selected : ''}`} onClick={() => handleSelectOrder(order)}>
      <div>
        <div>Order Id: <span className="smaller">{order.orderId}</span></div>
        <div className="smaller">{new Date(order.updatedAt).toLocaleDateString()}</div>
      </div>
      <div className="align-rt">
        <div>${order.orderTotal.toFixed(2)}</div>
        <div className="smaller">{order.totalQty} Item{order.totalQty > 1 ? 's' : ''}</div>
      </div>
    </div>
  );
}
```

Create `src/components/OrderListItem/OrderListItem.module.scss`:

```scss
.OrderListItem {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 3vmin;
  padding: 2vmin;
  color: var(--text-light);
  background-color: var(--white);
  border: .2vmin solid var(--tan-3);
  border-radius: 1vmin;
  font-size: 2vmin;
  cursor: pointer;
}

.OrderListItem > div> div:first-child {
  margin-bottom: .5vmin;
}

.OrderListItem.selected {
  border-color: var(--orange);
  border-width: .2vmin;
  cursor: default;
}

.OrderListItem:not(.selected):hover {
  border-color: var(--orange);
  border-width: .2vmin;
}
```

**What this does**: Individual order summary item that displays order ID, date, total, and item count with selection highlighting and click handling.

---

## 13. Pages

### 13a. AuthPage Component

Create `src/pages/AuthPage/AuthPage.jsx`:

```javascript
import { useState } from 'react';
import styles from './AuthPage.module.scss';
import LoginForm from '../../components/LoginForm/LoginForm';
import SignUpForm from '../../components/SignUpForm/SignUpForm';
import Logo from '../../components/Logo/Logo';

export default function AuthPage({ setUser }) {
  const [showLogin, setShowLogin] = useState(true);

  return (
    <main className={styles.AuthPage}>
      <div>
        <Logo />
        <h3 onClick={() => setShowLogin(!showLogin)}>{showLogin ? 'SIGN UP' : 'LOG IN'}</h3>
      </div>
      {showLogin ? <LoginForm setUser={setUser} /> : <SignUpForm setUser={setUser} />}
    </main>
  );
}
```

Create `src/pages/AuthPage/AuthPage.module.scss`:

```scss
.AuthPage {
  height: 100%;
  display: flex;
  justify-content: space-evenly;
  align-items: center;
  background-color: var(--white);
  border-radius: 2vmin;
}

.AuthPage h3 {
  margin-top: 4vmin;
  text-align: center;
  color: var(--text-light);
  cursor: pointer;
}
```

**What this does**: Authentication page that toggles between login and signup forms with the GOAT CAFE logo and clickable header to switch modes.

### 13b. NewOrderPage Component

Create `src/pages/NewOrderPage/NewOrderPage.jsx`:

```javascript
import { useState, useEffect, useRef } from 'react';
import * as itemsAPI from '../../utilities/items-api';
import * as ordersAPI from '../../utilities/orders-api';
import styles from './NewOrderPage.module.scss';
import { Link, useNavigate } from 'react-router-dom';
import Logo from '../../components/Logo/Logo';
import MenuList from '../../components/MenuList/MenuList';
import CategoryList from '../../components/CategoryList/CategoryList';
import OrderDetail from '../../components/OrderDetail/OrderDetail';
import UserLogOut from '../../components/UserLogOut/UserLogOut';

export default function NewOrderPage({ user, setUser }) {
  const [menuItems, setMenuItems] = useState([]);
  const [activeCat, setActiveCat] = useState('');
  const [cart, setCart] = useState(null);
  const categoriesRef = useRef([]);
  const navigate = useNavigate();

  useEffect(function() {
    async function getItems() {
      const items = await itemsAPI.getAll();
      categoriesRef.current = items.reduce((cats, item) => {
        const cat = item.category.name;
        return cats.includes(cat) ? cats : [...cats, cat];
      }, []);
      setMenuItems(items);
      setActiveCat(categoriesRef.current[0]);
    }
    getItems();
    async function getCart() {
      const cart = await ordersAPI.getCart();
      setCart(cart);
    }
    getCart();
  }, []);

  /*-- Event Handlers --*/
  async function handleAddToOrder(itemId) {
    const updatedCart = await ordersAPI.addItemToCart(itemId);
    setCart(updatedCart);
  }

  async function handleChangeQty(itemId, newQty) {
    const updatedCart = await ordersAPI.setItemQtyInCart(itemId, newQty);
    setCart(updatedCart);
  }

  async function handleCheckout() {
    await ordersAPI.checkout();
    navigate('/orders');
  }

  return (
    <main className={styles.NewOrderPage}>
      <aside>
        <Logo />
        <CategoryList
          categories={categoriesRef.current}
          cart={setCart}
          setActiveCat={setActiveCat}
        />
        <Link to="/orders" className="button btn-sm">PREVIOUS ORDERS</Link>
        <UserLogOut user={user} setUser={setUser} />
      </aside>
      <MenuList
        menuItems={menuItems.filter(item => item.category.name === activeCat)}
        handleAddToOrder={handleAddToOrder}
      />
      <OrderDetail
        order={cart}
        handleChangeQty={handleChangeQty}
        handleCheckout={handleCheckout}
      />
    </main>
  );
}
```

Create `src/pages/NewOrderPage/NewOrderPage.module.scss`:

```scss
.NewOrderPage {
  height: 100%;
  display: grid;
  grid-template-columns: 1.6fr 3.5fr 3fr;
  grid-template-rows: 1fr;
  background-color: var(--white);
  border-radius: 2vmin;
  div {
    background-color: #fdfcf1;
    a {
      background-color: blue;
    }
  }
}

.NewOrderPage aside {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  margin: 3vmin 2vmin;
}
```

**What this does**: Main ordering page with three-column layout: sidebar (logo, categories, navigation), menu items filtered by category, and order detail/cart with checkout functionality.

### 13c. OrderHistoryPage Component

Create `src/pages/OrderHistoryPage/OrderHistoryPage.jsx`:

```javascript
import styles from './OrderHistoryPage.module.scss';
import { useState, useEffect } from 'react';
import { Link } from 'react-router-dom';
import * as ordersAPI from '../../utilities/orders-api';
import Logo from '../../components/Logo/Logo';
import UserLogOut from '../../components/UserLogOut/UserLogOut';
import OrderList from '../../components/OrderList/OrderList';
import OrderDetail from '../../components/OrderDetail/OrderDetail';

export default function OrderHistoryPage({ user, setUser }) {
  /*--- State --- */
  const [orders, setOrders] = useState([]);
  const [activeOrder, setActiveOrder] = useState(null);

  /*--- Side Effects --- */
  useEffect(function () {
    // Load previous orders (paid)
    async function fetchOrderHistory() {
      const orders = await ordersAPI.getOrderHistory();
      setOrders(orders);
      // If no orders, activeOrder will be set to null below
      setActiveOrder(orders[0] || null);
    }
    fetchOrderHistory();
  }, []);

  /*--- Event Handlers --- */
  function handleSelectOrder(order) {
    setActiveOrder(order);
  }

  /*--- Rendered UI --- */
  return (
    <main className={styles.OrderHistoryPage}>
      <aside className={styles.aside}>
        <Logo />
        <Link to="/orders/new" className="button btn-sm">NEW ORDER</Link>
        <UserLogOut user={user} setUser={setUser} />
      </aside>
      <OrderList
        orders={orders}
        activeOrder={activeOrder}
        handleSelectOrder={handleSelectOrder}
      />
      <OrderDetail
        order={activeOrder}
      />
    </main>
  );
}
```

Create `src/pages/OrderHistoryPage/OrderHistoryPage.module.scss`:

```scss
.OrderHistoryPage {
  height: 100%;
  display: grid;
  grid-template-columns: 1.6fr 3.5fr 3fr;
  grid-template-rows: 1fr;
  background-color: var(--white);
  border-radius: 2vmin;
}

.OrderHistoryPage .aside {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  margin: 3vmin 2vmin;
}
```

**What this does**: Order history page that displays a list of completed orders on the left, order details on the right, and navigation controls in the sidebar.

---

## 14. Explanation of SCSS and SCSS Modules for first time users

### What is SCSS?

SCSS (Sass) is a CSS preprocessor that extends CSS with additional features like variables, nesting, mixins, and more. It compiles down to regular CSS that browsers can understand.

### What are SCSS Modules?

SCSS Modules let you use the same CSS class name in different files without worrying about naming clashes. The module system automatically generates unique class names for each component.

### Key Benefits:

1. **Scoped Styling**: Styles are scoped to specific components
2. **No Naming Conflicts**: Same class names can exist in different files
3. **Better Organization**: Styles are co-located with components
4. **Type Safety**: Better IDE support and error catching

### Example of SCSS Modules:

**Button.module.scss**:
```scss
.error {
  background-color: red;
}
```

**Another-stylesheet.css**:
```css
.error {
  color: red;
}
```

**Button.js**:
```jsx
import styles from './Button.module.scss';

function Button() {
  return <button className={styles.error}>Error Button</button>;
}
```

**Result**: No clashes from other `.error` class names. The browser sees:
```html
<button class="Button_error_ax7yz">Error Button</button>
```

---

## 15. SCSS integration

### How to Use SCSS Modules in This Project:

1. **File Naming**: Use `.module.scss` extension for component-specific styles
2. **Import**: Import styles as an object: `import styles from './Component.module.scss'`
3. **Usage**: Reference classes as object properties: `className={styles.className}`
4. **Global Classes**: Use regular CSS classes for global styles (like utility classes)

### SCSS Features Used:

- **Variables**: CSS custom properties for consistent colors and spacing
- **Nesting**: Logical grouping of related styles
- **Grid Layout**: Modern CSS Grid for responsive layouts
- **Flexbox**: Flexible box layout for component alignment
- **Responsive Units**: `vmin` and `vw` for viewport-relative sizing

---

## 16. Testing the full app in the browser and expected output

### 16a. Starting the Application

1. **Start the backend server**:
   ```bash
   npm run server
   ```
   Expected: Console shows "Express app running on port 3000" and "Connected to MongoDB"

2. **Seed the database** (if not already done):
   ```bash
   node config/seed.js
   ```
   Expected: Console shows created items and categories

3. **Start the frontend** (in a new terminal):
   ```bash
   npm run dev
   ```
   Expected: Vite dev server starts and opens browser to `http://localhost:5173`

### 16b. Testing User Flow

#### Authentication Flow:
1. **Landing Page**: Should show GOAT CAFE logo with "LOG IN" form
2. **Sign Up**: Click "LOG IN" text to switch to signup form
3. **Create Account**: Fill out name, email, password, confirm password
4. **Validation**: Form should disable submit button until passwords match
5. **Success**: Should redirect to New Order page

#### New Order Flow:
1. **Page Layout**: Three-column grid with sidebar, menu, and cart
2. **Categories**: Left sidebar shows food categories (Sandwiches, Seafood, etc.)
3. **Menu Items**: Center shows items filtered by selected category
4. **Add to Cart**: Click "ADD" button on any menu item
5. **Cart Updates**: Right side shows cart with added items and quantities
6. **Quantity Controls**: Use +/- buttons to adjust item quantities
7. **Checkout**: Click "CHECKOUT" button when ready to complete order

#### Order History Flow:
1. **Navigation**: Click "PREVIOUS ORDERS" link from New Order page
2. **Order List**: Left side shows list of completed orders
3. **Order Details**: Right side shows details of selected order
4. **Navigation**: Click "NEW ORDER" to return to ordering

### 16c. Expected Visual Output

#### Overall Design:
- **Color Scheme**: Tan/beige background with orange accents
- **Layout**: Clean, modern interface with rounded corners
- **Typography**: Sans-serif fonts with consistent sizing
- **Spacing**: Consistent use of viewport-relative units (vmin, vw)

#### Component States:
- **Categories**: Hover effects with orange background, active state with border
- **Menu Items**: White cards with emoji icons and pricing
- **Cart**: Line items with quantity controls and calculated totals
- **Buttons**: Orange background with white text, disabled states for invalid forms

#### Responsive Behavior:
- **Grid Layout**: Three-column layout that adapts to viewport size
- **Scrolling**: Overflow content scrolls within designated areas
- **Touch Friendly**: Adequate button sizes for mobile interaction

### 16d. Common Issues and Solutions

#### Backend Issues:
- **Database Connection**: Ensure MongoDB is running and MONGO_URI is correct
- **Port Conflicts**: Check if port 3000 is available, change in .env if needed
- **JWT Errors**: Verify SECRET environment variable is set

#### Frontend Issues:
- **Proxy Errors**: Ensure Vite config proxy is set to correct backend port
- **Import Errors**: Check file paths and component exports
- **Styling Issues**: Verify SCSS files are properly imported and compiled

#### Data Issues:
- **Empty Menus**: Run seed script to populate database
- **Authentication Failures**: Check user credentials and JWT token handling
- **Cart Issues**: Verify order model methods and API endpoints

### 16e. Final Verification

The application should provide a complete restaurant ordering experience:
- ✅ User authentication (signup/login)
- ✅ Menu browsing by category
- ✅ Cart management with quantity controls
- ✅ Order checkout and completion
- ✅ Order history viewing
- ✅ Responsive design across devices
- ✅ Secure JWT-based authentication
- ✅ Real-time cart updates
- ✅ Clean, intuitive user interface

This completes the full-stack GOAT CAFE application with all components, pages, and functionality working together seamlessly.

---

## 17. Refactoring to a Dynamic App Router

After building and testing the initial application, we can refactor our routing to be more scalable and maintainable. Instead of hardcoding routes in `App.jsx`, we'll create a dynamic routing system that reads from a configuration file.

### 17a. Create Routes Configuration

Create `src/router/routes.js` to define all application routes in one place:

```javascript
import NewOrderPage from '../pages/NewOrderPage/NewOrderPage';
import OrderHistoryPage from '../pages/OrderHistoryPage/OrderHistoryPage';

const routes = [
	{
		Component: NewOrderPage,
		key: 'NewOrder',
		path: '/orders/new'
	},
	{
		Component: OrderHistoryPage,
		key: 'OrderHistory',
		path: '/orders'
	}
];

export default routes;
```
**What this does**: Centralizes all route definitions into an array of objects, making it easy to add or remove pages without touching the main router component.

### 17b. Create the App Router Component

Create `src/router/index.jsx`. This component will replace `src/pages/App/App.jsx` as the main router.

```javascript
import { BrowserRouter as Router, Routes, Route, Navigate } from 'react-router-dom';
import routes from './routes';
import { useState } from 'react'
import styles from './AppRouter.module.scss';
import { getUser } from '../utilities/users-service';
import AuthPage from '../pages/AuthPage/AuthPage';

const AppRouter = () => {
	const [user, setUser] = useState(getUser())
	return (
		<Router>
			<main className={styles.App}>
			{
				user ?
			<>
			<Routes>
				{routes.map(({ Component, key, path }) => (
					<Route
						key={key}
						path={path}
						element={
						<Component 
							page={key} 
							user={user}
							setUser={setUser}
						/>
						}
					></Route>
				))}
				<Route path='/*' element={<Navigate to="/orders/new"/>}/>
			</Routes>
			</>
			:
		<AuthPage setUser={setUser}/>
		}
		</main>
		</Router>
	);
};

export default AppRouter;
```
**What this does**: This component dynamically generates routes by mapping over the `routes.js` configuration. It handles user authentication state and renders either the authenticated routes or the `AuthPage`.

Create the corresponding style file `src/router/AppRouter.module.scss`:
```scss
.App {
    height: 100%;
  }
```

### 17c. Update the Main Entry Point

Finally, update `src/main.jsx` to use the new `AppRouter`.

```javascript
import {StrictMode} from "react";
import { createRoot } from "react-dom/client";
import AppRouter from './router/index.jsx';
import './index.scss';
const root = createRoot(document.getElementById("root"))
root.render(<StrictMode><AppRouter/></StrictMode>)
```
**What this does**: Changes the application's entry point to render `AppRouter` instead of the old `App` component, completing the refactor.

### 17d. Clean Up

You can now safely delete the `src/pages/App/` directory as it has been replaced by the `src/router/` setup.

This refactored setup provides a much cleaner and more scalable way to manage routes as your application grows.

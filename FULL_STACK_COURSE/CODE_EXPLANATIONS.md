# Code Explanations: Backend Models, Config, Controllers, and Routes

## Overview
This file provides detailed, line-by-line explanations of the backend code structure, explaining what each line does and why it's important. This helps you understand the architecture and make informed decisions when modifying the code.

## Table of Contents
1. [Configuration Files](#configuration-files)
2. [Database Models](#database-models)
3. [Controllers](#controllers)
4. [Routes](#routes)

---

## 1. Configuration Files

### `config/database.js`
```javascript
import mongoose from 'mongoose';
import dotenv from 'dotenv';

dotenv.config();    

mongoose.connect(process.env.MONGO_URI)

mongoose.connection.once('open', () => {
    console.log('Mongo is showing us love')
})
```
- **`import` statements**: Imports the `mongoose` library for MongoDB interaction and `dotenv` to load environment variables.
- **`dotenv.config()`**: Loads variables from the `.env` file into `process.env`.
- **`mongoose.connect(...)`**: Connects to the MongoDB database using the connection string stored in the `MONGO_URI` environment variable.
- **`mongoose.connection.once('open', ...)`**: Sets up a one-time event listener that logs a success message to the console once the connection to MongoDB is established.

### `config/checkToken.js`
```javascript
import jwt from 'jsonwebtoken';

export default (req, res, next) => {
    let token = req.get('Authorization')
    if(token){
        token = token.split(' ')[1]
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
- **Purpose**: This is an Express middleware function designed to check for a JWT on incoming requests.
- **`req.get('Authorization')`**: Retrieves the `Authorization` header from the request.
- **`token.split(' ')[1]`**: The token is expected in the format "Bearer <token>". This line splits the string and takes the actual token part.
- **`jwt.verify(...)`**: Decodes and verifies the token using the `SECRET` from your `.env` file.
    - If the token is valid, the decoded `user` payload is attached to the request object (`req.user`). The expiration date is also attached (`req.exp`).
    - If the token is invalid or expired, `err` will exist, and `req.user` is set to `null`.
- **`next()`**: This is a crucial function in middleware. It passes control to the next middleware function in the stack. This middleware *never stops* the request; it only adds information (`req.user`) to it.

### `config/ensureLoggedIn.js`
```javascript
export default (req, res, next ) => {
    if(req.user) return next()
    res.status('401').json({ msg: 'Unauthorized You Shall Not Pass'})
}
```
- **Purpose**: This middleware protects routes by ensuring a user is logged in.
- **`if(req.user)`**: It checks if the `req.user` object was successfully attached by the `checkToken` middleware.
- **`return next()`**: If `req.user` exists, the user is authenticated, and the request is allowed to proceed to the actual route handler.
- **`res.status('401')...`**: If `req.user` is null, it means the user is not authenticated. The middleware stops the request-response cycle and sends a `401 Unauthorized` status back to the client.

---

## 2. Database Models

### `models/user.js`
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
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, SALT_ROUNDS);
  return next();
});

export default mongoose.model('User', userSchema);
```
- **`userSchema`**: Defines the structure for documents in the `users` collection. It includes validation rules like `required`, `unique`, and `minlength`.
- **`timestamps: true`**: Automatically adds `createdAt` and `updatedAt` fields.
- **`toJSON: { transform ... }`**: A Mongoose option that allows you to modify the object when it's converted to JSON (e.g., before being sent in an API response). Here, it's used to delete the `password` field for security.
- **`userSchema.pre('save', ...)`**: A pre-save hook (middleware) that runs before a user document is saved to the database.
    - `if (!this.isModified('password'))`: This ensures the hashing logic only runs if the password field is new or has been changed.
    - `bcrypt.hash(...)`: Hashes the plain-text password with a salt factor of 6.

### `models/order.js`
- **`lineItemSchema`**: This is a sub-schema that defines the structure of individual items within an order's `lineItems` array.
- **`lineItemSchema.virtual('extPrice').get(...)`**: A virtual property that calculates the extended price (`quantity * item.price`) for a line item on the fly without storing it in the database.
- **`orderSchema`**: The main schema for an order.
    - **`user: { type: Schema.Types.ObjectId, ref: 'User' }`**: This creates a relationship. It stores a reference to a `User` document's `_id`. The `ref` tells Mongoose which model to look in when populating.
    - **`lineItems: [lineItemSchema]`**: An array of documents that must conform to the `lineItemSchema`.
- **`orderSchema.virtual(...)`**: Defines several virtual properties for calculated values like `orderTotal`, `totalQty`, and a formatted `orderId`.
- **`orderSchema.statics.getCart`**: Defines a static method on the `Order` model itself. Static methods are helper functions that operate on the entire collection. `getCart` is used to find an existing unpaid order for a user or create a new one if none exists (`upsert: true`).
- **`orderSchema.methods.addItemToCart` & `setItemQty`**: Defines instance methods. These methods are available on individual `order` documents and are used to modify a specific cart's `lineItems` array.

---

## 3. Controllers

### `controllers/api/users.js`
```javascript
// ... imports
const dataController = {
  async create (req, res, next) { /* ... */ },
  async login (req, res, next) { /* ... */ }
}
const apiController = {
  auth (req, res) { /* ... */ }
}
// ...
```
- **Two-Controller Pattern**: This file separates logic into two parts:
    1.  **`dataController`**: Interacts with the database (the "Model" part of MVC). It creates users, finds users, and prepares data. It doesn't send the final response but passes data to the next middleware via `res.locals`.
    2.  **`apiController`**: Handles the final response to the client (the "View" part of MVC, but for an API). It takes the data prepared by `dataController` and formats it into a JSON response.
- **`create` function**:
    - Creates a new `User` in the database.
    - Calls `createJWT(user)` to generate a token.
    - Attaches the `user` and `token` to `res.locals.data` to be used by `apiController.auth`.
- **`login` function**:
    - Finds a user by email.
    - Uses `bcrypt.compare` to check if the provided password matches the hashed password in the database.
    - If credentials are valid, it generates a JWT and attaches data to `res.locals`.
- **`auth` function**:
    - Takes the `token` and `user` from `res.locals.data`.
    - Sends them back to the client in a structured JSON object: `{ token: "...", user: {...} }`.

### `controllers/api/orders.js`
- This file follows a more traditional controller pattern where each exported function is a complete request handler.
- **`cart` function**: Uses the `Order.getCart` static method to find or create a cart and sends it as a response.
- **`addToCart` function**: First gets the cart, then uses the `cart.addItemToCart` instance method to modify it, saves the changes, and sends the updated cart back.
- **`checkout` function**: Gets the cart, marks `isPaid` as `true`, saves it, and sends the final, paid order back.
- **`history` function**: Finds all orders for the current user where `isPaid` is `true` and sorts them by most recently updated.

---

## 4. Routes

### `routes/api/users.js`
```javascript
// ... imports
// POST /api/users
router.post('/', dataController.create, apiController.auth)
// POST /api/users/login
router.post('/login', dataController.login, apiController.auth)
// GET /api/users/check-token
router.get('/check-token', ensureLoggedIn, checkToken)
```
- **Middleware Chain**: The routes demonstrate the power of middleware. For the `POST /` route:
    1. Express receives the request.
    2. It first executes `dataController.create`.
    3. `dataController.create` calls `next()`, passing control to `apiController.auth`.
    4. `apiController.auth` sends the final response.
- **Protected Route**: The `/check-token` route first runs the `ensureLoggedIn` middleware. If that middleware calls `next()`, the request proceeds to the `checkToken` controller function. If not, `ensureLoggedIn` sends a `401` response and the chain stops.

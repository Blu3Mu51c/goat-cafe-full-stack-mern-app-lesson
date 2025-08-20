# 05. Controllers and API Routes

## Overview
This file covers implementing controllers for business logic and setting up API routes for items and orders.

## Table of Contents
1. [Controller Pattern](#controller-pattern)
2. [Items Controller](#items-controller)
3. [Orders Controller](#orders-controller)
4. [API Routes](#api-routes)
5. [Middleware Integration](#middleware-integration)
6. [Testing Controllers](#testing-controllers)

---

## Controller Pattern

### Understanding the Pattern
Controllers handle the business logic between routes and models. We use a two-controller pattern:

1. **Data Controllers**: Handle data operations and business logic
2. **API Controllers**: Format and send responses

### Why This Pattern?
- **Separation of Concerns**: Business logic vs. response formatting
- **Reusability**: Controllers can be used by multiple routes
- **Testability**: Easy to unit test business logic
- **Maintainability**: Clear structure for complex applications

---

## Items Controller

### Create controllers/api/items.js
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

### Controller Methods Explained

#### 1. Index (List All Items)
```javascript
async index(req, res, next) {
    try {
        const items = await Item.find({});
        res.locals.data.items = items;
        next();
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
}
```

**Purpose**: Retrieve all items from database
**Process**: Query database → Store in locals → Continue to API controller
**Response**: Array of all items

#### 2. Show (Get Single Item)
```javascript
async show(req, res, next) {
    try {
        const item = await Item.findById(req.params.id);
        if (!item) throw new Error('Item not found');
        res.locals.data.item = item;
        next();
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
}
```

**Purpose**: Retrieve single item by ID
**Process**: Find by ID → Check existence → Store in locals → Continue
**Response**: Single item object

#### 3. Create (Add New Item)
```javascript
async create(req, res, next) {
    try {
        const item = await Item.create(req.body);
        res.locals.data.item = item;
        next();
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
}
```

**Purpose**: Create new item in database
**Process**: Create from request body → Store in locals → Continue
**Response**: Newly created item

#### 4. Update (Modify Existing Item)
```javascript
async update(req, res, next) {
    try {
        const item = await Item.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true, runValidators: true }
        );
        if (!item) throw new Error('Item not found');
        res.locals.data.item = item;
        next();
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
}
```

**Purpose**: Update existing item
**Options**:
- **`new: true`**: Return updated document
- **`runValidators: true`**: Run schema validation on update

#### 5. Delete (Remove Item)
```javascript
async delete(req, res, next) {
    try {
        const item = await Item.findByIdAndDelete(req.params.id);
        if (!item) throw new Error('Item not found');
        res.locals.data.item = item;
        next();
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
}
```

**Purpose**: Remove item from database
**Process**: Find and delete → Store deleted item in locals → Continue

---

## Orders Controller

### Create controllers/api/orders.js
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

### Orders Controller Features

#### 1. User-Specific Data
```javascript
const orders = await Order.find({ user: req.user._id })
    .populate('items.item')
    .sort({ createdAt: -1 });
```

**Security**: Only return orders belonging to authenticated user
**Population**: Include full item details
**Sorting**: Most recent orders first

#### 2. Authorization Checks
```javascript
if (order.user._id.toString() !== req.user._id.toString()) {
    throw new Error('Unauthorized to view this order');
}
```

**Purpose**: Ensure users can only access their own orders
**Comparison**: Convert ObjectIds to strings for comparison

#### 3. Automatic User Assignment
```javascript
req.body.user = req.user._id;
```

**Security**: Automatically assign user from JWT token
**Prevention**: Users cannot create orders for other users

---

## API Routes

### Create routes/api/items.js
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

### Create routes/api/orders.js
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

### Route Structure Explained

#### 1. RESTful Design
- **GET**: Retrieve data
- **POST**: Create new data
- **PUT**: Update existing data
- **DELETE**: Remove data

#### 2. Middleware Chain
```javascript
router.get('/', dataController.index, apiController.index);
```

**Flow**: Request → Data Controller → API Controller → Response

#### 3. Parameter Extraction
```javascript
router.get('/:id', dataController.show, apiController.show);
```

**`:id`**: URL parameter accessible via `req.params.id`

---

## Middleware Integration

### Protected Routes in app-server.js
```javascript
// Public routes (no authentication required)
app.use('/api/users', userRoutes);

// Protected routes (authentication required)
app.use('/api/items', checkToken, ensureLoggedIn, itemRoutes);
app.use('/api/orders', checkToken, ensureLoggedIn, orderRoutes);
```

### Middleware Flow
```
Request → checkToken → ensureLoggedIn → Route Handler → Response
```

**`checkToken`**: Extracts and verifies JWT token
**`ensureLoggedIn`**: Ensures user is authenticated
**Route Handler**: Processes the specific request

---

## Testing Controllers

### Start Your Server
```bash
npm run server
```

### Test Items API

#### 1. Create Item (Requires Authentication)
```bash
curl -X POST http://localhost:3000/api/items \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{
    "name": "Margherita Pizza",
    "emoji": "🍕",
    "category": "pizza",
    "price": 14.99
  }'
```

#### 2. Get All Items
```bash
curl -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  http://localhost:3000/api/items
```

#### 3. Get Single Item
```bash
curl -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  http://localhost:3000/api/items/ITEM_ID_HERE
```

### Test Orders API

#### 1. Create Order
```bash
curl -X POST http://localhost:3000/api/orders \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{
    "items": [
      {
        "item": "ITEM_ID_HERE",
        "qty": 2
      }
    ]
  }'
```

#### 2. Get User's Orders
```bash
curl -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  http://localhost:3000/api/orders
```

---

## Advanced Controller Features

### Error Handling
```javascript
// Global error handler
app.use((error, req, res, next) => {
    console.error(error.stack);
    res.status(500).json({ error: 'Something went wrong!' });
});

// Async error wrapper
const asyncHandler = (fn) => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};

// Usage
router.get('/', asyncHandler(dataController.index), apiController.index);
```

### Input Validation
```javascript
// Add validation middleware
import { body, validationResult } from 'express-validator';

const validateItem = [
    body('name').trim().isLength({ min: 2, max: 50 }),
    body('price').isFloat({ min: 0.01 }),
    body('category').isIn(['pizza', 'drink', 'dessert', 'appetizer']),
    (req, res, next) => {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({ errors: errors.array() });
        }
        next();
    }
];

// Apply validation
router.post('/', validateItem, dataController.create, apiController.create);
```

### Pagination
```javascript
async index(req, res, next) {
    try {
        const page = parseInt(req.query.page) || 1;
        const limit = parseInt(req.query.limit) || 10;
        const skip = (page - 1) * limit;
        
        const items = await Item.find({})
            .skip(skip)
            .limit(limit);
        
        const total = await Item.countDocuments({});
        
        res.locals.data = {
            items,
            pagination: {
                page,
                limit,
                total,
                pages: Math.ceil(total / limit)
            }
        };
        next();
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
}
```

---

## Common Issues and Solutions

### Issue: Controllers Not Working
**Problem**: Routes return 404 or errors

**Solutions**:
1. Check import/export statements
2. Verify route registration in `app-server.js`
3. Ensure middleware order is correct

### Issue: Authentication Bypass
**Problem**: Protected routes accessible without token

**Solutions**:
1. Verify `checkToken` middleware is included
2. Check `ensureLoggedIn` is working
3. Test with and without valid tokens

### Issue: Data Not Persisting
**Problem**: Create/update operations not saving

**Solutions**:
1. Check database connection
2. Verify model validation
3. Check for required fields

---

## Next Steps

After completing this setup:

1. **Test All Routes**: Verify CRUD operations work
2. **Test Authentication**: Ensure protected routes are secure
3. **Test Error Handling**: Verify proper error responses
4. **Move to Next File**: Continue with [06_FRONTEND_SETUP.md](./06_FRONTEND_SETUP.md)

## Verification Checklist

- [ ] Items controller implemented with all CRUD operations
- [ ] Orders controller implemented with user-specific access
- [ ] API routes configured correctly
- [ ] Middleware integration working
- [ ] Protected routes require authentication
- [ ] Error handling implemented
- [ ] All CRUD operations tested
- [ ] Authorization working correctly
- [ ] Data validation in place
- [ ] No authentication bypass possible

Once all items are checked, you're ready to proceed to the next file!

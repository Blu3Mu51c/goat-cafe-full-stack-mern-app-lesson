# 07. Utilities and Services Layer

## Overview
This file covers implementing the service layer that handles API calls, business logic, and data management between the frontend and backend.

## Table of Contents
1. [Service Layer Architecture](#service-layer-architecture)
2. [Centralized Request Utility](#centralized-request-utility)
3. [User Authentication Services](#user-authentication-services)
4. [Items API Services](#items-api-services)
5. [Orders API Services](#orders-api-services)
6. [Error Handling](#error-handling)
7. [Testing Services](#testing-services)

---

## Service Layer Architecture

### Understanding the Pattern
The service layer acts as an abstraction between the UI components and the API:

```
UI Components → Service Layer → API Layer → Backend
```

### Benefits of Service Layer
- **Separation of Concerns**: UI logic separate from API logic
- **Reusability**: Services can be used by multiple components
- **Testability**: Easy to mock services for testing
- **Maintainability**: Centralized API logic and error handling

---

## Centralized Request Utility

### Create src/utilities/send-request.js
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
**What this does**: This core utility handles all API requests. It automatically adds the JWT token to the `Authorization` header, sets the `Content-Type` for payloads, and throws an error if the server returns a non-OK response.

---

## User Authentication Services

### Create src/utilities/users-api.js
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

### Create src/utilities/users-service.js
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

### Service Functions Explained

#### 1. API Layer (users-api.js)
**Purpose**: Handle HTTP requests to the backend authentication endpoints.

#### 2. Service Layer (users-service.js)
**Purpose**: Handle business logic related to authentication, such as storing the JWT and decoding user information.

---

## Items API Services

### Create src/utilities/items-api.js
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

---

## Orders API Services

### Create src/utilities/orders-api.js
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

---

## Next Steps

After completing this setup:

1. **Test All Services**: Verify API calls work correctly
2. **Move to Next File**: Continue with [08_COMPONENT_STATE_LOGIC.md](./08_COMPONENT_STATE_LOGIC.md)

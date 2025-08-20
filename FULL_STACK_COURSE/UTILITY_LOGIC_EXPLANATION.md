# Utility Logic Explanation: Frontend Service Layer

## Overview
This file explains the utility logic that powers the frontend application. The utilities handle API communication, data management, and authentication state, providing a clean interface between React components and the backend services.

## Table of Contents
1. [Service Layer Architecture](#service-layer-architecture)
2. [The `send-Request` Core Utility](#send-request-utility)
3. [Authentication Utilities (`users-api` & `users-service`)](#authentication-utilities)
4. [Data Fetching Utilities (`items-api` & `orders-api`)](#data-fetching-utilities)

---

## 1. Service Layer Architecture

### Purpose and Benefits
The service layer acts as an abstraction between React components and the backend API, providing:

- **Separation of Concerns**: Components focus on UI, utilities handle data logic.
- **Reusability**: API logic can be shared across multiple components.
- **Maintainability**: A centralized place for API endpoint changes.
- **Error Handling**: Consistent error handling for API calls.

### Architecture Pattern in This Project
Your project uses a clear two-level utility structure:
1.  **Core API Utility (`send-request.js`)**: A single, powerful function that handles all `fetch` calls, automatically attaching JWTs and handling basic response validation.
2.  **Domain-Specific API Modules (`users-api.js`, `items-api.js`, etc.)**: These files define specific functions for each type of data (e.g., `signUp`, `getCart`). They import and use `send-request.js` to do the actual fetching.
3.  **Service Modules (`users-service.js`)**: These files handle business logic that involves more than just an API call, such as interacting with `localStorage`.

```
React Component → Service Module (e.g., users-service) → API Module (e.g., users-api) → Core Utility (send-request) → Backend
```

---

## 2. The `send-Request` Core Utility

### `src/utilities/send-request.js`
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
- **Purpose**: This is the heart of the frontend's communication with the backend. Every API call goes through this function.
- **Parameters**: It takes a `url`, a `method` (defaulting to 'GET'), and an optional `payload` (for 'POST' or 'PUT' requests).
- **Payload Handling**: If a `payload` is provided, it automatically sets the `Content-Type` header to `application/json` and stringifies the payload object.
- **JWT Authentication**:
    - It calls `getToken()` from `users-service` to retrieve the current JWT from `localStorage`.
    - If a token exists, it automatically adds the `Authorization: Bearer <token>` header to the request. This is how your backend's protected routes are accessed.
- **Response Handling**:
    - It makes the `fetch` call with the configured options.
    - `if (res.ok)`: This is a crucial check. `res.ok` is true only for HTTP status codes in the 200-299 range. If the request was successful, it parses the JSON body and returns it.
    - `throw new Error(...)`: If the status code is 4xx or 5xx, `res.ok` is false, and this line throws an error, which will be caught by the calling function in the component.

---

## 3. Authentication Utilities (`users-api` & `users-service`)

### `src/utilities/users-api.js`
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
- **Purpose**: This file defines the functions that map directly to the backend's user authentication API endpoints.
- **`sendRequest` in Action**: Notice how simple these functions are. All the complexity of `fetch`, headers, and tokens is handled by `sendRequest`. These functions just provide the correct URL, method, and payload.

### `src/utilities/users-service.js`
```javascript
import * as usersAPI  from './users-api';

export async function signUp(userData) {
  const response = await usersAPI.signUp(userData);
  localStorage.setItem('token', response.token);
  return response.user;
}

export async function login(credentials) {
  const response = await usersAPI.login(credentials);
  localStorage.setItem('token', response.token);
  return response.user;
}

export function getToken() { /* ... */ }
export function getUser() { /* ... */ }
export function logOut() { /* ... */ }
```
- **Purpose**: This file handles the *business logic* of authentication on the frontend.
- **`signUp` & `login`**: These functions orchestrate the authentication process:
    1. They call the corresponding function in `users-api.js` to make the network request.
    2. The backend sends back `{ token: "...", user: {...} }`.
    3. They **store the token** in `localStorage`. This is a crucial step for session persistence.
    4. They **return the user object**, which the component will use to set its state.
- **`getToken`**: This function is responsible for retrieving the token from `localStorage` and, importantly, **validating its expiration**. If the token is expired or malformed, it is removed from storage, and `null` is returned, effectively logging the user out.
- **`getUser`**: This function decodes the user payload from a valid token without needing an extra API call.
- **`logOut`**: This function simply removes the token from `localStorage`, which is the sole indicator of a user's session on the frontend.

---

## 4. Data Fetching Utilities (`items-api` & `orders-api`)

### `src/utilities/items-api.js`
```javascript
import sendRequest from './send-request';
const BASE_URL = '/api/items';

export function getAll() {
  return sendRequest(BASE_URL);
}
// ...
```
### `src/utilities/orders-api.js`
```javascript
import sendRequest from './send-request';
const BASE_URL = '/api/orders';

export function getCart() {
  return sendRequest(`${BASE_URL}/cart`);
}

export function addItemToCart(itemId) {
  return sendRequest(`${BASE_URL}/cart/items/${itemId}`, 'POST');
}
// ...
```
- **Purpose**: These files define functions for interacting with the protected `items` and `orders` API endpoints.
- **Simplicity**: Because `sendRequest` automatically includes the authentication token, these functions are extremely clean. They only need to know the correct URL and method for each endpoint. This makes adding new API calls very easy and maintainable.

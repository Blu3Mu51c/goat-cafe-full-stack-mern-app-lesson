# Debug Guide: Troubleshooting Common Issues

## Overview
This guide provides solutions to common problems you may encounter while building and running this full-stack application. Each section includes the error message, likely causes, and step-by-step solutions.

## Table of Contents
1. [Backend & Server Issues](#backend-issues)
2. [Frontend & React Issues](#frontend-issues)
3. [Authentication & JWT Issues](#auth-issues)
4. [Database & Mongoose Issues](#database-issues)

---

## 1. Backend & Server Issues

### Issue: Server Won't Start or Crashes Immediately

**Error Message:** `SyntaxError: Cannot use import statement outside a module` or similar import errors.

**Root Cause:** The project is configured to use ES Modules (`import`/`export`), but there might be a misconfiguration.

**Solution:**
1.  **Check `package.json`**: Ensure this line exists: `"type": "module"`.
2.  **Check File Extensions**: Make sure you are including the `.js` extension on all relative imports in your backend files (e.g., `import User from '../models/user.js'`). CommonJS (`require`) does not need this, but ES Modules do.
3.  **Consistent Syntax**: Ensure you are using `import`/`export` syntax throughout your backend code, not `require`/`module.exports`.

### Issue: Port Conflict

**Error Message:** `Error: listen EADDRINUSE: address already in use :::3000`

**Root Cause:** Another application (or a stray process from a previous run) is already using port 3000.

**Solution:**
1.  **Find and Stop the Process**:
    -   On Mac/Linux: `lsof -i :3000` to find the Process ID (PID), then `kill -9 <PID>`.
    -   On Windows: `netstat -ano | findstr :3000` to find the PID, then `taskkill /PID <PID> /F`.
2.  **Change the Port**: Temporarily change the `PORT` in your `.env` file to something else (e.g., `3001`) and restart the server.

---

## 2. Frontend & React Issues

### Issue: White Screen or "Cannot GET /" on a route like `/orders`

**Root Cause:** This is a classic Single Page Application (SPA) issue. The backend server doesn't know how to handle frontend routes directly.

**Solution:**
-   **Verify Catch-All Route**: Ensure the catch-all route in `app-server.js` is correctly configured and is placed *after* all your API routes. It should serve your `index.html` file for any non-API request.
    ```javascript
    // In app-server.js
    app.get('*', (req, res) => {
        if (req.path.startsWith('/api/')) {
            return res.status(404).json({ error: 'API endpoint not found' });
        }
        res.sendFile(path.resolve(path.join(__dirname, indexPath)));
    });
    ```

### Issue: API Calls Fail with "Failed to fetch" (Network Error)

**Root Cause:** The frontend can't reach the backend. This is almost always a proxy issue in development.

**Solution:**
1.  **Check `vite.config.js`**: Make sure the proxy is set up correctly to forward `/api` requests to your backend server's port (e.g., `http://localhost:3000`).
2.  **Is the Backend Running?**: Ensure your backend server (`npm run server`) is running in a separate terminal and hasn't crashed.
3.  **Check Ports**: Double-check that the `target` port in your Vite proxy config matches the `PORT` your Express server is running on.

### Issue: State Not Updating as Expected in React Components

**Root Cause:** Usually caused by direct mutation of state or incorrect use of `useEffect` dependencies.

**Solution:**
1.  **Never Mutate State Directly**: Always use the setter function from `useState` and create a new object or array.
    ```javascript
    // BAD
    const newCredentials = credentials;
    newCredentials.email = 'new@email.com';
    setCredentials(newCredentials);

    // GOOD
    setCredentials({ ...credentials, email: 'new@email.com' });
    ```
2.  **Check `useEffect` Dependencies**: If your `useEffect` should re-run when a value changes, make sure that value is in the dependency array. If it should only run once, make sure the array is empty (`[]`).

---

## 3. Authentication & JWT Issues

### Issue: Login/Signup Works, but User is Immediately Logged Out or Can't Access Protected Routes

**Error Message:** `401 Unauthorized` on API calls after logging in.

**Root Cause:** The JWT is not being correctly stored, retrieved, or sent with API requests.

**Solution:**
1.  **Check Browser `localStorage`**: After logging in, open the browser's DevTools, go to the "Application" tab, and check "Local Storage". You should see a `key` named `token` with a long string value.
2.  **Verify `users-service.js`**: Ensure `signUp` and `login` are correctly storing the token: `localStorage.setItem('token', response.token);`.
3.  **Verify `send-request.js`**: This is the most critical part. Make sure it's retrieving the token from localStorage and adding it to the `Authorization` header on **every** request. A `console.log(token)` inside this file can be very helpful for debugging.

### Issue: "Bad Credentials" on Login

**Root Cause:** The email was not found, or the password comparison failed.

**Solution:**
1.  **Check Database**: Make sure a user with that email actually exists in your MongoDB `users` collection.
2.  **Check Password Hashing**: The `bcrypt.compare` function in your `controllers/api/users.js` `login` function is failing. This can happen if the password wasn't hashed correctly on signup. Check the `pre('save', ...)` middleware in `models/user.js`.
3.  **Typos**: It's often a simple typo in the email or password being entered.

---

## 4. Database & Mongoose Issues

### Issue: Data from `.populate()` is `null` or Missing

**Root Cause:** The `ref` in your schema does not match the model name you're trying to populate from, or the referenced `_id` does not exist.

**Solution:**
1.  **Check Schema `ref`**: In your schema (e.g., `order.js`), make sure the `ref` value is the string name you used when creating the other model.
    ```javascript
    // In orderSchema
    user: { type: Schema.Types.ObjectId, ref: 'User' } // 'User' must match the name in mongoose.model('User', userSchema)
    ```
2.  **Verify Data Integrity**: Check the actual documents in your database. Does the `ObjectId` stored in the order's `user` field actually correspond to a user's `_id` in the `users` collection? If not, the population will return `null`.

### Issue: Virtual Properties Not Showing Up in Response

**Root Cause:** Mongoose virtuals are not included in `JSON.stringify()` or `toObject()` output by default.

**Solution:**
-   **Enable Virtuals in Schema Options**: You must explicitly tell the schema to include virtuals when converting a document to JSON.
    ```javascript
    // In order.js
    const orderSchema = new Schema({ /* ... */ }, {
      timestamps: true,
      toJSON: { virtuals: true } // This line is required!
    });
    ```

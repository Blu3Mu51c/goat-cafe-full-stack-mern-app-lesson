# 01. Project Setup and Configuration

## Overview
This file covers the initial setup of your full-stack e-commerce application using Vite + React for the frontend and Express + MongoDB for the backend.

## Table of Contents
1. [Initial Project Creation](#initial-project-creation)
2. [Dependencies Installation](#dependencies-installation)
3. [Project Structure](#project-structure)
4. [Environment Configuration](#environment-configuration)
5. [Package.json Scripts](#packagejson-scripts)

---

## Initial Project Creation

### Create Vite Project
```bash
# Create a new Vite project with React template
npm create vite@latest goat-cafe-vite -- --template react

# Navigate to project directory
cd goat-cafe-vite

# Install base dependencies
npm install
```

### Why Vite?
- **Lightning-fast development server** with Hot Module Replacement (HMR)
- **Modern ES modules** support out of the box
- **Optimized build process** using Rollup
- **Plugin system** for extensibility
- **CSS preprocessing** support including SCSS

---

## Dependencies Installation

### Backend Dependencies
```bash
# Core backend & react router packages
npm install express cors mongoose bcrypt jsonwebtoken dotenv react-router-dom

# Development dependencies
npm install --save-dev nodemon concurrently sass
```

### Package Breakdown
- **express**: Web framework for Node.js
- **cors**: Cross-Origin Resource Sharing middleware
- **mongoose**: MongoDB object modeling tool
- **bcrypt**: Password hashing library
- **jsonwebtoken**: JWT authentication
- **dotenv**: Environment variable management
- **nodemon**: Auto-restart server during development
- **concurrently**: Run multiple npm scripts simultaneously
- **sass**: CSS preprocessor

---

## Project Structure

### Directory Layout
```
goat-cafe-vite/
├── src/                    # Frontend source code
│   ├── components/         # Reusable UI components
│   ├── pages/             # Page-level components
│   ├── utilities/          # API calls and business logic
│   ├── main.jsx           # React entry point
│   └── index.scss         # Global styles
├── public/                 # Static assets (development)
├── dist/                   # Built files (production)
├── routes/                 # Express API routes
├── controllers/            # Business logic handlers
├── models/                 # Database models
├── config/                 # Configuration files
├── server.js              # Main server entry point
├── app-server.js          # Express app configuration
├── vite.config.js         # Vite configuration
├── package.json           # Dependencies and scripts
└── .env                   # Environment variables
```

### Understanding the Structure
- **Frontend (`src/`)**: React components and logic
- **Backend (`routes/`, `controllers/`, `models/`)**: API and database logic
- **Configuration (`config/`)**: Database and middleware setup
- **Entry Points**: `server.js` (backend) and `main.jsx` (frontend)

---

## Environment Configuration

### Create .env File
Create a `.env` file in your root directory:
```bash
# MongoDB Connection String
MONGO_URI=mongodb://127.0.0.1:27017/goat-cafe

# JWT Secret Key (use a strong, unique secret)
SECRET=your-super-secret-jwt-key-here

# Server Port
PORT=3000
```

### Environment Variables Explained
- **MONGO_URI**: Connection string to your MongoDB database
- **SECRET**: Secret key for JWT token signing/verification
- **PORT**: Port number for the backend server

### Security Notes
- **Never commit `.env` to version control**
- **Use strong, unique secrets** for production
- **Rotate secrets regularly** for security
- **Use different values** for development and production

---

## Package.json Scripts

### Script Configuration
```json
{
  "scripts": {
    "dev": "vite",
    "server": "nodemon server.js",
    "dev:full": "concurrently \"npm run server\" \"npm run dev\"",
    "build": "vite build",
    "start": "NODE_ENV=production node server.js",
    "start:dev": "NODE_ENV=development node server.js",
    "lint": "eslint .",
    "preview": "vite preview"
  }
}
```

### Script Purposes
- **`dev`**: Start Vite development server (frontend only)
- **`server`**: Start Express backend server with auto-restart
- **`dev:full`**: Start both frontend and backend simultaneously
- **`build`**: Build frontend for production
- **`start`**: Start production server
- **`start:dev`**: Start development server
- **`lint`**: Run ESLint for code quality
- **`preview`**: Preview production build locally

---

## Next Steps

After completing this setup:

1. **Verify Installation**: Run `npm run dev` to test frontend
2. **Test Backend**: Run `npm run server` to test backend
3. **Check Dependencies**: Ensure all packages are installed correctly
4. **Environment Variables**: Verify `.env` file is properly configured
5. **Move to Next File**: Continue with [02_BACKEND_FOUNDATION.md](./02_BACKEND_FOUNDATION.md)

## Common Setup Issues

### Issue: Port Already in Use
```bash
# Find process using port 3000
lsof -i :3000

# Kill the process
kill -9 <PID>

# Or use different port
PORT=8001 npm run server
```

### Issue: Dependencies Not Found
```bash
# Clear npm cache
npm cache clean --force

# Remove node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Issue: Environment Variables Not Loading
- Ensure `.env` file is in root directory
- Check file permissions
- Verify no spaces around `=` in `.env` file
- Restart server after `.env` changes

---

## Verification Checklist

- [ ] Vite project created successfully
- [ ] All dependencies installed
- [ ] `.env` file created with correct values
- [ ] Package.json scripts configured
- [ ] Project structure created
- [ ] Frontend starts without errors (`npm run dev`)
- [ ] Backend starts without errors (`npm run server`)

Once all items are checked, you're ready to proceed to the next file!

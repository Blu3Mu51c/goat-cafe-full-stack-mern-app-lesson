# 06. Frontend Setup with Vite and React

## Overview
This file covers setting up the React frontend with Vite, configuring SCSS support, and setting up the basic application structure.

## Table of Contents
1. [Vite Configuration](#vite-configuration)
2. [React Application Structure](#react-application-structure)
3. [SCSS Integration](#scss-integration)
4. [Component Architecture](#component-architecture)
5. [State Management Setup](#state-management-setup)
6. [Testing Frontend](#testing-frontend)

---

## Vite Configuration

### Update vite.config.js
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

### Configuration Explained

#### 1. React Plugin
```javascript
plugins: [react()]
```
**Purpose**: Enables React support with Fast Refresh and JSX transformation

#### 2. SCSS Support
```javascript
css: {
  preprocessorOptions: {
    scss: {
      // SCSS configuration
    }
  }
}
```
**Purpose**: Enables SCSS compilation and preprocessing

#### 3. Development Server
```javascript
server: {
  port: 5173,
  proxy: {
    '/api': {
      target: 'http://localhost:3000',
      changeOrigin: true
    }
  }
}
```

**Port**: Frontend runs on port 5173
**Proxy**: Redirects `/api/*` calls to backend on port 3000
**Change Origin**: Handles CORS for development

---

## React Application Structure

### Directory Layout
```
src/
├── components/           # Reusable UI components
│   ├── Logo/
│   │   ├── Logo.jsx
│   │   └── Logo.module.scss
│   ├── LoginForm/
│   │   ├── LoginForm.jsx
│   │   └── LoginForm.module.scss
│   └── Navigation/
│       ├── Navigation.jsx
│       └── Navigation.module.scss
├── pages/               # Page-level components
│   ├── HomePage/
│   │   ├── HomePage.jsx
│   │   └── HomePage.module.scss
│   ├── NewOrderPage/
│   │   ├── NewOrderPage.jsx
│   │   └── NewOrderPage.module.scss
│   └── OrderHistoryPage/
│       ├── OrderHistoryPage.jsx
│       └── OrderHistoryPage.module.scss
├── utilities/           # Business logic and API calls
│   ├── users-service.js
│   ├── users-api.js
│   ├── items-api.js
│   └── orders-api.js
├── contexts/            # React Context providers
│   └── AuthContext.jsx
├── main.jsx            # Application entry point
├── App.jsx             # Main application component
├── index.scss          # Global styles
└── App.scss            # App-specific styles
```

### Understanding the Structure

#### 1. Components Directory
**Purpose**: Reusable UI pieces that can be used across multiple pages
**Examples**: Buttons, forms, navigation bars, modals

#### 2. Pages Directory
**Purpose**: Full page layouts that compose multiple components
**Examples**: Home page, order creation page, order history page

#### 3. Utilities Directory
**Purpose**: Business logic, API calls, and helper functions
**Examples**: Authentication services, data fetching, form validation

#### 4. Contexts Directory
**Purpose**: Global state management using React Context
**Examples**: User authentication state, shopping cart state

---

## SCSS Integration

### Install SCSS Support
```bash
npm install sass
```

### Create Global SCSS File
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

### Import Global SCSS
Update `src/main.jsx`:
```javascript
import {StrictMode} from "react";
import { createRoot } from "react-dom/client";
import AppRouter from './router/index.jsx';
import './index.scss';
const root = createRoot(document.getElementById("root"))
root.render(<StrictMode><AppRouter/></StrictMode>)
```

---

## Component Architecture

### App Router Setup
Create `src/router/routes.js`:
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

Create `src/router/index.jsx`:
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

---

## Testing Frontend

### Start Development Server
```bash
# Terminal 1: Start backend
npm run server

# Terminal 2: Start frontend
npm run dev

# Or run both simultaneously
npm run dev:full
```

### Test Frontend
1. **Open Browser**: Navigate to http://localhost:5173
2. **Check Styling**: Verify SCSS is compiling correctly
3. **Test Components**: Ensure LoginForm displays properly
4. **Check Console**: Look for any JavaScript errors
5. **Test Responsiveness**: Resize browser window

### Expected Results
- ✅ Frontend loads without errors
- ✅ SCSS styles are applied correctly
- ✅ Login form displays with proper styling
- ✅ No console errors
- ✅ Components render correctly

---

## Common Issues and Solutions

### Issue: SCSS Not Compiling
**Problem**: Styles not applying or SCSS errors

**Solutions**:
1. Verify `sass` package is installed
2. Check import statements in components
3. Ensure file extensions are `.scss`
4. Restart development server

### Issue: Proxy Not Working
**Problem**: API calls going to wrong port

**Solutions**:
1. Check Vite proxy configuration
2. Ensure backend is running on port 3000
3. Verify API calls use relative paths (`/api/users`)
4. Check CORS configuration

### Issue: Components Not Rendering
**Problem**: Blank page or component errors

**Solutions**:
1. Check import/export statements
2. Verify component file structure
3. Check for JavaScript syntax errors
4. Ensure React components are properly exported

---

## Next Steps

After completing this setup:

1. **Verify Frontend**: Ensure all components render correctly
2. **Test Styling**: Verify SCSS compilation works
3. **Check Proxy**: Ensure API calls work correctly
4. **Move to Next File**: Continue with [07_UTILITIES_AND_SERVICES.md](./07_UTILITIES_AND_SERVICES.md)

## Verification Checklist

- [ ] Vite configuration updated with proxy and SCSS
- [ ] React application structure created
- [ ] Global SCSS file created and imported
- [ ] Component architecture implemented
- [ ] AuthContext created for state management
- [ ] Basic components (Logo, LoginForm) created
- [ ] SCSS modules working correctly
- [ ] Frontend development server starts
- [ ] No console errors
- [ ] Components render with proper styling

Once all items are checked, you're ready to proceed to the next file!

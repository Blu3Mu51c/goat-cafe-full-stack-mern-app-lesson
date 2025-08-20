# 08. Component State Logic and React Hooks

## Overview
This file covers implementing React components with proper state management, using hooks like `useState`, `useEffect`, and `useRef`, and connecting components to the service layer.

## Table of Contents
1. [State Management in `AuthPage`](#authpage-state)
2. [State in `LoginForm` & `SignUpForm`](#form-state)
3. [Complex State in `NewOrderPage`](#neworderpage-state)
4. [State Management in `OrderHistoryPage`](#orderhistorypage-state)
5. [Testing Components](#testing-components)

---

## 1. State Management in `AuthPage`

### `pages/AuthPage/AuthPage.jsx`
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

### State Logic Explained
- **`useState(true)`**: This is the core of the component's logic. A single boolean state variable, `showLogin`, determines which of the two forms (`LoginForm` or `SignUpForm`) is rendered.
- **Conditional Rendering**: A ternary operator (`showLogin ? <LoginForm ... /> : <SignUpForm ... />`) uses this state to conditionally render the correct component.
- **State Toggling**: The `onClick` handler on the `<h3>` element simply toggles the boolean value of `showLogin`, causing the component to re-render and display the other form.
- **Props Drilling**: The `setUser` function is passed down (`drilled`) to both `LoginForm` and `SignUpForm` so they can update the application's global user state upon successful authentication.

---

## 2. State in `LoginForm` & `SignUpForm`

### `components/LoginForm/LoginForm.jsx` (Functional Component)
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
        evt.preventDefault();
        try {
            const user = await usersService.login(credentials);
            setUser(user);
        } catch {
            setError('Log In Failed - Try Again');
        }
    }
    // ... render logic
}
```

### `components/SignUpForm/SignUpForm.jsx` (Class Component)
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
            const user = await signUp(formData);
            this.props.setUser(user);
        } catch {
            this.setState({ error: 'Sign Up Failed - Try Again' });
        }
    };
    // ... render logic
}
```

### State Logic Explained
Both forms manage local state for their input fields and any potential error messages.
- **`useState` (in LoginForm)**: The functional component uses the `useState` hook to manage `credentials` (an object) and `error` (a string).
- **`this.state` (in SignUpForm)**: The class component uses the traditional `this.state` object to hold all state properties.
- **Controlled Components**: In both cases, the input fields' values are tied to the component's state, and the `onChange` (or `handleChange`) handler updates the state. This makes them "controlled components."
- **Error State**: The `error` state is used to display feedback to the user if an API call to the backend fails. It's cleared whenever the user types again, providing a good user experience.

---

## 3. Complex State in `NewOrderPage`

### `pages/NewOrderPage/NewOrderPage.jsx`
```javascript
import { useState, useEffect, useRef } from 'react';
// ... other imports

export default function NewOrderPage({ user, setUser }) {
  const [menuItems, setMenuItems] = useState([]);
  const [activeCat, setActiveCat] = useState('');
  const [cart, setCart] = useState(null);
  const categoriesRef = useRef([]);
  const navigate = useNavigate();

  useEffect(function() {
    async function getItems() {
      const items = await itemsAPI.getAll();
      categoriesRef.current = //... categories logic ...
      setMenuItems(items);
      setActiveCat(categoriesRef.current[0]);
    }
    getItems();

    async function getCart() {
      const cart = await ordersAPI.getCart();
      setCart(cart);
    }
    getCart();
  }, []); // Empty dependency array means this runs once on mount

  // ... event handlers like handleAddToOrder, handleChangeQty, etc.
}
```

### State Logic Explained
This component orchestrates multiple pieces of state and side effects.
- **`useState`**:
    - `menuItems`: Holds the list of all available menu items fetched from the server.
    - `activeCat`: Stores the currently selected category name (a string), used for filtering `menuItems`.
    - `cart`: Holds the user's current order object, including line items.
- **`useRef`**:
    - `categoriesRef`: Holds an array of unique category names. `useRef` is used here because this data doesn't need to trigger a re-render when it changes. It's calculated once and its value persists across re-renders.
- **`useEffect`**:
    - This hook is used to perform "side effects"—in this case, fetching data from the API when the component first mounts.
    - The empty dependency array `[]` is crucial; it ensures the fetch operations run only once, preventing an infinite loop of API calls.
    - Inside, it calls `itemsAPI.getAll()` and `ordersAPI.getCart()` and then updates the component's state with the results, causing the component to re-render and display the fetched data.

---

## 4. State Management in `OrderHistoryPage`

### `pages/OrderHistoryPage/OrderHistoryPage.jsx`
```javascript
import { useState, useEffect } from 'react';
// ... other imports

export default function OrderHistoryPage({ user, setUser }) {
  const [orders, setOrders] = useState([]);
  const [activeOrder, setActiveOrder] = useState(null);

  useEffect(function () {
    async function fetchOrderHistory() {
      const orders = await ordersAPI.getOrderHistory();
      setOrders(orders);
      setActiveOrder(orders[0] || null);
    }
    fetchOrderHistory();
  }, []);

  function handleSelectOrder(order) {
    setActiveOrder(order);
  }
  // ... render logic
}
```

### State Logic Explained
- **`useState`**:
    - `orders`: An array that holds the list of the user's past paid orders.
    - `activeOrder`: Holds the currently selected order object from the `orders` list. This determines which order details are displayed.
- **`useEffect`**:
    - Similar to `NewOrderPage`, this hook fetches the user's order history when the component mounts.
    - After fetching, it updates the `orders` state and sets the `activeOrder` to be the most recent one in the list (or `null` if there are no orders).
- **Event Handling**:
    - `handleSelectOrder` is a simple state setter. When a user clicks on an order in the `OrderList` component, this function updates the `activeOrder` state, which causes `OrderDetail` to re-render with the newly selected order's information.

---

## 5. Next Steps

After completing this setup:
- **Test Components**: Verify all components work correctly
- **Move to Next File**: Continue with [09_END_TO_END_FLOW.md](./09_END_TO_END_FLOW.md)

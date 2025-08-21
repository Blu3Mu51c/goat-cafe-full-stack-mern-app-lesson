# MERN Stack Jeopardy Quiz

This quiz is designed to test your knowledge of the full-stack MERN application, based on the concepts used in the Goat Cafe Vite project.

---

##  jeopardy Questions

| CSS & SCSS | JSX Rules | React | Express Restful Routing | Mongoose |
| :--- | :--- | :--- | :--- | :--- |
| **100** - What is the file extension used to enable CSS Modules with SCSS in our project? | **100** - Name the `useEffect` hook's second argument, which controls when the effect is re-run. | **100** - What is the name of the React Hook used to add state to a functional component? | **100** - What are the two parts of the controller pattern used in our `users` controller? | **100** - What is the name for a Mongoose schema property that is calculated on the fly and not saved to the database? |
| **200** - Explain the primary benefit of using CSS Modules over global CSS stylesheets. | **200** - Why can't you return multiple adjacent elements from a JSX component without a wrapper? | **200** - In a React form, what is the key difference between a "controlled" and an "uncontrolled" input? | **200** - In our project, what is the purpose of the `ensureLoggedIn` middleware? | **200** - What is the purpose of the `ref` property in a Mongoose schema field, such as `ref: 'User'`? |
| **300** - In an SCSS file, how would you use a global CSS custom property named `--primary-color` and an SCSS variable named `$secondary-color`? | **300** - A user logs in, but a component doesn't update to show their name. The `user` state exists in the parent `AppRouter` but not in the child component. What is the most likely cause of this issue? | **300** - In a controlled form component, which React hook manages the input's value, and which HTML attribute on the `<input>` element is used to update it? | **300** - A `GET` request to a protected API route fails with a `401 Unauthorized` error, even with a valid token. What middleware is likely missing from the route's definition in `app-server.js`? | **300** - What option must be added to a Mongoose schema to ensure its virtual properties are included in JSON responses? |
| **400** - Write the SCSS code for a class `.card` that sets a `background-color` and `padding`, and then adds a `box-shadow` that only appears on hover. | **400** - Write a small JSX component that takes an array of strings called `items` as a prop and renders them as an unordered list. Each list item must have a unique `key`. | **400** - Write the complete code for a functional React component that fetches data from `'/api/items'` on mount and displays "Loading..." until the data arrives. | **400** - Write a complete Express route that will handle a `PUT` request to update an item's quantity in the cart at the URL `/api/orders/cart/qty`. | **400** - Write a Mongoose static method named `findByName` that can be called on the `Item` model to find all items that have a specific name. |

---

##  jeopardy Answers

| CSS & SCSS | JSX Rules | React | Express Restful Routing | Mongoose |
| :--- | :--- | :--- | :--- | :--- |
| **100** - `.module.scss` | **100** - The Dependency Array. | **100** - `useState` | **100** - The `dataController` and the `apiController`. | **100** - A virtual property (or "virtual"). |
| **200** - CSS Modules automatically scopes class names to a specific component, preventing style collisions and conflicts in a large application. | **200** - A component's `return` can only return a single value. Multiple JSX elements must be wrapped in a single parent element or a React Fragment (`<>...</>`). | **200** - A controlled input's value is managed by React state (`useState`), while an uncontrolled input's value is handled by the DOM itself (often accessed via a `useRef`). | **200** - To protect a route by checking if `req.user` exists, ensuring that only authenticated users can access it. | **200** - It tells Mongoose which model to look at when using `.populate()` to replace an `ObjectId` with the full referenced document. |
| **300** - `color: var(--primary-color);` and `background-color: $secondary-color;` | **300** - The `user` state object was not passed down as a prop from the `AppRouter` component to the child component. | **300** - The `useState` hook manages the value, and the `onChange` attribute is used to call the state setter function. | **300** - The `checkToken` middleware, which is responsible for decoding the token and attaching `req.user` to the request object. | **300** - You must add `{ toJSON: { virtuals: true } }` to the schema options. |
| **400** - ```scss .card { background-color: white; padding: 16px; &:hover { box-shadow: 0 4px 8px rgba(0,0,0,0.1); } } ```| **400** - ```jsx function ItemList({ items }) { return ( <ul> {items.map((item, index) => ( <li key={index}>{item}</li> ))} </ul> ); } ``` | **400** - ```jsx import { useState, useEffect } from 'react'; function ItemList() { const [items, setItems] = useState(null); useEffect(() => { async function getItems() { const res = await fetch('/api/items'); const data = await res.json(); setItems(data); } getItems(); }, []); return items ? ( <ul> {items.map(item => <li key={item._id}>{item.name}</li>)} </ul> ) : <p>Loading...</p>; } ``` | **400** - ```javascript // In routes/api/orders.js import { setItemQtyInCart } from '../../controllers/api/orders.js'; router.put('/cart/qty', setItemQtyInCart); ``` | **400** - ```javascript // In models/itemSchema.js itemSchema.statics.findByName = function(name) { // 'this' refers to the Item model return this.find({ name: name }); }; ``` |

---

## 🎲 Full Jeopardy Answer List with Explanations

---

### 1.
*   **Category**: CSS & SCSS
*   **Points**: 100
*   **Question**: What is the file extension used to enable CSS Modules with SCSS in our project?
*   **Answer**: `.module.scss`
*   **Explanation**: Vite is configured to treat files with the `.module.scss` extension as CSS Modules, which automatically generates a unique class name for each style to prevent global conflicts.
*   **Acceptable Alternatives**: ".module.scss file"

---

### 2.
*   **Category**: JSX Rules
*   **Points**: 100
*   **Question**: Name the `useEffect` hook's second argument, which controls when the effect is re-run.
*   **Answer**: The Dependency Array.
*   **Explanation**: The dependency array tells React to only re-run the effect if one of the values in the array has changed since the last render. An empty array `[]` means it only runs once when the component mounts.
*   **Acceptable Alternatives**: "dependencies", "the array"

---

### 3.
*   **Category**: React
*   **Points**: 100
*   **Question**: What is the name of the React Hook used to add state to a functional component?
*   **Answer**: `useState`
*   **Explanation**: `useState` is the fundamental hook for declaring and managing state variables within functional components. It returns a state variable and a function to update it.
*   **Acceptable Alternatives**: None.

---

### 4.
*   **Category**: Express Restful Routing
*   **Points**: 100
*   **Question**: What are the two parts of the controller pattern used in our `users` controller?
*   **Answer**: The `dataController` and the `apiController`.
*   **Explanation**: This pattern separates the database logic (`dataController`) from the response logic (`apiController`), making the code more organized and adhering to the separation of concerns principle.
*   **Acceptable Alternatives**: "Data and API controllers"

---

### 5.
*   **Category**: Mongoose
*   **Points**: 100
*   **Question**: What is the name for a Mongoose schema property that is calculated on the fly and not saved to the database?
*   **Answer**: A virtual property (or "virtual").
*   **Explanation**: Virtuals are ideal for computed properties like the `orderTotal` in our Order model, which can be derived from other fields (`lineItems`) without taking up database space.
*   **Acceptable Alternatives**: None.

---

### 6.
*   **Category**: CSS & SCSS
*   **Points**: 200
*   **Question**: Explain the primary benefit of using CSS Modules over global CSS stylesheets.
*   **Answer**: CSS Modules automatically scopes class names to a specific component, preventing style collisions and conflicts.
*   **Explanation**: This means you can have a `.container` class in `LoginForm.module.scss` and a different `.container` class in `MenuList.module.scss` without them interfering with each other.
*   **Acceptable Alternatives**: "It prevents class name conflicts," "It makes styles local to a component."

---

### 7.
*   **Category**: JSX Rules
*   **Points**: 200
*   **Question**: Why can't you return multiple adjacent elements from a JSX component without a wrapper?
*   **Answer**: A component's `return` can only return a single value. Multiple JSX elements must be wrapped in a single parent element or a React Fragment (`<>...</>`).
*   **Explanation**: Under the hood, a JSX expression compiles to a single `React.createElement()` function call, and a function can only return one thing.
*   **Acceptable Alternatives**: "You can only return one root element," "JSX needs a single parent."

---

### 8.
*   **Category**: React
*   **Points**: 200
*   **Question**: In a React form, what is the key difference between a "controlled" and an "uncontrolled" input?
*   **Answer**: A controlled input's value is managed by React state (`useState`), while an uncontrolled input's value is handled by the DOM itself (often accessed via a `useRef`).
*   **Explanation**: In our project, all forms use controlled inputs, where every keystroke calls a `handleChange` function to update state, making React the "single source of truth" for the input's value.
*   **Acceptable Alternatives**: "Controlled uses state, Uncontrolled uses refs," "State controls the value vs. the DOM controls the value."

---

### 9.
*   **Category**: Express Restful Routing
*   **Points**: 200
*   **Question**: In our project, what is the purpose of the `ensureLoggedIn` middleware?
*   **Answer**: To protect a route by checking if `req.user` exists, ensuring that only authenticated users can access it.
*   **Explanation**: It acts as a gatekeeper. If `req.user` (which should have been set by the `checkToken` middleware) is not present, it sends a `401 Unauthorized` response and stops the request from proceeding.
*   **Acceptable Alternatives**: "To make a route private," "To check if a user is logged in."

---

### 10.
*   **Category**: Mongoose
*   **Points**: 200
*   **Question**: What is the purpose of the `ref` property in a Mongoose schema field, such as `ref: 'User'`?
*   **Answer**: It tells Mongoose which model to look at when using `.populate()` to replace an `ObjectId` with the full referenced document.
*   **Explanation**: This is the key to creating relationships between collections in MongoDB. The `ref` must match the string name used in `mongoose.model('ModelName', schema)`.
*   **Acceptable Alternatives**: "It links to another model," "It's for populating data."

---

### 11.
*   **Category**: CSS & SCSS
*   **Points**: 300
*   **Question**: In an SCSS file, how would you use a global CSS custom property named `--primary-color` and an SCSS variable named `$secondary-color`?
*   **Answer**: `color: var(--primary-color);` and `background-color: $secondary-color;`
*   **Explanation**: CSS custom properties (variables) are accessed using the `var()` function and are available globally in the browser. SCSS variables start with a `$` and are compiled away into static CSS values before the code ever reaches the browser.
*   **Acceptable Alternatives**: Any correct CSS properties using `var()` for the CSS variable and the `$` for the SCSS variable.

---

### 12.
*   **Category**: JSX Rules
*   **Points**: 300
*   **Question**: A user logs in, but a component doesn't update to show their name. The `user` state exists in the parent `AppRouter` but not in the child component. What is the most likely cause of this issue?
*   **Answer**: The `user` state object was not passed down as a prop from the `AppRouter` component to the child component.
*   **Explanation**: This is a classic "prop drilling" issue. For a child component to access state from a parent, the parent must explicitly pass that state down as a prop.
*   **Acceptable Alternatives**: "You forgot to pass the user prop."

---

### 13.
*   **Category**: React
*   **Points**: 300
*   **Question**: In a controlled form component, which React hook manages the input's value, and which HTML attribute on the `<input>` element is used to update it?
*   **Answer**: The `useState` hook manages the value, and the `onChange` attribute is used to call the state setter function.
*   **Explanation**: The input's `value` is set from the state variable, and its `onChange` handler calls the setter function (e.g., `setCredentials`) to keep the state synchronized with what the user types.
*   **Acceptable Alternatives**: "useState and onChange"

---

### 14.
*   **Category**: Express Restful Routing
*   **Points**: 300
*   **Question**: A `GET` request to a protected API route fails with a `401 Unauthorized` error, even with a valid token. What middleware is likely missing from the route's definition in `app-server.js`?
*   **Answer**: The `checkToken` middleware, which is responsible for decoding the token and attaching `req.user` to the request object.
*   **Explanation**: The `ensureLoggedIn` middleware only checks for the existence of `req.user`. If `checkToken` doesn't run first to create `req.user` from the token, then `ensureLoggedIn` will always fail.
*   **Acceptable Alternatives**: "The middleware order is wrong," "checkToken."

---

### 15.
*   **Category**: Mongoose
*   **Points**: 300
*   **Question**: What option must be added to a Mongoose schema to ensure its virtual properties are included in JSON responses?
*   **Answer**: You must add `{ toJSON: { virtuals: true } }` to the schema options.
*   **Explanation**: Mongoose requires you to explicitly opt-in to including virtuals when a document is converted to JSON. Without this option, virtual properties are ignored during `res.json()`.
*   **Acceptable Alternatives**: "Enable virtuals in toJSON."

---

### 16.
*   **Category**: CSS & SCSS
*   **Points**: 400
*   **Question**: Write the SCSS code for a class `.card` that sets a `background-color` and `padding`, and then adds a `box-shadow` that only appears on hover.
*   **Answer**: ```scss .card { background-color: white; padding: 16px; &:hover { box-shadow: 0 4px 8px rgba(0,0,0,0.1); } } ```
*   **Explanation**: This demonstrates basic SCSS nesting using the `&` parent selector to apply styles specifically to the `:hover` pseudo-class of the `.card` element.
*   **Acceptable Alternatives**: Any syntactically correct SCSS that achieves the goal.

---

### 17.
*   **Category**: JSX Rules
*   **Points**: 400
*   **Question**: Write a small JSX component that takes an array of strings called `items` as a prop and renders them as an unordered list. Each list item must have a unique `key`.
*   **Answer**: ```jsx function ItemList({ items }) { return ( <ul> {items.map((item, index) => ( <li key={index}>{item}</li> ))} </ul> ); } ```
*   **Explanation**: This shows the standard `.map()` pattern for rendering lists. The `key` prop is crucial for React to efficiently update the list. Using the array `index` is acceptable for static lists but can cause issues if the list is sorted or filtered. A unique ID from the data (`item.id`) is preferred when available.
*   **Acceptable Alternatives**: Arrow function syntax, using a unique ID for the key if provided.

---

### 18.
*   **Category**: React
*   **Points**: 400
*   **Question**: Write the complete code for a functional React component that fetches data from `'/api/items'` on mount and displays "Loading..." until the data arrives.
*   **Answer**: ```jsx import { useState, useEffect } from 'react'; function ItemList() { const [items, setItems] = useState(null); useEffect(() => { async function getItems() { const res = await fetch('/api/items'); const data = await res.json(); setItems(data); } getItems(); }, []); return items ? ( <ul> {items.map(item => <li key={item._id}>{item.name}</li>)} </ul> ) : <p>Loading...</p>; } ```
*   **Explanation**: This demonstrates `useState` (initialized to `null` to track the initial loading state), `useEffect` to fetch data on mount, and a ternary operator for conditional rendering.
*   **Acceptable Alternatives**: Using `.then()` syntax, initializing state to an empty array `[]` and checking `items.length`.

---

### 19.
*   **Category**: Express Restful Routing
*   **Points**: 400
*   **Question**: Write a complete Express route that will handle a `PUT` request to update an item's quantity in the cart at the URL `/api/orders/cart/qty`.
*   **Answer**: ```javascript // In routes/api/orders.js import { setItemQtyInCart } from '../../controllers/api/orders.js'; router.put('/cart/qty', setItemQtyInCart); ```
*   **Explanation**: This shows the standard way to define a route. It specifies the HTTP method (`.put`), the URL path (`'/cart/qty'`), and the imported controller function (`setItemQtyInCart`) that will handle the request's logic.
*   **Acceptable Alternatives**: None.

---

### 20.
*   **Category**: Mongoose
*   **Points**: 400
*   **Question**: Write a Mongoose static method named `findByName` that can be called on the `Item` model to find all items that have a specific name.
*   **Answer**: ```javascript // In models/itemSchema.js itemSchema.statics.findByName = function(name) { // 'this' refers to the Item model return this.find({ name: name }); }; ```
*   **Explanation**: Static methods are added to the `statics` object of a schema. Inside a static method, `this` refers to the Model itself, allowing you to call top-level Mongoose query methods like `.find()`.
*   **Acceptable Alternatives**: `return this.find({ name })` (using ES6 shorthand).

---

## 💥 Final Jeopardy

*   **Category**: Full-Stack Implementation Challenge
*   **Question**:
    Your task is to create a brand new, dynamic page in the Goat Cafe application. You must use the existing Mongoose models (`User`, `Order`, `Item`, `Category`) without changing them, but you can add any new controllers, routes, utilities, components, and pages needed to support your new feature.

    **Ideas for Your Page:**
    -   A **"Menu Insights"** page that shows statistics, like the most popular items (based on order history) or the number of items per category.
    -   A **"User Profile"** page where a logged-in user can see a summary of their own order history and stats.
    -   A **"Daily Special"** page that randomly features one item from the menu each day.

    **What you need to do:**
    1.  **Backend**: Create a new route and a controller function to fetch the specific data your page needs from the database.
    2.  **Frontend**: Create new API utility functions, a new page component, and any necessary child components to display the data you fetched.
    3.  **Routing**: Add your new page to the React Router.
*   **Answer**: (Student presents their solution)
*   **Explanation**:
    A correct solution must demonstrate a full end-to-end data flow. The student needs to successfully:
    1.  Create a new backend endpoint (route + controller) that queries the database using existing Mongoose models.
    2.  Create a new frontend API utility function to call that endpoint.
    3.  Create a new React page component that uses `useEffect` to call the new utility function on mount.
    4.  Use `useState` to store the fetched data.
    5.  Render the data in a meaningful way using JSX.
    6.  Add the new page's route to the `router/routes.js` file.
*   **Acceptable Alternatives**: Any implementation that successfully creates a new, non-static page that correctly fetches and displays data derived from the existing models will be considered a success.

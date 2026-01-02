
# React Router DOM – Revision Notes (v6+)

## 1. What is React Router DOM?
React Router DOM is a client-side routing library for React that enables navigation between different UI views without reloading the page.
* **URL changes** update components
* **No full page refresh**
* **Routing handled** in the browser

---

## 2. Core Mental Model
1.  **Router** (The Configuration)
2.  **RouterProvider** (The Context Provider)
3.  **Routes + Components** (The View)

* **Router** defines *WHAT* routes exist.
* **RouterProvider** provides routing capability to the app.
* **Routes** decide *WHICH* component renders based on the URL.

---

## 3. createBrowserRouter
### Purpose
Creates a router configuration using the browser history API. This is the recommended router for all React Router web projects.

### Key Point
`createBrowserRouter` **defines routes**, it does not render anything.

### Example
```jsx
const router = createBrowserRouter([
  { 
    path: "/", 
    element: <Home /> 
  }
]);

```

---

## 4. RouterProvider

### Purpose

Provides the router configuration to the React component tree.

### Why it is required

Without `RouterProvider`, the following hooks and components will **NOT** work:

* `<Route />`
* `<Outlet />`
* `useParams`
* `useNavigate`

### Example

```jsx
<RouterProvider router={router} />

```

> **Remember:** `RouterProvider` does NOT define routes. It enables routing by supplying the configuration to the app.

---

## 5. Route Component

### Purpose

Maps a URL path to a React component.

### Example

```jsx
<Route path="/about" element={<About />} />

```

* **path**: The URL segment.
* **element**: The Component to render.

---

## 6. Nested Routes

### Why Nested Routes?

To reuse layouts across multiple pages (e.g., Navbar, Footer, Sidebar).

### Example

```jsx
<Route path="/" element={<App />}>
  <Route index element={<Home />} />
  <Route path="about" element={<About />} />
</Route>

```

---

## 7. Outlet

### Purpose

A placeholder component that renders the matched child route.

### Example

```jsx
function App() {
  return (
    <>
      <Navbar />
      <Outlet /> {/* Child routes render here */}
      <Footer />
    </>
  );
}

```

---

## 8. Index Routes

### What is an Index Route?

The default child route that renders when the parent's path is matched exactly.

### Example

```jsx
<Route path="/" element={<App />}>
  <Route index element={<Home />} /> {/* Renders at "/" */}
</Route>

```

---

## 9. Dynamic Routes (URL Params)

### Purpose

To pass dynamic values (like IDs) through the URL.

### Example

```jsx
<Route path="doctors/:speciality" element={<Doctors />} />

```

---

## 10. useParams

### Purpose

A hook used to read the dynamic values from the URL segments.

### Example

```jsx
const { speciality } = useParams();

```

---

## 11. useNavigate

### Purpose

Allows for **programmatic navigation** (navigating via code rather than a user click).

### Example

```jsx
const navigate = useNavigate();

const handleLogin = () => {
  // Logic here
  navigate("/dashboard");
};

```

---

## 12. Link vs NavLink

| Feature | Link | NavLink |
| --- | --- | --- |
| **Purpose** | Simple navigation | Navigation with "active" styling |
| **Usage** | Standard internal links | Navbar/Sidebar links |
| **Active State** | No | Yes (adds `.active` class) |

---

## 13. Error Handling

### Purpose

Handles 404s or rendering errors gracefully using the `errorElement` property.

### Example

```jsx
const router = createBrowserRouter([
  {
    path: "/",
    element: <App />,
    errorElement: <ErrorPage />
  }
]);

```

---

## 14. Common Mistakes to Avoid

1. **Forgetting `<Outlet />**`: Child routes won't appear.
2. **Using `path=""**`: Use the `index` prop instead for default children.
3. **Missing RouterProvider**: The app will crash on any router hook.
4. **Mixing Routers**: Don't use `BrowserRouter` and `createBrowserRouter` together.

---

## 15. Interview Summary (Cheat Sheet)

In **React Router v6+**, `createBrowserRouter` defines the config, while `RouterProvider` injects it into the app. We use **Nested Routes** and the `<Outlet />` to build persistent layouts. **Dynamic Routing** is handled via `:params` and accessed with `useParams`. For navigation, use `<Link>` for clicks and `useNavigate` for logic.
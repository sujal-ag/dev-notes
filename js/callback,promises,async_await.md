# 📌 Callbacks, Promises & Async/Await — JavaScript Revision Notes

## 1️⃣ Synchronous vs Asynchronous JavaScript

### 🔹 Synchronous Code

* Executes **line by line**
* Blocking in nature

```js
console.log("A");
console.log("B");
console.log("C");
```

Output:

```
A
B
C
```

---

### 🔹 Asynchronous Code

* Non‑blocking
* Used for:

  * API calls
  * Timers
  * File / DB operations

```js
console.log("A");
setTimeout(() => console.log("B"), 1000);
console.log("C");
```

Output:

```
A
C
B
```

---

## 2️⃣ Callbacks

### 🔹 What is a Callback?

A function passed as an argument to another function and executed later.

```js
function fetchData(callback) {
  setTimeout(() => {
    callback("Data received");
  }, 1000);
}

fetchData((data) => {
  console.log(data);
});
```

---

## 3️⃣ Callback Hell (Problem)

### ❌ Issues with Callbacks

* Deep nesting
* Poor readability
* Difficult error handling

```js
setTimeout(() => {
  console.log("Task 1");
  setTimeout(() => {
    console.log("Task 2");
    setTimeout(() => {
      console.log("Task 3");
    }, 1000);
  }, 1000);
}, 1000);
```

➡️ This structure is called **Callback Hell**.

---

## 4️⃣ Promises

### 🔹 What is a Promise?

An object that represents the **eventual result** of an asynchronous operation.

### 🔹 Promise States

| State     | Meaning              |
| --------- | -------------------- |
| Pending   | Initial state        |
| Fulfilled | Operation successful |
| Rejected  | Operation failed     |

---

### Creating a Promise

```js
const promise = new Promise((resolve, reject) => {
  let success = true;
  if (success)
    resolve("Operation successful");
  else
    reject("Operation failed");
});
```

---

### Consuming a Promise

```js
promise
  .then(result => {
    console.log(result);
  })
  .catch(error => {
    console.error(error);
  });
```

---

## 5️⃣ Promise Chaining

Used when multiple async operations depend on each other.

```js
fetch("/api/user")
  .then(res => res.json())
  .then(user => fetch(`/api/orders/${user.id}`))
  .then(res => res.json())
  .then(orders => console.log(orders))
  .catch(err => console.error(err));
```

---

## 6️⃣ Problems with Promises

* Long `.then()` chains reduce readability
* Error handling becomes scattered

---

## 7️⃣ Async / Await (Modern Approach)

### 🔹 What is `async`?

* Makes a function return a Promise

### 🔹 What is `await`?

* Pauses execution until the Promise resolves

---

### Example: Async/Await

```js
async function getData() {
  try {
    const res = await fetch("/api/data");
    const data = await res.json();
    console.log(data);
  } catch (error) {
    console.error(error);
  }
}
```

---

## 8️⃣ Why Async/Await is Better

| Feature        | Callbacks | Promises | Async/Await |
| -------------- | --------- | -------- | ----------- |
| Readability    | ❌         | 🙂       | ✅           |
| Error handling | ❌         | 🙂       | ✅           |
| Debugging      | ❌         | 🙂       | ✅           |

---

## 9️⃣ Error Handling in Fetch (IMPORTANT)

⚠️ `fetch()` does **not** throw errors for HTTP failures.

```js
async function getUser() {
  const res = await fetch("/api/user");
  if (!res.ok)
    throw new Error("HTTP Error");

  return await res.json();
}
```

---

## 🔟 Parallel Async Calls (`Promise.all`)

```js
async function loadData() {
  const [posts, comments] = await Promise.all([
    fetch("/posts"),
    fetch("/comments")
  ]);

  console.log(await posts.json());
  console.log(await comments.json());
}
```

---

## 🧠 Interview One‑Liners

* Callbacks lead to callback hell.
* Promises improve structure but can still get messy.
* Async/Await provides clean, readable, and maintainable async code.

---

## ✅ Final Verdict

✔ Use **async/await + try/catch** by default
✔ Use `.then()` for very small chains
✔ Use `Promise.all()` for parallel operations

---

📌 **This approach is industry‑standard and interview‑preferred.**

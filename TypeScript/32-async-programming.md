# TypeScript Async Programming

---

## `Promise<T>` — Typed Promises

`T` = the resolved value type.

```ts
const fetchGreeting = (): Promise<string> => {
  return new Promise((resolve, reject) => {
    resolve("Hello!");          // resolves to string
    reject(new Error("Failed")); // rejected
  });
};

fetchGreeting()
  .then(msg => msg.toUpperCase()) // TS knows: string
  .catch((err: Error) => console.error(err.message));
```

| Return type | Meaning |
|---|---|
| `Promise<string>` | Resolves to string |
| `Promise<void>` | Resolves with no value |
| `Promise<never>` | Never resolves (always throws/times out) |

---

## `async/await`

```ts
interface User { id: number; name: string; role: "admin" | "user"; }

async function fetchUsers(): Promise<User[]> {
  await new Promise(r => setTimeout(r, 1000)); // simulate delay
  return [{ id: 1, name: "Alice", role: "admin" }];
}

async function processUsers() {
  try {
    const users = await fetchUsers(); // TS knows: User[]
    const admins = users.filter(u => u.role === "admin").map(u => u.name);
    console.log(admins);
  } catch (error) {
    if (error instanceof Error) console.error(error.message);
  }
}
```

> All `async` functions return `Promise<T>`. Declaring return type as plain `string` is an error.

```ts
async function get(): string { }         // ❌ Error
async function get(): Promise<string> {} // ✅
```

---

## Promise Combination Methods

| Method | Behavior |
|---|---|
| `Promise.all([...])` | Wait for ALL to resolve — fails fast on any rejection |
| `Promise.race([...])` | First to settle wins (resolve or reject) |
| `Promise.allSettled([...])` | Wait for ALL regardless of success/failure |
| `Promise.any([...])` | First to **fulfill** wins (ignores rejections) |

---

## `Promise.all` — Parallel Execution

```ts
async function loadDashboard(userId: number) {
  const [user, posts, stats] = await Promise.all([
    fetchUser(userId),   // Promise<User>
    fetchPosts(userId),  // Promise<Post[]>
    fetchStats(userId),  // Promise<Stats>
  ]);
  // TS infers each type correctly from the tuple
  console.log(user.name, posts.length, stats.likes);
}
```

---

## `Promise.race` — Timeout Pattern

```ts
const timeout = (ms: number): Promise<never> =>
  new Promise((_, reject) => setTimeout(() => reject(new Error(`Timeout ${ms}ms`)), ms));

async function fetchWithTimeout<T>(promise: Promise<T>, ms = 5000): Promise<T> {
  return Promise.race([promise, timeout(ms)]);
}

const data = await fetchWithTimeout(fetch("/api/user"), 3000);
```

---

## `Promise.allSettled` — Don't Fail Fast

```ts
const results = await Promise.allSettled([fetchA(), fetchB(), fetchC()]);

results.forEach(result => {
  if (result.status === "fulfilled") {
    console.log("Success:", result.value);
  } else {
    console.error("Failed:", result.reason);
  }
});
```

---

## Typed Callbacks

```ts
type FetchCallback = (error: Error | null, data?: string) => void;

function fetchData(url: string, callback: FetchCallback): void {
  setTimeout(() => callback(null, "response"), 1000);
}

fetchData("/api", (err, data) => {
  if (err) { console.error(err.message); return; }
  data?.toUpperCase(); // TS knows: string | undefined
});
```

---

## Custom Error Classes

```ts
class AppError extends Error {
  constructor(message: string, public readonly code: string) {
    super(message);
    this.name = this.constructor.name;
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, id: string | number) {
    super(`${resource} with ID ${id} not found`, "NOT_FOUND");
  }
}

class NetworkError extends AppError {
  constructor(message: string) { super(message, "NETWORK_ERROR"); }
}
```

Handle by type:
```ts
try {
  await fetchUser("123");
} catch (error) {
  if (error instanceof NotFoundError) { /* show 404 */ }
  else if (error instanceof NetworkError) { /* show retry */ }
  else { /* generic fallback */ }
}
```

---

## Async Generator

```ts
async function* generateNumbers(): AsyncGenerator<number> {
  for (let i = 0; i < 5; i++) {
    await new Promise(r => setTimeout(r, 100));
    yield i; // yields number one at a time
  }
}

for await (const num of generateNumbers()) {
  console.log(num * 2); // TS knows: number
}
```

---

## Quick Reference

```ts
// Typed promise
const p: Promise<number> = fetch("/api").then(r => r.json());

// Async function return type
async function load(): Promise<User[]> { ... }

// Parallel
const [a, b] = await Promise.all([fetchA(), fetchB()]);

// Timeout with race
await Promise.race([fetch("/api"), timeout(3000)]);

// All settled (never throws)
const results = await Promise.allSettled([...]);

// Error type guard
catch (e) { if (e instanceof Error) e.message; }
```

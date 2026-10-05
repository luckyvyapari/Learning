# TypeScript Error Handling

---

## Basic Try/Catch

```ts
function divide(a: number, b: number): number {
  if (b === 0) throw new Error("Division by zero");
  return a / b;
}

try {
  divide(10, 0);
} catch (error) {
  if (error instanceof Error) {
    console.error(error.message); // ✅ typed
  }
}
```

> TS 4.0+: caught `error` is `unknown` by default — must narrow before accessing `.message`.

---

## Custom Error Classes

```ts
class ValidationError extends Error {
  constructor(message: string, public field?: string) {
    super(message);
    this.name = "ValidationError";
    Object.setPrototypeOf(this, ValidationError.prototype); // fix instanceof
  }
}

class NetworkError extends Error {
  constructor(public status: number, message: string) {
    super(message);
    this.name = "NetworkError";
    Object.setPrototypeOf(this, NetworkError.prototype);
  }
}
```

---

## Type Guards for Errors

```ts
// Check unknown error has .message
function isErrorWithMessage(e: unknown): e is { message: string } {
  return typeof e === "object" && e !== null && "message" in e;
}

try {
  // ...
} catch (error: unknown) {
  if (error instanceof ValidationError) {
    console.error(`Validation [${error.field}]: ${error.message}`);
  } else if (error instanceof NetworkError) {
    console.error(`Network ${error.status}: ${error.message}`);
  } else if (isErrorWithMessage(error)) {
    console.error(error.message);
  } else {
    console.error("Unknown error");
  }
}
```

Assertion function alternative:
```ts
function assertIsError(e: unknown): asserts e is Error {
  if (!(e instanceof Error)) throw new Error("Not an Error instance");
}

catch (e) {
  assertIsError(e);
  console.error(e.message); // TS knows: Error
}
```

---

## Async Error Handling

```ts
async function fetchUser(id: number): Promise<User> {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new NetworkError(res.status, "Fetch failed");
    return await res.json() as User;
  } catch (error) {
    if (error instanceof Error) console.error(error.message);
    throw error; // re-throw so caller can handle
  }
}
```

Always handle promise rejections:
```ts
// ❌ Unhandled rejection
fetchData().then(d => console.log(d));

// ✅ Handled
fetchData()
  .then(d => console.log(d))
  .catch(e => console.error(e));
```

---

## React Error Boundary

```ts
import { Component, ErrorInfo, ReactNode } from "react";

interface Props { children: ReactNode; fallback?: ReactNode; }
interface State { hasError: boolean; error?: Error; }

class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: ErrorInfo) {
    console.error("Caught:", error, info);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <p>Something went wrong.</p>;
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary fallback={<p>Oops!</p>}>
  <MyComponent />
</ErrorBoundary>
```

---

## Common Pitfalls

```ts
// ❌ Empty catch — silently swallows error
try { ... } catch { }

// ❌ Unknown type — no narrowing
catch (e) { console.log(e.message); } // Error: 'e' is unknown

// ✅ Narrow first
catch (e) {
  if (e instanceof Error) console.log(e.message);
}

// ❌ Unhandled promise
fetchData();

// ✅
fetchData().catch(console.error);
```

---

## Quick Reference

```
Sync error         →  try/catch + instanceof check
Async error        →  try/catch in async fn OR .catch() on promise
Custom errors      →  extend Error + Object.setPrototypeOf fix
Unknown error      →  instanceof / isErrorWithMessage guard
React component    →  ErrorBoundary class component
Never swallow      →  always log or re-throw
```

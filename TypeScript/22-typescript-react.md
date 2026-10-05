# TypeScript with React

---

## Setup with Vite

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
npm run dev
```

---

## `tsconfig.json` for React

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "Node",
    "jsx": "react-jsx",
    "strict": true,
    "skipLibCheck": true,
    "noEmit": true,
    "resolveJsonModule": true,
    "esModuleInterop": true
  },
  "include": ["src"]
}
```

Key options: `"jsx": "react-jsx"` + `"noEmit": true` (Vite handles bundling, not tsc).

---

## Component Props

```ts
// type alias for props
type GreetingProps = {
  name: string;
  age?: number;        // optional
};

export function Greeting({ name, age }: GreetingProps) {
  return (
    <div>
      <h2>Hello, {name}!</h2>
      {age !== undefined && <p>You are {age} years old</p>}
    </div>
  );
}

// Usage
<Greeting name="Alice" />         // ✅
<Greeting name="Alice" age={30} /> // ✅
<Greeting />                       // ❌ name is required
```

---

## `useState` with Types

```ts
import { useState } from "react";

// TS infers type from initial value
const [count, setCount] = useState(0);          // number
const [name, setName] = useState("");           // string

// Explicit type for nullable or complex state
const [user, setUser] = useState<User | null>(null);

interface User { id: number; name: string; }
setUser({ id: 1, name: "Alice" }); // ✅
setUser("Alice");                   // ❌
```

---

## `useRef` with Types

```ts
import { useRef } from "react";

// DOM element ref
const inputRef = useRef<HTMLInputElement>(null);

// Mutable value ref (not tied to DOM)
const timerRef = useRef<number>(0);

// Usage
<input ref={inputRef} />
inputRef.current?.focus();
```

---

## `useEffect`

No special typing needed — just type the deps correctly:

```ts
import { useEffect, useState } from "react";

const [data, setData] = useState<User[]>([]);

useEffect(() => {
  fetch("/api/users")
    .then(res => res.json())
    .then((users: User[]) => setData(users));
}, []); // typed dep array
```

---

## Event Handlers

```ts
// Input change
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value);
};

// Button click
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  e.preventDefault();
};

// Form submit
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault();
};
```

| Event | Type |
|---|---|
| Input change | `React.ChangeEvent<HTMLInputElement>` |
| Button click | `React.MouseEvent<HTMLButtonElement>` |
| Form submit | `React.FormEvent<HTMLFormElement>` |
| Key press | `React.KeyboardEvent<HTMLInputElement>` |

---

## `children` Prop

```ts
import { ReactNode } from "react";

type CardProps = {
  title: string;
  children: ReactNode;  // anything React can render
};

function Card({ title, children }: CardProps) {
  return (
    <div>
      <h3>{title}</h3>
      {children}
    </div>
  );
}

// Usage
<Card title="Info">
  <p>Some content here</p>
</Card>
```

---

## Context with Types

```ts
import { createContext, useContext, useState } from "react";

interface AuthContext {
  user: User | null;
  login: (user: User) => void;
  logout: () => void;
}

const AuthContext = createContext<AuthContext | undefined>(undefined);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  return (
    <AuthContext.Provider value={{
      user,
      login: (u) => setUser(u),
      logout: () => setUser(null)
    }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error("useAuth must be inside AuthProvider");
  return ctx;
}
```

---

## `FC` Type (optional)

```ts
import { FC } from "react";

// FC<Props> = React.FunctionComponent<Props>
const Button: FC<{ label: string; onClick: () => void }> = ({ label, onClick }) => (
  <button onClick={onClick}>{label}</button>
);

// Most devs just use plain function — FC is optional
function Button({ label, onClick }: { label: string; onClick: () => void }) { ... }
```

---

## Quick Reference

| Pattern | How to type |
|---|---|
| Component props | `type Props = { name: string }` |
| Optional prop | `age?: number` |
| useState | `useState<User \| null>(null)` |
| DOM ref | `useRef<HTMLInputElement>(null)` |
| Children | `children: ReactNode` |
| Event handler | `React.ChangeEvent<HTMLInputElement>` |
| Context | `createContext<MyContext \| undefined>(undefined)` |

# TypeScript with Node.js

---

## Setup

```bash
mkdir my-app && cd my-app
npm init -y
npm install typescript @types/node --save-dev
npx tsc --init
```

```bash
# Dev tools
npm install express body-parser
npm install --save-dev ts-node nodemon @types/express
```

---

## Folder Structure

```
my-app/
  src/
    server.ts
    middleware/auth.ts
    entity/User.ts
    config/database.ts
  dist/          ← compiled JS (gitignored)
  package.json
  tsconfig.json
```

---

## `tsconfig.json` for Node

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "rootDir": "./src",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "sourceMap": true,
    "moduleResolution": "node",
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

---

## `package.json` Scripts

```json
{
  "scripts": {
    "dev":   "nodemon --exec ts-node src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js"
  }
}
```

| Script | When to use |
|---|---|
| `npm run dev` | Development — hot reload via nodemon |
| `npm run build` | Compile TS → JS |
| `npm start` | Production — run compiled JS |

---

## Basic Express Server (`src/server.ts`)

```ts
import express, { Request, Response, NextFunction } from "express";
import { json } from "body-parser";

interface User {
  id: number;
  username: string;
  email: string;
}

const app = express();
const PORT = process.env.PORT || 3000;

app.use(json());

const users: User[] = [
  { id: 1, username: "user1", email: "user1@example.com" },
];

// GET all users
app.get("/api/users", (req: Request, res: Response) => {
  res.json(users);
});

// GET user by id
app.get("/api/users/:id", (req: Request, res: Response) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ message: "User not found" });
  res.json(user);
});

// POST create user
app.post("/api/users", (req: Request, res: Response) => {
  const { username, email } = req.body;
  if (!username || !email) {
    return res.status(400).json({ message: "Username and email required" });
  }
  const newUser: User = { id: users.length + 1, username, email };
  users.push(newUser);
  res.status(201).json(newUser);
});

// Error handler
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  res.status(500).json({ message: "Something went wrong!" });
});

app.listen(PORT, () => console.log(`Running on http://localhost:${PORT}`));
```

---

## Auth Middleware with Extended Request Type (`src/middleware/auth.ts`)

```ts
import { Request, Response, NextFunction } from "express";

// Extend Express Request globally
declare global {
  namespace Express {
    interface Request {
      user?: { id: number; role: string };
    }
  }
}

export const authenticate = (req: Request, res: Response, next: NextFunction) => {
  const token = req.header("Authorization")?.replace("Bearer ", "");
  if (!token) return res.status(401).json({ message: "No token" });

  req.user = { id: 1, role: "admin" }; // mock — replace with JWT verify
  next();
};

export const authorize = (roles: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) return res.status(401).json({ message: "Not authenticated" });
    if (!roles.includes(req.user.role)) return res.status(403).json({ message: "Not authorized" });
    next();
  };
};
```

```ts
// Use in server.ts
import { authenticate, authorize } from "./middleware/auth";

app.get("/api/admin", authenticate, authorize(["admin"]), (req, res) => {
  res.json({ message: `Hello admin ${req.user?.id}` });
});
```

---

## TypeORM Entity (`src/entity/User.ts`)

```ts
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn } from "typeorm";

@Entity("users")
export class User {
  @PrimaryGeneratedColumn()   id: number;
  @Column({ unique: true })   username: string;
  @Column({ unique: true })   email: string;
  @Column({ select: false })  password: string;
  @Column({ default: "user" }) role: string;
  @CreateDateColumn()         createdAt: Date;
}
```

Enable decorators in `tsconfig.json`:
```json
{ "experimentalDecorators": true, "emitDecoratorMetadata": true }
```

---

## Database Config (`src/config/database.ts`)

```ts
import "reflect-metadata";
import { DataSource } from "typeorm";
import { User } from "../entity/User";

export const AppDataSource = new DataSource({
  type: "postgres",
  host: process.env.DB_HOST || "localhost",
  port: parseInt(process.env.DB_PORT || "5432"),
  username: process.env.DB_USERNAME || "postgres",
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME || "mydb",
  synchronize: process.env.NODE_ENV !== "production", // never true in prod
  entities: [User],
});

// In server.ts startup:
// AppDataSource.initialize().then(() => app.listen(PORT)).catch(console.error);
```

---

## Debugging with Source Maps

```bash
node --enable-source-maps dist/server.js
```

With `sourceMap: true` in tsconfig — breakpoints map back to `.ts` files in VS Code.

---

## Best Practices

```
ts-node + nodemon  →  dev only
tsc + node         →  production
strict: true       →  always
Never sync: true   →  in production TypeORM
Env vars           →  for DB creds, PORT, secrets
dist/              →  gitignore it
```

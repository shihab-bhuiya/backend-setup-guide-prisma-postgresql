# Backend-setup-guide-Prisma-Postgresql
A reusable backend setup and reference guide using Node.js, Express, TypeScript, Prisma, and PostgreSQL
# Node.js + Express + TypeScript + Prisma + PostgreSQL

## Reusable Backend Setup Documentation
shihab 
This is a clean, repeatable setup guide. Use it whenever you start a new
backend project.

------------------------------------------------------------------------

# 1. Architecture

``` text
Frontend
   ↓
HTTP Request
   ↓
Express
   ↓
Router
   ↓
Service
   ↓
Prisma
   ↓
PostgreSQL
```

### Mental model

-   **Express** → receives requests
-   **Router** → decides which endpoint runs
-   **Service** → contains business/database logic
-   **Prisma** → communicates with PostgreSQL
-   **PostgreSQL** → stores data
-   **Frontend** → communicates only with the Express API

------------------------------------------------------------------------

# 2. Create the Project

``` bash
mkdir my-project-server
cd my-project-server
npm init -y
```

Install Express and basic packages:

``` bash
npm install express cors dotenv
```

Install TypeScript:

``` bash
npm install -D typescript tsx @types/node @types/express @types/cors
```

Initialize TypeScript:

``` bash
npx tsc --init
```

------------------------------------------------------------------------

# 3. Configure TypeScript

Open `tsconfig.json`.

For a modern ESM Node + TypeScript project:

``` json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "rootDir": "./src",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

------------------------------------------------------------------------

# 4. Configure package.json

Add:

``` json
"type": "module"
```

Add scripts:

``` json
"scripts": {
  "dev": "tsx watch src/server.ts",
  "build": "tsc",
  "start": "node dist/server.js"
}
```

------------------------------------------------------------------------

# 5. Create the Folder Structure

Start with:

``` text
my-project-server/
│
├── prisma/
│   └── schema.prisma
│
├── src/
│   ├── app.ts
│   ├── server.ts
│   ├── routes/
│   ├── services/
│   ├── middleware/
│   └── lib/
│
├── .env
├── .gitignore
├── package.json
└── tsconfig.json
```

------------------------------------------------------------------------

# 6. Create .gitignore

``` gitignore
node_modules/
dist/
.env
.env.local
*.log
```

Never commit `.env`.

------------------------------------------------------------------------

# 7. Install Prisma + PostgreSQL Packages

``` bash
npm install -D prisma
npm install @prisma/client @prisma/adapter-pg pg dotenv
```

Initialize Prisma:

``` bash
npx prisma init
```

Depending on the Prisma version, you may also get:

``` text
prisma.config.ts
```

------------------------------------------------------------------------

# 8. Configure the Database

Create your PostgreSQL database and put the connection string in `.env`:

``` env
DATABASE_URL="your-database-url"
PORT=5000
FRONTEND_URL="http://localhost:3000"
```

Never put database credentials directly into source code.

------------------------------------------------------------------------

# 9. Design Prisma Schema

Open:

``` text
prisma/schema.prisma
```

Add your:

-   Models
-   Enums
-   Relations
-   Indexes
-   Timestamps
-   Soft-delete fields

Typical development order:

``` text
Database
   ↓
Models
   ↓
Relations
   ↓
Migration
```

------------------------------------------------------------------------

# 10. Prisma Migration Workflow

After changing `schema.prisma`:

``` bash
npx prisma format
```

Create a migration:

``` bash
npx prisma migrate dev --name init
```

For later changes, use meaningful names:

``` bash
npx prisma migrate dev --name add-category
npx prisma migrate dev --name add-product-status
npx prisma migrate dev --name add-review
```

Do **not** use `init` for every migration.

Generate Prisma Client when needed:

``` bash
npx prisma generate
```

------------------------------------------------------------------------

# 11. Prisma Studio

Open the database visually:

``` bash
npx prisma studio
```

Use it to inspect:

-   Users
-   Categories
-   Products
-   Orders
-   Reviews

------------------------------------------------------------------------

# 12. Create Prisma Client

Create:

``` text
src/lib/prisma.ts
```

``` ts
import "dotenv/config";
import { PrismaPg } from "@prisma/adapter-pg";
import { PrismaClient } from "../generated/prisma/client.js";

const connectionString = process.env.DATABASE_URL;

if (!connectionString) {
  throw new Error("DATABASE_URL is not defined");
}

const adapter = new PrismaPg({
  connectionString,
});

const prisma = new PrismaClient({
  adapter,
});

export default prisma;
```

Architecture:

``` text
Service
   ↓
prisma.ts
   ↓
Prisma Client
   ↓
PostgreSQL
```

------------------------------------------------------------------------

# 13. Create Express App

Create:

``` text
src/app.ts
```

``` ts
import express from "express";
import cors from "cors";

const app = express();

app.use(cors());
app.use(express.json());

app.get("/", (req, res) => {
  res.json({
    success: true,
    message: "Server is running",
  });
});

export default app;
```

------------------------------------------------------------------------

# 14. Create Server

Create:

``` text
src/server.ts
```

``` ts
import "dotenv/config";
import app from "./app.js";

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

Start development:

``` bash
npm run dev
```

------------------------------------------------------------------------

# 15. Routing

Create routes:

``` text
src/routes/
├── user.routes.ts
├── category.routes.ts
├── product.routes.ts
├── review.routes.ts
└── order.routes.ts
```

Basic router:

``` ts
import { Router } from "express";

const userRouter = Router();

userRouter.get("/", async (req, res) => {
  res.json({
    success: true,
    message: "Users fetched successfully",
  });
});

export default userRouter;
```

------------------------------------------------------------------------

# 16. Connect Routers to app.ts

In `app.ts`:

``` ts
import userRouter from "./routes/user.routes.js";

app.use("/api/users", userRouter);
```

The combination creates the endpoint:

``` text
app.use("/api/users", userRouter)
+
userRouter.get("/")
=
GET /api/users
```

Other examples:

``` ts
app.use("/api/categories", categoryRouter);
app.use("/api/products", productRouter);
app.use("/api/reviews", reviewRouter);
app.use("/api/orders", orderRouter);
```

------------------------------------------------------------------------

# 17. Router Rule

Remember:

``` text
app.use("/api/users", userRouter)
```

is the **base path**.

Inside the router:

``` ts
userRouter.get("/");
```

becomes:

``` text
GET /api/users
```

``` ts
userRouter.get("/:id");
```

becomes:

``` text
GET /api/users/:id
```

``` ts
userRouter.post("/");
```

becomes:

``` text
POST /api/users
```

------------------------------------------------------------------------

# 18. Services

Do not put all Prisma queries directly inside routes.

Use:

``` text
routes
   ↓
services
   ↓
Prisma
```

Example structure:

``` text
src/
├── routes/
│   └── user.routes.ts
│
└── services/
    └── user/
        └── user.service.ts
```

------------------------------------------------------------------------

# 19. Example Service

`src/services/user/user.service.ts`

``` ts
import prisma from "../../lib/prisma.js";

export const getAllUsers = async () => {
  return prisma.user.findMany({
    where: {
      isDeleted: false,
    },
    select: {
      id: true,
      name: true,
      email: true,
      role: true,
      createdAt: true,
      updatedAt: true,
    },
  });
};
```

------------------------------------------------------------------------

# 20. Use the Service in a Router

``` ts
import { Router } from "express";
import { getAllUsers } from "../services/user/user.service.js";

const userRouter = Router();

userRouter.get("/", async (req, res) => {
  try {
    const users = await getAllUsers();

    res.status(200).json({
      success: true,
      message: "Users fetched successfully",
      data: users,
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: "Failed to fetch users",
    });
  }
});

export default userRouter;
```

------------------------------------------------------------------------

# 21. CRUD Pattern

For every major resource, remember:

  Operation   Method   Endpoint
  ----------- -------- ---------------------
  Create      POST     `/api/products`
  Read all    GET      `/api/products`
  Read one    GET      `/api/products/:id`
  Update      PATCH    `/api/products/:id`
  Delete      DELETE   `/api/products/:id`

CRUD means:

``` text
Create → POST
Read   → GET
Update → PATCH
Delete → DELETE
```

------------------------------------------------------------------------

# 22. Prisma CRUD Cheat Sheet

### Create

``` ts
await prisma.product.create({
  data: {
    title: "Laptop",
    description: "Gaming laptop",
    price: 80000,
    stock: 10,
    categoryId,
  },
});
```

### Get all

``` ts
await prisma.product.findMany();
```

### Get one

``` ts
await prisma.product.findUnique({
  where: {
    id,
  },
});
```

### Update

``` ts
await prisma.product.update({
  where: {
    id,
  },
  data: {
    title: "New Laptop",
  },
});
```

### Soft delete

``` ts
await prisma.product.update({
  where: {
    id,
  },
  data: {
    isDeleted: true,
  },
});
```

------------------------------------------------------------------------

# 23. findUnique vs findFirst

### findUnique

Use when searching by a unique field:

``` ts
await prisma.user.findUnique({
  where: {
    id,
  },
});
```

Or:

``` ts
await prisma.user.findUnique({
  where: {
    email,
  },
});
```

`email` must be unique for this use.

### findFirst

Use when the conditions are not necessarily unique:

``` ts
await prisma.user.findFirst({
  where: {
    name: "Ahsan",
    isDeleted: false,
  },
});
```

------------------------------------------------------------------------

# 24. select vs include

### select

Use when you only want specific fields:

``` ts
select: {
  id: true,
  name: true,
  email: true,
  role: true,
}
```

Do not return password hashes unnecessarily.

### include

Use when you want related data:

``` ts
await prisma.product.findUnique({
  where: {
    id,
  },
  include: {
    category: true,
    reviews: true,
  },
});
```

Think:

``` text
Product
├── Category
└── Reviews
```

------------------------------------------------------------------------

# 25. Validation with Zod

Install:

``` bash
npm install zod
```

Example:

``` ts
const productSchema = z.object({
  title: z.string().min(1),
  price: z.number().positive(),
  stock: z.number().int().nonnegative(),
});
```

Request flow:

``` text
Request
   ↓
Validation
   ↓
Router
   ↓
Service
   ↓
Database
```

------------------------------------------------------------------------

# 26. Business Validation

Not every validation belongs to Zod.

Examples:

-   Does the category exist?
-   Does the user exist?
-   Is the product deleted?
-   Is there enough stock?
-   Can this user edit this review?

This is business logic and normally belongs in the **service layer**.

------------------------------------------------------------------------

# 27. Error Handling

Create:

``` text
src/middleware/errorHandler.ts
```

Instead of writing complicated error handling in every route, eventually
use:

``` text
Route
   ↓
Service
   ↓
Error
   ↓
Global Error Handler
   ↓
JSON Response
```

------------------------------------------------------------------------

# 28. Consistent API Responses

### Success

``` json
{
  "success": true,
  "message": "Product fetched successfully",
  "data": {}
}
```

### Multiple records

``` json
{
  "success": true,
  "message": "Products fetched successfully",
  "data": []
}
```

### Error

``` json
{
  "success": false,
  "message": "Product not found"
}
```

Keep response structures consistent across the API.

------------------------------------------------------------------------

# 29. Authentication & Authorization

Authentication answers:

> Who is this user?

Authorization answers:

> What can this user do?

For the next project, use Better Auth rather than manually building
authentication.

Conceptually:

``` text
Frontend
   ↓
Better Auth
   ↓
Session
   ↓
Express Backend
   ↓
Protected API
```

Example roles:

``` text
User
├── View products
├── Create orders
└── Create reviews

Admin
├── Manage users
├── Manage products
├── Manage categories
└── Manage orders
```

------------------------------------------------------------------------

# 30. Orders and Transactions

Orders may require multiple database operations.

Example:

``` text
Product stock = 10
Customer buys = 3
New stock = 7
```

You need to:

``` text
Create Order
+
Decrease Stock
```

Use a Prisma transaction:

``` ts
await prisma.$transaction(async (tx) => {
  // create order

  // update stock
});
```

Think:

``` text
Transaction
├── Create Order
├── Update Stock
└── Success?
      ├── YES → Commit
      └── NO  → Rollback
```

------------------------------------------------------------------------

# 31. Testing the API

Test the backend before connecting the frontend.

Tools from the source:

-   Thunder Client
-   Postman
-   Bruno
-   Insomnia

For Products:

``` text
POST   /api/products
GET    /api/products
GET    /api/products/:id
PATCH  /api/products/:id
DELETE /api/products/:id
```

------------------------------------------------------------------------

# 32. Build in This Order

Do not build everything at once.

Use:

``` text
1. Database
      ↓
2. User
      ↓
3. Category
      ↓
4. Product
      ↓
5. Review
      ↓
6. Order
      ↓
7. Authentication
      ↓
8. Authorization
      ↓
9. Frontend
```

This keeps debugging manageable.

------------------------------------------------------------------------

# 33. Connect the Frontend

The frontend does **not** communicate directly with Prisma.

Correct:

``` text
Frontend
   ↓
Express
   ↓
Router
   ↓
Service
   ↓
Prisma
   ↓
PostgreSQL
```

Example:

``` ts
fetch("http://localhost:5000/api/products");
```

The frontend only needs to know the API endpoint.

It does not need to know:

``` text
Prisma
PostgreSQL
SQL
```

------------------------------------------------------------------------

# 34. Frontend Environment Variable

For Next.js:

``` env
NEXT_PUBLIC_API_URL="http://localhost:5000"
```

Then:

``` ts
fetch(`${process.env.NEXT_PUBLIC_API_URL}/api/products`);
```

Production:

``` env
NEXT_PUBLIC_API_URL="https://your-backend-url.com"
```

------------------------------------------------------------------------

# 35. CORS

Development:

``` ts
app.use(cors());
```

Later:

``` ts
app.use(
  cors({
    origin: process.env.FRONTEND_URL,
    credentials: true,
  })
);
```

For cookie/session authentication, correct `origin` and `credentials`
configuration are important.

------------------------------------------------------------------------

# 36. Git Setup

Initialize:

``` bash
git init
```

Add files:

``` bash
git add .
```

First commit:

``` bash
git commit -m "chore: initialize backend project"
```

Good commit examples:

``` text
chore: configure prisma and postgresql
feat: add user schema and migration
feat: add user crud api
feat: add category crud api
feat: add product crud api
feat: add review crud api
feat: add order crud api
feat: add request validation
feat: add error handling
feat: integrate authentication
feat: add role based authorization
docs: add api documentation
```

------------------------------------------------------------------------

# 37. Daily Development Workflow

Whenever you work on the project:

### 1. Start server

``` bash
npm run dev
```

### 2. If Prisma schema changed

``` bash
npx prisma format
```

### 3. Create migration

``` bash
npx prisma migrate dev --name meaningful-name
```

### 4. Generate if needed

``` bash
npx prisma generate
```

### 5. Check database

``` bash
npx prisma studio
```

### 6. Test API

Use Postman, Thunder Client, Bruno, or Insomnia.

### 7. Commit

``` bash
git add .
git commit -m "feat: ..."
```

------------------------------------------------------------------------

# 38. Prisma Command Cheat Sheet

  Purpose            Command
  ------------------ ----------------------------------------------
  Initialize         `npx prisma init`
  Format             `npx prisma format`
  Migration          `npx prisma migrate dev --name init`
  Later migration    `npx prisma migrate dev --name add-category`
  Generate client    `npx prisma generate`
  Open Studio        `npx prisma studio`
  Migration status   `npx prisma migrate status`
  Prisma version     `npx prisma -v`

------------------------------------------------------------------------

# 39. NPM Command Cheat Sheet

``` bash
npm install
```

Install a package:

``` bash
npm install package-name
```

Install a development package:

``` bash
npm install -D package-name
```

Development:

``` bash
npm run dev
```

Build:

``` bash
npm run build
```

Production:

``` bash
npm start
```

------------------------------------------------------------------------

# 40. Final Project Structure

``` text
server/
│
├── prisma/
│   ├── schema.prisma
│   └── migrations/
│
├── src/
│   ├── app.ts
│   ├── server.ts
│   │
│   ├── routes/
│   │   ├── auth.routes.ts
│   │   ├── user.routes.ts
│   │   ├── category.routes.ts
│   │   ├── product.routes.ts
│   │   ├── review.routes.ts
│   │   └── order.routes.ts
│   │
│   ├── services/
│   │   ├── user/
│   │   │   └── user.service.ts
│   │   ├── category/
│   │   │   └── category.service.ts
│   │   ├── product/
│   │   │   └── product.service.ts
│   │   ├── review/
│   │   │   └── review.service.ts
│   │   └── order/
│   │       └── order.service.ts
│   │
│   ├── middleware/
│   │   ├── auth.ts
│   │   ├── validate.ts
│   │   └── errorHandler.ts
│   │
│   └── lib/
│       └── prisma.ts
│
├── .env
├── .gitignore
├── package.json
├── prisma.config.ts
├── tsconfig.json
└── README.md
```

------------------------------------------------------------------------

# 41. Learning Order

Do not learn everything simultaneously.

### Level 1 --- Express

Learn:

``` text
app
middleware
req
res
Router
app.use()
GET
POST
PATCH
DELETE
```

### Level 2 --- PostgreSQL

Learn:

``` text
Database
Table
Row
Column
Primary Key
Foreign Key
Relationship
Index
```

### Level 3 --- Prisma

Learn:

``` text
schema.prisma
model
enum
relation
migration
Prisma Client

findMany
findUnique
findFirst
create
update
delete
select
include
```

### Level 4 --- Backend Architecture

Learn:

``` text
routes
services
middleware
error handling
validation
```

### Level 5 --- Advanced Prisma

Learn:

``` text
transactions
pagination
filtering
search
relations
nested queries
indexes
```

### Level 6 --- Authentication

Learn:

``` text
sessions
cookies
Better Auth
authentication
authorization
roles
```

### Level 7 --- Production

Learn:

``` text
environment variables
CORS
deployment
logging
security
API documentation
frontend integration
```

------------------------------------------------------------------------

# 42. Complete Learning Roadmap

``` text
Express Basics
      ↓
PostgreSQL
      ↓
Prisma
      ↓
CRUD Operations
      ↓
Routes + Services
      ↓
Validation + Error Handling
      ↓
Authentication
      ↓
Authorization
      ↓
Frontend
      ↓
Deployment
```

------------------------------------------------------------------------

# 43. NEW PROJECT CHECKLIST

Copy this section whenever starting a new backend.

## Project Setup

-   [ ] Create PostgreSQL database
-   [ ] Create `.env`
-   [ ] Add `DATABASE_URL`
-   [ ] Run `npx prisma init`
-   [ ] Design schema
-   [ ] Add models
-   [ ] Add enums
-   [ ] Add relations
-   [ ] Add indexes
-   [ ] Add timestamps
-   [ ] Add soft delete

## Prisma

-   [ ] Run `npx prisma format`
-   [ ] Run `npx prisma migrate dev --name init`
-   [ ] Run `npx prisma generate`
-   [ ] Create `prisma.ts`
-   [ ] Run `npx prisma studio`

## Express

-   [ ] Configure CORS
-   [ ] Configure `express.json()`
-   [ ] Create routers
-   [ ] Create services
-   [ ] Connect routers
-   [ ] Implement CRUD
-   [ ] Add validation
-   [ ] Add error handling

## Authentication

-   [ ] Configure Better Auth
-   [ ] Registration
-   [ ] Login
-   [ ] Logout
-   [ ] Sessions
-   [ ] Protected routes
-   [ ] Roles
-   [ ] Authorization

## Testing

-   [ ] Test User
-   [ ] Test Category
-   [ ] Test Product
-   [ ] Test Review
-   [ ] Test Order
-   [ ] Test authentication
-   [ ] Test authorization
-   [ ] Test errors
-   [ ] Test invalid requests

## Frontend

-   [ ] Add API URL
-   [ ] Connect API
-   [ ] Connect authentication
-   [ ] Connect CRUD
-   [ ] Handle loading
-   [ ] Handle errors

## Git

-   [ ] Create `.gitignore`
-   [ ] Run `git init`
-   [ ] Run `git add .`
-   [ ] Create first commit
-   [ ] Create GitHub repository

## Deployment

-   [ ] Production database
-   [ ] Production environment variables
-   [ ] Production CORS URL
-   [ ] Run `npm run build`
-   [ ] Deploy backend
-   [ ] Connect frontend
-   [ ] Test production API

## Documentation

-   [ ] README
-   [ ] API endpoints
-   [ ] Request bodies
-   [ ] Responses
-   [ ] Status codes
-   [ ] Live URL

------------------------------------------------------------------------

# 44. The Only Mental Model You Must Remember

If you forget everything else, remember this:

``` text
FRONTEND
   ↓
API REQUEST
   ↓
EXPRESS
   ↓
ROUTER
   ↓
SERVICE
   ↓
PRISMA
   ↓
POSTGRESQL
```

When the database schema changes:

``` text
schema.prisma
      ↓
prisma format
      ↓
prisma migrate dev
      ↓
PostgreSQL updated
      ↓
Prisma Client generated
      ↓
TypeScript can use the new model
```

The five backend layers:

``` text
1. DATABASE
      ↓
2. PRISMA
      ↓
3. SERVICE
      ↓
4. ROUTER
      ↓
5. EXPRESS
```

And the frontend sits outside:

``` text
FRONTEND
   ↓
EXPRESS
   ↓
ROUTER
   ↓
SERVICE
   ↓
PRISMA
   ↓
POSTGRESQL
```

**app.ts** connects routers.

**Routes** decide which API endpoint is called.

**Services** contain business/database logic.

**Prisma** communicates with PostgreSQL.

**PostgreSQL** stores the data.

**Frontend** communicates with the Express API.

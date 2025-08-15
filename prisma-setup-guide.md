# Prisma Setup Guide for Freelancer Management App

## 1. Install Prisma

First, install Prisma and the Prisma Client in your project:

```bash
npm install prisma @prisma/client
npm install -D prisma
```

## 2. Initialize Prisma

Initialize Prisma in your project:

```bash
npx prisma init
```

This command creates:
- `prisma/schema.prisma` - Your database schema file
- `.env` - Environment variables file (including DATABASE_URL)

## 3. Configure Database Connection

Edit the `.env` file in your project root:

```env
# For PostgreSQL
DATABASE_URL="postgresql://username:password@localhost:5432/freelancer_db"

# For SQLite (recommended for learning)
DATABASE_URL="file:./dev.db"

# For MySQL
DATABASE_URL="mysql://username:password@localhost:3306/freelancer_db"
```

## 4. Define Your Database Models

Edit `prisma/schema.prisma` to define your data structure:

```prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite" // or "postgresql", "mysql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int        @id @default(autoincrement())
  email     String     @unique
  name      String
  customers Customer[]
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt

  @@map("users")
}

model Customer {
  id       Int       @id @default(autoincrement())
  name     String
  phone    String?
  address  String?
  email    String?
  notes    String?
  userId   Int
  user     User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  projects Project[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@map("customers")
}

model Project {
  id         Int      @id @default(autoincrement())
  name       String
  notes      String?
  todoList   String[] // Array of todo items
  price      Float?   // Project price
  timeSpent  Int?     // Time spent in minutes
  customerId Int
  customer   Customer @relation(fields: [customerId], references: [id], onDelete: Cascade)
  createdAt  DateTime @default(now())
  updatedAt  DateTime @updatedAt

  @@map("projects")
}
```

## 5. Generate Prisma Client and Apply Schema

Generate the TypeScript client and create the database tables:

```bash
# Generate the Prisma Client (creates type-safe database client)
npx prisma generate

# Apply the schema to your database (creates tables)
npx prisma db push
```

## 6. Using Prisma in Your TypeScript Code

```typescript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

// Example: Create a new customer
async function createCustomer(userId: number, customerData: {
  name: string
  phone?: string
  address?: string
  email?: string
  notes?: string
}) {
  const customer = await prisma.customer.create({
    data: {
      ...customerData,
      userId
    }
  })
  return customer
}

// Example: Get all customers for a user
async function getUserCustomers(userId: number) {
  const customers = await prisma.customer.findMany({
    where: { userId },
    include: { projects: true } // Include related projects
  })
  return customers
}
```

## 7. Useful Prisma Commands

```bash
# View your database in the browser
npx prisma studio

# Reset your database (careful - deletes all data!)
npx prisma db push --force-reset

# Generate client after schema changes
npx prisma generate

# Format your schema file
npx prisma format
```

## 8. Next Steps

After setting up Prisma:
1. Create your Express.js routes
2. Implement CRUD operations using Prisma Client
3. Add input validation
4. Implement authentication
5. Build your frontend to connect to these APIs

## Database Relationships Explained

- **User** → **Customer**: One user can have many customers (1:N)
- **Customer** → **Project**: One customer can have many projects (1:N)
- **User** → **Project**: Through the customer relationship (1:N:N)

This setup allows each freelancer (user) to manage multiple customers, and each customer can have multiple projects.
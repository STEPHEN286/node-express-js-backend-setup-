Node.js Backend Setup Guide

A reusable setup guide for starting a Node.js backend with Express, TypeScript, Prisma ORM, PostgreSQL, Zod, authentication, security middleware, and common development tools.

This guide uses Yarn 4 with the traditional node_modules folder.

1. Prerequisites

Install these tools before starting:

Node.js

Yarn through Corepack

PostgreSQL

Git

A code editor such as Visual Studio Code

Check the installed versions:

node -v
npm -v
yarn -v
git --version

Enable Corepack when Yarn is unavailable:

corepack enable

2. Create and initialize the project

Create a folder and move into it:

mkdir server
cd server

Initialize the project:

yarn init

3. Configure Yarn 4

Yarn 4 may use Plug'n'Play by default, which does not create a node_modules folder.

Configure Yarn to use the traditional node_modules structure:

yarn config set nodeLinker node-modules

The generated .yarnrc.yml file should contain:

nodeLinker: node-modules

Install or refresh dependencies:

yarn install

4. Install Express and TypeScript

Install Express:

yarn add express

Install TypeScript and the development runner:

yarn add -D typescript tsx @types/node @types/express

Create the TypeScript configuration:

yarn tsc --init

5. Install common backend packages

Install the main runtime dependencies:

yarn add dotenv cors helmet compression morgan cookie-parser zod express-rate-limit http-status-codes date-fns

Install their TypeScript declarations where needed:

yarn add -D @types/cors @types/compression @types/morgan @types/cookie-parser

Package purposes

Package

Purpose

dotenv

Loads environment variables from .env

cors

Controls frontend access to the API

helmet

Adds security-related HTTP headers

compression

Compresses HTTP responses

morgan

Logs incoming HTTP requests

cookie-parser

Reads cookies from incoming requests

zod

Validates requests and environment variables

express-rate-limit

Limits repeated requests

http-status-codes

Provides readable HTTP status constants

date-fns

Provides date utilities

6. Install authentication packages

Install password hashing and JSON Web Token support:

yarn add bcrypt jsonwebtoken

Install the TypeScript declarations:

yarn add -D @types/bcrypt @types/jsonwebtoken

7. Install Prisma ORM with Prisma Postgres

The current Prisma setup uses Prisma ORM, Prisma Client, the PostgreSQL driver adapter, and the pg driver.

Install the Prisma development dependencies:

yarn add -D prisma @types/node @types/pg

Install the runtime dependencies:

yarn add @prisma/client @prisma/adapter-pg pg dotenv

Initialize Prisma:

yarn prisma init --db --output ../generated/prisma

This setup normally creates or updates:

prisma/
└── schema.prisma

prisma.config.ts
.env

The generated Prisma Client will be written to:

generated/prisma/

Do not manually edit files inside generated/prisma. They are recreated by prisma generate.

8. Configure ESM

Add the following field to package.json:

{
  "type": "module"
}

A simple package.json structure can look like this:

{
  "name": "backend",
  "version": "1.0.0",
  "private": true,
  "type": "module"
}

Use an ESM-compatible TypeScript configuration:

{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "rootDir": ".",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "sourceMap": true
  },
  "include": ["src/**/*.ts", "prisma/**/*.ts", "prisma.config.ts"],
  "exclude": ["node_modules", "dist", "generated"]
}

9. Configure environment variables

Create or update .env:

PORT=****

DATABASE_URL="postgresql://USERNAME:PASSWORD@HOST:5432/DATABASE_NAME"

JWT_ACCESS_SECRET="replace-with-a-secure-secret"
JWT_REFRESH_SECRET="replace-with-another-secure-secret"

Do not commit .env to GitHub.

Add this to .gitignore:

node_modules
dist
.env
generated
.yarn/cache
.pnp.*

10. Define a Prisma model

Open prisma/schema.prisma and add a model.

Example:

generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}

datasource db {
  provider = "postgresql"
}

model User {
  id        String   @id @default(cuid())
  name      String
  email     String   @unique
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

11. Create and apply a database migration

Create the first migration:

yarn prisma migrate dev --name init

Generate Prisma Client:

yarn prisma generate

Open Prisma Studio:

yarn prisma studio

Format the Prisma schema:

yarn prisma format

12. Create the basic application files

Recommended starting structure:

src/
├── config/
├── middleware/
├── modules/
├── shared/
├── app.ts
└── server.ts

prisma/
└── schema.prisma

generated/
└── prisma/

src/app.ts

import compression from "compression";
import cookieParser from "cookie-parser";
import cors from "cors";
import express from "express";
import helmet from "helmet";
import morgan from "morgan";

const app = express();

app.use(helmet());
app.use(
  cors({
    origin: "http://localhost:3000",
    credentials: true,
  }),
);
app.use(compression());
app.use(cookieParser());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(morgan("dev"));

app.get("/health", (_request, response) => {
  response.status(200).json({
    success: true,
    message: "Server is running",
  });
});

export default app;

src/server.ts

import "dotenv/config";
import app from "./app.js";

const port = Number(process.env.PORT) || 5000;

app.listen(port, () => {
  console.log(`Server running on http://localhost:${port}`);
});

13. Add scripts to package.json

{
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/src/server.js",
    "typecheck": "tsc --noEmit",
    "prisma:generate": "prisma generate",
    "prisma:migrate": "prisma migrate dev",
    "prisma:studio": "prisma studio",
    "prisma:format": "prisma format"
  }
}

Run the development server:

yarn dev

14. Optional packages

Install these only when the project needs them.

File uploads

yarn add multer
yarn add -D @types/multer

Email

yarn add nodemailer
yarn add -D @types/nodemailer

Redis

yarn add ioredis

Background jobs

yarn add bullmq ioredis

Excel and CSV

yarn add xlsx csv-parse

Short reference codes

yarn add nanoid

15. Quick installation commands

Runtime dependencies

yarn add express dotenv cors helmet compression morgan cookie-parser zod express-rate-limit http-status-codes date-fns bcrypt jsonwebtoken @prisma/client @prisma/adapter-pg pg

Development dependencies

yarn add -D typescript tsx prisma @types/node @types/express @types/pg @types/cors @types/compression @types/morgan @types/cookie-parser @types/bcrypt @types/jsonwebtoken

16. Common Yarn 4 issue

When Yarn does not create node_modules, run:

yarn config set nodeLinker node-modules
yarn install

When installation files are corrupted, remove the generated Yarn files and reinstall.

Windows Command Prompt:

rmdir /s /q .yarn
del yarn.lock
del .pnp.cjs
del .pnp.loader.mjs
yarn install

Only delete files that exist.

Setup checklist

Install Node.js, Yarn, PostgreSQL, and Git

Initialize the project

Configure Yarn to use node_modules

Install Express and TypeScript

Install common middleware

Install authentication packages

Install and initialize Prisma

Configure ESM

Create .env

Define the Prisma schema

Run the first migration

Generate Prisma Client

Add development scripts

Start the server

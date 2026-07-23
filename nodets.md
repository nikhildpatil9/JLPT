# Production-Level Node.js + TypeScript Project Structure

There is no single official Node.js folder structure. For most production APIs, I recommend a **feature-based modular monolith with Clean Architecture boundaries**.

As of July 23, 2026, **Node.js 24 is an LTS release**, while Node.js 26 is the Current release. Production applications should normally use a supported LTS version and pin it in `.nvmrc`, Docker, CI/CD, and `package.json`. ([Node.js][1])

---

## 1. High-level architecture diagram

```text
┌──────────────────────────────────────────────────────────────┐
│                         HTTP Client                          │
│              React / Angular / Mobile / Other API            │
└──────────────────────────────┬───────────────────────────────┘
                               │ HTTP Request
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                    Presentation Layer                        │
│                                                              │
│  Routes → Middleware → Controller → Request Validation       │
│                                                              │
│  Responsibility: HTTP request and response handling          │
└──────────────────────────────┬───────────────────────────────┘
                               │ DTO / Command
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                     Application Layer                        │
│                                                              │
│           Use Cases / Services / Application Ports           │
│                                                              │
│  Responsibility: application workflow and orchestration      │
└──────────────────────────────┬───────────────────────────────┘
                               │ Domain objects
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                        Domain Layer                          │
│                                                              │
│     Entities / Value Objects / Business Rules / Interfaces   │
│                                                              │
│  Responsibility: core business logic                         │
└──────────────────────────────┬───────────────────────────────┘
                               │ Repository interface
                               ▼
┌──────────────────────────────────────────────────────────────┐
│                   Infrastructure Layer                       │
│                                                              │
│ PostgreSQL / Redis / Kafka / Email / External APIs / Storage │
│                                                              │
│  Responsibility: implementation of technical integrations    │
└──────────────────────────────────────────────────────────────┘
```

### Main dependency rule

```text
Presentation ───────► Application ───────► Domain
                            ▲
                            │
Infrastructure ─────────────┘
```

The important rule is:

```text
Domain must not import Express, Prisma, PostgreSQL, Redis or Kafka.
```

The business layer should remain usable even when the framework or database changes.

---

# 2. Recommended project structure

```text
node-typescript-api/
│
├── src/
│   ├── server.ts
│   ├── app.ts
│   │
│   ├── config/
│   │   ├── env.ts
│   │   ├── app.config.ts
│   │   ├── database.config.ts
│   │   ├── logger.config.ts
│   │   └── index.ts
│   │
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   │   └── refresh-token.entity.ts
│   │   │   │   ├── value-objects/
│   │   │   │   │   └── password.value-object.ts
│   │   │   │   ├── repositories/
│   │   │   │   │   └── token.repository.ts
│   │   │   │   └── errors/
│   │   │   │       └── invalid-credentials.error.ts
│   │   │   │
│   │   │   ├── application/
│   │   │   │   ├── use-cases/
│   │   │   │   │   ├── login.use-case.ts
│   │   │   │   │   ├── refresh-token.use-case.ts
│   │   │   │   │   └── logout.use-case.ts
│   │   │   │   ├── dto/
│   │   │   │   │   ├── login.input.ts
│   │   │   │   │   └── login.output.ts
│   │   │   │   └── ports/
│   │   │   │       ├── password-hasher.port.ts
│   │   │   │       └── token-service.port.ts
│   │   │   │
│   │   │   ├── infrastructure/
│   │   │   │   ├── persistence/
│   │   │   │   │   └── postgres-token.repository.ts
│   │   │   │   ├── security/
│   │   │   │   │   ├── bcrypt-password-hasher.ts
│   │   │   │   │   └── jwt-token.service.ts
│   │   │   │   └── mappers/
│   │   │   │       └── token.mapper.ts
│   │   │   │
│   │   │   ├── presentation/
│   │   │   │   └── http/
│   │   │   │       ├── auth.controller.ts
│   │   │   │       ├── auth.routes.ts
│   │   │   │       ├── auth.schema.ts
│   │   │   │       └── auth.presenter.ts
│   │   │   │
│   │   │   └── auth.module.ts
│   │   │
│   │   ├── users/
│   │   │   ├── domain/
│   │   │   ├── application/
│   │   │   ├── infrastructure/
│   │   │   ├── presentation/
│   │   │   └── users.module.ts
│   │   │
│   │   ├── orders/
│   │   │   ├── domain/
│   │   │   ├── application/
│   │   │   ├── infrastructure/
│   │   │   ├── presentation/
│   │   │   └── orders.module.ts
│   │   │
│   │   └── health/
│   │       ├── health.controller.ts
│   │       └── health.routes.ts
│   │
│   ├── infrastructure/
│   │   ├── database/
│   │   │   ├── client.ts
│   │   │   ├── migrations/
│   │   │   ├── seeds/
│   │   │   └── transaction-manager.ts
│   │   │
│   │   ├── cache/
│   │   │   ├── redis.client.ts
│   │   │   └── redis-cache.service.ts
│   │   │
│   │   ├── messaging/
│   │   │   ├── kafka.client.ts
│   │   │   ├── message-producer.ts
│   │   │   └── message-consumer.ts
│   │   │
│   │   ├── observability/
│   │   │   ├── logger.ts
│   │   │   ├── metrics.ts
│   │   │   ├── tracing.ts
│   │   │   └── request-context.ts
│   │   │
│   │   └── external-services/
│   │       ├── email.client.ts
│   │       ├── payment.client.ts
│   │       └── storage.client.ts
│   │
│   ├── shared/
│   │   ├── constants/
│   │   ├── errors/
│   │   │   ├── application.error.ts
│   │   │   ├── validation.error.ts
│   │   │   ├── not-found.error.ts
│   │   │   └── conflict.error.ts
│   │   │
│   │   ├── middleware/
│   │   │   ├── authentication.middleware.ts
│   │   │   ├── authorization.middleware.ts
│   │   │   ├── error-handler.middleware.ts
│   │   │   ├── request-id.middleware.ts
│   │   │   └── rate-limit.middleware.ts
│   │   │
│   │   ├── types/
│   │   ├── utils/
│   │   └── validation/
│   │
│   └── bootstrap/
│       ├── dependency-container.ts
│       ├── register-routes.ts
│       └── shutdown.ts
│
├── tests/
│   ├── integration/
│   │   ├── auth/
│   │   └── users/
│   ├── e2e/
│   ├── fixtures/
│   ├── factories/
│   └── setup/
│
├── scripts/
│   ├── migrate.ts
│   ├── seed.ts
│   └── create-admin.ts
│
├── docs/
│   ├── architecture.md
│   ├── api.md
│   └── decisions/
│       └── 001-use-modular-monolith.md
│
├── deploy/
│   ├── docker/
│   ├── kubernetes/
│   └── helm/
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── security.yml
│       └── deploy.yml
│
├── .env.example
├── .gitignore
├── .dockerignore
├── .editorconfig
├── .nvmrc
├── Dockerfile
├── docker-compose.yml
├── eslint.config.js
├── package.json
├── package-lock.json
├── tsconfig.json
├── tsconfig.build.json
└── README.md
```

---

# 3. Why feature-based structure is better

Avoid organizing the complete application like this:

```text
src/
├── controllers/
├── services/
├── repositories/
├── models/
└── routes/
```

That structure is acceptable for a very small application, but becomes difficult to maintain when the application grows.

For example:

```text
controllers/
├── auth.controller.ts
├── user.controller.ts
├── order.controller.ts
├── payment.controller.ts
└── notification.controller.ts
```

Files belonging to one feature become spread across the entire project.

Instead, organize by business module:

```text
modules/
├── auth/
├── users/
├── orders/
├── payments/
└── notifications/
```

Now each module contains everything related to that feature.

```text
users/
├── domain/
├── application/
├── infrastructure/
└── presentation/
```

This provides:

* Better module ownership.
* Easier testing.
* Reduced cross-module coupling.
* Easier future extraction into microservices.
* Better navigation for large teams.

---

# 4. Responsibility of every main folder

## `server.ts`

`server.ts` should only handle process-level startup:

```text
Load configuration
       ↓
Connect to database
       ↓
Build application
       ↓
Start HTTP server
       ↓
Register SIGTERM/SIGINT handlers
```

```ts
import { createApp } from "./app.js";
import { config } from "./config/index.js";
import { connectDatabase, disconnectDatabase } from "./infrastructure/database/client.js";
import { logger } from "./infrastructure/observability/logger.js";

async function bootstrap(): Promise<void> {
  await connectDatabase();

  const app = createApp();

  const server = app.listen(config.port, () => {
    logger.info(
      {
        port: config.port,
        environment: config.nodeEnv,
      },
      "Application started",
    );
  });

  const shutdown = async (signal: string): Promise<void> => {
    logger.info({ signal }, "Shutdown signal received");

    server.close(async (error) => {
      if (error) {
        logger.error({ error }, "HTTP server shutdown failed");
        process.exitCode = 1;
        return;
      }

      await disconnectDatabase();

      logger.info("Application stopped");
      process.exitCode = 0;
    });
  };

  process.once("SIGTERM", () => void shutdown("SIGTERM"));
  process.once("SIGINT", () => void shutdown("SIGINT"));
}

bootstrap().catch((error: unknown) => {
  logger.fatal({ error }, "Application startup failed");
  process.exitCode = 1;
});
```

Production services should react to `SIGTERM`, stop accepting new traffic, complete active requests, and close resources such as database connections. Health checks should separately represent liveness and readiness when deployed with systems such as Kubernetes. ([Express.js][2])

---

## `app.ts`

`app.ts` should build and configure the web application.

It should not start the HTTP server.

```ts
import express, { type Express } from "express";
import helmet from "helmet";

import { registerRoutes } from "./bootstrap/register-routes.js";
import { errorHandler } from "./shared/middleware/error-handler.middleware.js";
import { requestIdMiddleware } from "./shared/middleware/request-id.middleware.js";

export function createApp(): Express {
  const app = express();

  app.disable("x-powered-by");

  app.use(helmet());
  app.use(express.json({ limit: "1mb" }));
  app.use(requestIdMiddleware);

  registerRoutes(app);

  // Error middleware must be registered last.
  app.use(errorHandler);

  return app;
}
```

Keeping `app.ts` separate from `server.ts` makes integration testing easier because tests can create the application without opening a real network port.

---

## `config/`

This folder loads and validates configuration.

```text
Environment variables
        ↓
Validation
        ↓
Typed immutable configuration
        ↓
Used by application
```

Do not access `process.env` throughout the application:

```ts
// Avoid
const port = process.env.PORT;
```

Access it once through the configuration module:

```ts
// Preferred
import { config } from "./config/index.js";

console.log(config.port);
```

Node.js supports environment variables through `process.env`, has an official `.env` specification, and supports loading environment files through `--env-file`. Environment values are still strings, so the application should validate and convert them during startup. ([Node.js][3])

Example using Zod:

```ts
import { z } from "zod";

const envSchema = z.object({
  NODE_ENV: z
    .enum(["development", "test", "production"])
    .default("development"),

  PORT: z.coerce.number().int().positive().default(3000),

  DATABASE_URL: z.string().min(1),

  JWT_ACCESS_SECRET: z.string().min(32),

  JWT_REFRESH_SECRET: z.string().min(32),

  LOG_LEVEL: z
    .enum(["fatal", "error", "warn", "info", "debug", "trace"])
    .default("info"),
});

const result = envSchema.safeParse(process.env);

if (!result.success) {
  console.error("Invalid environment configuration", result.error.flatten());
  throw new Error("Environment validation failed");
}

export const config = Object.freeze({
  nodeEnv: result.data.NODE_ENV,
  port: result.data.PORT,
  databaseUrl: result.data.DATABASE_URL,
  jwt: {
    accessSecret: result.data.JWT_ACCESS_SECRET,
    refreshSecret: result.data.JWT_REFRESH_SECRET,
  },
  logLevel: result.data.LOG_LEVEL,
});
```

---

# 5. Structure inside a business module

Consider the `users` module:

```text
users/
├── domain/
├── application/
├── infrastructure/
├── presentation/
└── users.module.ts
```

## Domain layer

```text
domain/
├── entities/
│   └── user.entity.ts
├── value-objects/
│   └── email.value-object.ts
├── repositories/
│   └── user.repository.ts
├── services/
│   └── user-policy.service.ts
└── errors/
    └── user-not-found.error.ts
```

The domain layer contains:

* Business entities.
* Business rules.
* Repository interfaces.
* Value objects.
* Domain errors.
* Domain services.

It should not contain:

* Express request or response types.
* ORM models.
* SQL queries.
* HTTP status codes.
* Redis clients.
* Kafka clients.

### Domain entity example

```ts
export interface UserProperties {
  id: string;
  name: string;
  email: string;
  isActive: boolean;
  createdAt: Date;
}

export class User {
  private constructor(private readonly properties: UserProperties) {}

  static create(properties: UserProperties): User {
    if (properties.name.trim().length < 2) {
      throw new Error("User name must contain at least two characters");
    }

    return new User({
      ...properties,
      email: properties.email.toLowerCase(),
    });
  }

  get id(): string {
    return this.properties.id;
  }

  get email(): string {
    return this.properties.email;
  }

  get name(): string {
    return this.properties.name;
  }

  get isActive(): boolean {
    return this.properties.isActive;
  }

  deactivate(): User {
    return User.create({
      ...this.properties,
      isActive: false,
    });
  }
}
```

---

## Application layer

```text
application/
├── use-cases/
│   ├── create-user.use-case.ts
│   ├── get-user.use-case.ts
│   ├── update-user.use-case.ts
│   └── delete-user.use-case.ts
├── dto/
│   ├── create-user.input.ts
│   └── user.output.ts
└── ports/
    ├── password-hasher.port.ts
    └── event-publisher.port.ts
```

The application layer:

* Executes a business operation.
* Coordinates repositories and services.
* Defines transaction boundaries.
* Returns application-level output.
* Must not directly know about Express.

### Use-case example

```ts
import type { UserRepository } from "../../domain/repositories/user.repository.js";
import { User } from "../../domain/entities/user.entity.js";
import type { CreateUserInput } from "../dto/create-user.input.js";
import type { UserOutput } from "../dto/user.output.js";

export class CreateUserUseCase {
  constructor(private readonly userRepository: UserRepository) {}

  async execute(input: CreateUserInput): Promise<UserOutput> {
    const existingUser = await this.userRepository.findByEmail(input.email);

    if (existingUser) {
      throw new Error("A user with this email already exists");
    }

    const user = User.create({
      id: crypto.randomUUID(),
      name: input.name,
      email: input.email,
      isActive: true,
      createdAt: new Date(),
    });

    await this.userRepository.save(user);

    return {
      id: user.id,
      name: user.name,
      email: user.email,
      isActive: user.isActive,
    };
  }
}
```

---

## Repository interface

The domain or application layer defines what it needs:

```ts
import type { User } from "../entities/user.entity.js";

export interface UserRepository {
  findById(id: string): Promise<User | null>;

  findByEmail(email: string): Promise<User | null>;

  save(user: User): Promise<void>;

  delete(id: string): Promise<void>;
}
```

This is an abstraction. It does not contain PostgreSQL or ORM-specific details.

---

## Infrastructure layer

```text
infrastructure/
├── persistence/
│   ├── postgres-user.repository.ts
│   └── user.database-model.ts
└── mappers/
    └── user.mapper.ts
```

The infrastructure layer implements repository interfaces.

```ts
import type { UserRepository } from "../../domain/repositories/user.repository.js";
import type { User } from "../../domain/entities/user.entity.js";

export class PostgresUserRepository implements UserRepository {
  async findById(id: string): Promise<User | null> {
    // Query PostgreSQL or ORM here.
    return null;
  }

  async findByEmail(email: string): Promise<User | null> {
    // Query PostgreSQL or ORM here.
    return null;
  }

  async save(user: User): Promise<void> {
    // Insert or update database record here.
  }

  async delete(id: string): Promise<void> {
    // Delete or soft-delete record here.
  }
}
```

---

## Presentation layer

```text
presentation/http/
├── user.controller.ts
├── user.routes.ts
├── user.schema.ts
└── user.presenter.ts
```

### Validation schema

```ts
import { z } from "zod";

export const createUserSchema = z.object({
  body: z.object({
    name: z.string().trim().min(2).max(100),
    email: z.string().email(),
  }),
});

export type CreateUserRequest = z.infer<
  typeof createUserSchema
>["body"];
```

### Controller

```ts
import type { Request, Response, NextFunction } from "express";

import type { CreateUserUseCase } from "../../application/use-cases/create-user.use-case.js";
import type { CreateUserRequest } from "./user.schema.js";

export class UserController {
  constructor(private readonly createUserUseCase: CreateUserUseCase) {}

  create = async (
    request: Request<object, object, CreateUserRequest>,
    response: Response,
    next: NextFunction,
  ): Promise<void> => {
    try {
      const result = await this.createUserUseCase.execute(request.body);

      response.status(201).json({
        success: true,
        data: result,
      });
    } catch (error: unknown) {
      next(error);
    }
  };
}
```

The controller should only:

```text
Read HTTP input
      ↓
Call use case
      ↓
Convert result into HTTP response
```

Do not place database queries or large business rules inside controllers.

---

# 6. Complete request flow

```text
POST /api/v1/users
          │
          ▼
┌────────────────────────┐
│ Global middleware      │
│ Request ID / Logging   │
│ Security / Rate limit  │
└───────────┬────────────┘
            ▼
┌────────────────────────┐
│ Route middleware       │
│ Authentication         │
│ Authorization          │
│ Request validation     │
└───────────┬────────────┘
            ▼
┌────────────────────────┐
│ UserController.create  │
└───────────┬────────────┘
            ▼
┌────────────────────────┐
│ CreateUserUseCase      │
│ Business workflow      │
└───────────┬────────────┘
            ▼
┌────────────────────────┐
│ UserRepository         │
│ Interface              │
└───────────┬────────────┘
            ▼
┌────────────────────────┐
│ PostgresUserRepository │
│ Database implementation│
└───────────┬────────────┘
            ▼
┌────────────────────────┐
│ PostgreSQL             │
└───────────┬────────────┘
            │
            ▼
       HTTP Response
```

---

# 7. Module composition

Each module should expose a small public interface.

```ts
import type { Router } from "express";

import { CreateUserUseCase } from "./application/use-cases/create-user.use-case.js";
import { PostgresUserRepository } from "./infrastructure/persistence/postgres-user.repository.js";
import { UserController } from "./presentation/http/user.controller.js";
import { createUserRouter } from "./presentation/http/user.routes.js";

export interface UsersModule {
  router: Router;
}

export function createUsersModule(): UsersModule {
  const userRepository = new PostgresUserRepository();

  const createUserUseCase = new CreateUserUseCase(userRepository);

  const userController = new UserController(createUserUseCase);

  return {
    router: createUserRouter(userController),
  };
}
```

The route registration file then composes the modules:

```ts
import type { Express } from "express";

import { createUsersModule } from "../modules/users/users.module.js";
import { createAuthModule } from "../modules/auth/auth.module.js";

export function registerRoutes(app: Express): void {
  const authModule = createAuthModule();
  const usersModule = createUsersModule();

  app.use("/api/v1/auth", authModule.router);
  app.use("/api/v1/users", usersModule.router);
}
```

This is manual dependency injection. A dependency-injection library is not required unless dependency construction becomes difficult to manage.

---

# 8. Error-handling design

Use an application error hierarchy:

```text
Error
  │
  └── ApplicationError
        ├── ValidationError          → 400
        ├── AuthenticationError      → 401
        ├── AuthorizationError       → 403
        ├── NotFoundError            → 404
        ├── ConflictError            → 409
        └── InfrastructureError      → 500/503
```

```ts
export abstract class ApplicationError extends Error {
  protected constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number,
    public readonly details?: unknown,
  ) {
    super(message);
    this.name = new.target.name;

    Error.captureStackTrace(this, new.target);
  }
}

export class NotFoundError extends ApplicationError {
  constructor(resource: string, identifier?: string) {
    super(
      identifier
        ? `${resource} '${identifier}' was not found`
        : `${resource} was not found`,
      "RESOURCE_NOT_FOUND",
      404,
    );
  }
}
```

### Central error middleware

```ts
import type {
  ErrorRequestHandler,
  NextFunction,
  Request,
  Response,
} from "express";

import { ApplicationError } from "../errors/application.error.js";
import { logger } from "../../infrastructure/observability/logger.js";

export const errorHandler: ErrorRequestHandler = (
  error: unknown,
  request: Request,
  response: Response,
  _next: NextFunction,
): void => {
  if (error instanceof ApplicationError) {
    response.status(error.statusCode).json({
      success: false,
      error: {
        code: error.code,
        message: error.message,
        details: error.details,
        requestId: request.headers["x-request-id"],
      },
    });

    return;
  }

  logger.error(
    {
      error,
      path: request.path,
      method: request.method,
    },
    "Unhandled application error",
  );

  response.status(500).json({
    success: false,
    error: {
      code: "INTERNAL_SERVER_ERROR",
      message: "An unexpected error occurred",
      requestId: request.headers["x-request-id"],
    },
  });
};
```

Never expose stack traces, SQL errors, internal paths or secrets in production responses.

---

# 9. Recommended TypeScript configuration

For modern Node.js projects, the TypeScript documentation recommends `NodeNext` for Node’s module behavior. Node.js also recommends explicitly defining the `type` field in `package.json`; setting it to `module` makes emitted `.js` files use ES modules. ([TypeScript][4])

## `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2023",
    "lib": ["ES2023"],

    "module": "NodeNext",
    "moduleResolution": "NodeNext",

    "rootDir": ".",
    "outDir": "./dist",

    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,
    "useUnknownInCatchVariables": true,

    "verbatimModuleSyntax": true,
    "isolatedModules": true,
    "resolveJsonModule": true,

    "sourceMap": true,
    "inlineSources": true,

    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,

    "types": ["node"]
  },
  "include": ["src/**/*.ts", "tests/**/*.ts", "scripts/**/*.ts"],
  "exclude": ["node_modules", "dist", "coverage"]
}
```

## `tsconfig.build.json`

```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "rootDir": "./src",
    "outDir": "./dist",
    "sourceMap": true,
    "declaration": false,
    "noEmit": false
  },
  "include": ["src/**/*.ts"],
  "exclude": [
    "node_modules",
    "dist",
    "tests",
    "**/*.test.ts",
    "**/*.spec.ts"
  ]
}
```

With `NodeNext`, local ESM imports generally use the emitted `.js` extension:

```ts
import { createApp } from "./app.js";
```

Even though the source file is:

```text
app.ts
```

---

# 10. Recommended `package.json`

```json
{
  "name": "production-node-typescript-api",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "engines": {
    "node": ">=24 <25"
  },
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc -p tsconfig.build.json",
    "start": "node --enable-source-maps dist/server.js",

    "typecheck": "tsc --noEmit",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check .",

    "test": "vitest run",
    "test:watch": "vitest",
    "test:unit": "vitest run tests/unit",
    "test:integration": "vitest run tests/integration",
    "test:coverage": "vitest run --coverage",

    "check": "npm run typecheck && npm run lint && npm run test && npm run build"
  }
}
```

Node also provides a stable built-in test runner through `node:test`, so using Vitest or Jest is optional rather than mandatory. ([Node.js][5])

Pin the runtime:

## `.nvmrc`

```text
24
```

---

# 11. Testing structure

Use a combination of colocated unit tests and separate integration/E2E tests.

```text
src/
└── modules/
    └── users/
        └── application/
            └── use-cases/
                ├── create-user.use-case.ts
                └── create-user.use-case.spec.ts

tests/
├── integration/
│   └── users/
│       └── create-user.integration.spec.ts
├── e2e/
│   └── user-lifecycle.e2e.spec.ts
├── fixtures/
├── factories/
└── setup/
```

### Testing pyramid

```text
                  ┌───────────────┐
                  │     E2E       │
                  │ Few and slow  │
                  └───────┬───────┘
                      ┌───▼────┐
                      │Integration│
                      │ Moderate │
                      └────┬────┘
                 ┌─────────▼─────────┐
                 │    Unit tests     │
                 │ Many and fast     │
                 └───────────────────┘
```

Test responsibilities:

| Test type   | Tests                                      |
| ----------- | ------------------------------------------ |
| Unit        | Entity rules, value objects, use cases     |
| Integration | Repository, database, Redis, Kafka, routes |
| E2E         | Complete API workflow                      |
| Contract    | External API or message contracts          |
| Load        | Throughput, latency and concurrency        |

---

# 12. Database organization

Keep database details under infrastructure.

```text
infrastructure/database/
├── client.ts
├── migrations/
├── seeds/
├── transaction-manager.ts
└── health-check.ts
```

Feature-specific repositories remain inside their respective modules:

```text
modules/users/infrastructure/persistence/
└── postgres-user.repository.ts
```

This distinction is useful:

```text
Global database infrastructure
├── Connection pool
├── Transaction manager
├── Migration configuration
└── Health check

Module persistence
├── User repository
├── Order repository
└── Payment repository
```

Do not create separate database connections inside every repository. Create one pool/client during application startup and inject or import the controlled database abstraction.

---

# 13. API versioning

Use versioning at the routing level:

```text
/api/v1/users
/api/v1/orders
/api/v1/auth
```

Structure:

```text
presentation/http/
├── v1/
│   ├── user.controller.ts
│   ├── user.routes.ts
│   └── user.schema.ts
└── v2/
    ├── user.controller.ts
    ├── user.routes.ts
    └── user.schema.ts
```

Do not duplicate domain logic for each version. Usually only presentation DTOs, validation and response formats should change.

---

# 14. Health endpoints

Provide at least two endpoints:

```text
GET /health/live
GET /health/ready
```

## Liveness

Answers:

```text
Is the Node.js process running?
```

```ts
app.get("/health/live", (_request, response) => {
  response.status(200).json({
    status: "UP",
    timestamp: new Date().toISOString(),
  });
});
```

## Readiness

Answers:

```text
Can this instance accept traffic?
```

It may check:

* Database connectivity.
* Redis availability.
* Required external dependencies.
* Application initialization state.

```ts
app.get("/health/ready", async (_request, response) => {
  const databaseHealthy = await checkDatabaseHealth();

  if (!databaseHealthy) {
    response.status(503).json({
      status: "DOWN",
      checks: {
        database: "DOWN",
      },
    });

    return;
  }

  response.status(200).json({
    status: "UP",
    checks: {
      database: "UP",
    },
  });
});
```

---

# 15. Logging and observability

Use structured JSON logs rather than `console.log()`.

```ts
logger.info(
  {
    requestId,
    userId,
    method: request.method,
    path: request.path,
    durationMs,
    statusCode,
  },
  "HTTP request completed",
);
```

Recommended observability structure:

```text
Observability
├── Logs
│   ├── Request ID
│   ├── User ID
│   ├── Operation
│   ├── Duration
│   └── Error context
│
├── Metrics
│   ├── Request count
│   ├── Error rate
│   ├── Request duration
│   ├── Database pool usage
│   └── Queue lag
│
└── Tracing
    ├── Incoming HTTP request
    ├── Database query
    ├── External API
    └── Kafka/message processing
```

Never log:

```text
Passwords
Access tokens
Refresh tokens
Authorization headers
Database URLs containing credentials
Payment card information
Sensitive personal information
```

---

# 16. Security placement

```text
src/
├── shared/
│   └── middleware/
│       ├── authentication.middleware.ts
│       ├── authorization.middleware.ts
│       ├── rate-limit.middleware.ts
│       └── security-headers.middleware.ts
│
└── modules/
    └── auth/
        ├── application/
        ├── domain/
        └── infrastructure/
            └── security/
                ├── jwt-token.service.ts
                └── password-hasher.ts
```

Security should cover:

```text
Internet
   │
   ▼
Reverse Proxy / Load Balancer
   │
   ├── TLS
   ├── Request-size limits
   └── Basic filtering
   │
   ▼
Node application
   │
   ├── Security headers
   ├── Rate limiting
   ├── Input validation
   ├── Authentication
   ├── Authorization
   ├── Parameterized queries
   └── Central error handling
```

Express’s production guidance covers security controls, production error handling, process management and reverse-proxy deployment concerns. ([Express.js][6])

---

# 17. CI/CD validation flow

```text
Developer pushes code
          │
          ▼
┌────────────────────┐
│ Install dependencies│
│ npm ci             │
└─────────┬──────────┘
          ▼
┌────────────────────┐
│ Type checking      │
│ tsc --noEmit       │
└─────────┬──────────┘
          ▼
┌────────────────────┐
│ Lint and formatting│
└─────────┬──────────┘
          ▼
┌────────────────────┐
│ Unit tests         │
└─────────┬──────────┘
          ▼
┌────────────────────┐
│ Integration tests  │
└─────────┬──────────┘
          ▼
┌────────────────────┐
│ Production build   │
└─────────┬──────────┘
          ▼
┌────────────────────┐
│ Security scanning  │
└─────────┬──────────┘
          ▼
┌────────────────────┐
│ Docker image build │
└─────────┬──────────┘
          ▼
┌────────────────────┐
│ Deploy             │
└────────────────────┘
```

A minimum pull-request command should be:

```bash
npm ci
npm run check
```

---

# 18. Files that must not be committed

## `.gitignore`

```gitignore
# Dependencies
node_modules/

# Build output
dist/
coverage/

# Environment files
.env
.env.*
!.env.example

# Logs
logs/
*.log

# Runtime
*.pid
*.pid.lock

# IDE
.idea/
.vscode/*
!.vscode/extensions.json
!.vscode/settings.json

# OS
.DS_Store
Thumbs.db

# Test containers and local data
.local/
tmp/
```

## `.env.example`

```dotenv
NODE_ENV=development
PORT=3000

DATABASE_URL=postgresql://username:password@localhost:5432/application

JWT_ACCESS_SECRET=replace-with-at-least-32-characters
JWT_REFRESH_SECRET=replace-with-at-least-32-characters

REDIS_URL=redis://localhost:6379
LOG_LEVEL=info
```

Only examples should be committed. Real credentials should come from the deployment environment or a secrets manager.

---

# 19. Naming conventions

Use one consistent naming pattern.

```text
user.entity.ts
user.repository.ts
postgres-user.repository.ts
create-user.use-case.ts
create-user.input.ts
create-user.schema.ts
user.controller.ts
user.routes.ts
authentication.middleware.ts
application.error.ts
```

Recommended conventions:

| Element         | Convention                   | Example                   |
| --------------- | ---------------------------- | ------------------------- |
| Files           | kebab-case                   | `create-user.use-case.ts` |
| Classes         | PascalCase                   | `CreateUserUseCase`       |
| Functions       | camelCase                    | `createUser()`            |
| Variables       | camelCase                    | `currentUser`             |
| Constants       | UPPER_SNAKE_CASE             | `MAX_LOGIN_ATTEMPTS`      |
| Interfaces      | Descriptive PascalCase       | `UserRepository`          |
| Boolean values  | `is`, `has`, `can`, `should` | `isActive`                |
| Database tables | snake_case                   | `refresh_tokens`          |
| API paths       | plural kebab-case            | `/api/v1/user-profiles`   |

Avoid unnecessary prefixes such as:

```ts
interface IUserRepository {}
class CUserService {}
type TUserInput = {};
```

Prefer:

```ts
interface UserRepository {}
class UserService {}
type UserInput = {};
```

---

# 20. Common production mistakes

```text
❌ Business logic inside controllers
❌ Database queries directly inside routes
❌ process.env used throughout the codebase
❌ One enormous service file
❌ Shared folder containing business-specific code
❌ Returning ORM entities directly through APIs
❌ Logging passwords, tokens or secrets
❌ Catching errors and silently ignoring them
❌ Missing request validation
❌ Missing graceful shutdown
❌ Missing readiness and liveness endpoints
❌ Creating one database pool per request
❌ Importing infrastructure code into domain entities
❌ Mixing CommonJS and ESM without a clear reason
❌ Using path aliases that work in TypeScript but fail at runtime
❌ Running development tools in the production container
```

---

# 21. Recommended starting structure

You do not need every enterprise folder on day one. Start with:

```text
src/
├── server.ts
├── app.ts
├── config/
├── modules/
│   ├── auth/
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.repository.ts
│   │   ├── auth.schema.ts
│   │   └── auth.routes.ts
│   └── users/
│       ├── user.controller.ts
│       ├── user.service.ts
│       ├── user.repository.ts
│       ├── user.schema.ts
│       └── user.routes.ts
├── infrastructure/
│   ├── database/
│   └── observability/
└── shared/
    ├── errors/
    ├── middleware/
    ├── types/
    └── utils/
```

When a module becomes complex, expand it:

```text
users/
├── domain/
├── application/
├── infrastructure/
└── presentation/
```

This avoids overengineering while preserving a clear growth path.

---

# Final recommended architecture

```text
Production Node.js TypeScript Application
│
├── Feature-based modules
│   ├── Auth
│   ├── Users
│   ├── Orders
│   └── Payments
│
├── Layered module internals
│   ├── Presentation
│   ├── Application
│   ├── Domain
│   └── Infrastructure
│
├── Shared technical capabilities
│   ├── Configuration
│   ├── Error handling
│   ├── Logging
│   ├── Database
│   ├── Cache
│   └── Messaging
│
├── Operational readiness
│   ├── Health checks
│   ├── Graceful shutdown
│   ├── Metrics
│   ├── Tracing
│   └── Structured logging
│
└── Delivery
    ├── Tests
    ├── Docker
    ├── CI/CD
    ├── Security scanning
    └── Kubernetes/Cloud deployment
```

The best standard for most Node.js applications is therefore:

> **Feature-first modular monolith + Clean Architecture boundaries + centralized infrastructure + strict TypeScript + automated testing and deployment.**

[1]: https://nodejs.org/en/about/previous-releases?utm_source=chatgpt.com "Node.js Releases"
[2]: https://expressjs.com/en/advanced/healthcheck-graceful-shutdown/?utm_source=chatgpt.com "Health Checks and Graceful Shutdown · Express.js"
[3]: https://nodejs.org/learn/command-line/how-to-read-environment-variables-from-nodejs?utm_source=chatgpt.com "How to read environment variables from Node.js"
[4]: https://www.typescriptlang.org/tsconfig/module?utm_source=chatgpt.com "TSConfig Option: module"
[5]: https://nodejs.org/download/release/latest-jod/docs/api/test.html?utm_source=chatgpt.com "Test runner | Node.js v22.23.1 Documentation"
[6]: https://expressjs.com/en/advanced/best-practice-performance/?utm_source=chatgpt.com "Production best practices: performance and reliability · Express.js"

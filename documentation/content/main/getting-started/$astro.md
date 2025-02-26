---
title: "Getting started in Astro"
description: "Learn how to set up Lucia in your Astro project"
---

We recommend using Astro v2.6+. Install Lucia using your package manager of your choice.

```
npm i lucia
pnpm add lucia
yarn add lucia
```

## Initialize Lucia

Import [`lucia()`](/reference/lucia/modules/main#lucia) from `lucia` and initialize it in its own module (file). Export `auth` and its type as `Auth`. Make sure to pass the `astro()` middleware. We also need to provide an `adapter` but since it'll be specific to the database you're using, we'll cover that in the next section.

```ts
// src/lib/lucia.ts
import { lucia } from "lucia";
import { astro } from "lucia/middleware";

// expect error (see next section)
export const auth = lucia({
	env: import.meta.env.DEV ? "DEV" : "PROD",
	middleware: astro()
});

export type Auth = typeof auth;
```

## Setup your database

Lucia uses adapters to connect to your database. We provide official adapters for a wide range of database options, but you can always [create your own](/reference/database-adapter). The schema and usage are described in each adapter's documentation. The example below is for the Prisma adapter.

```ts
import { lucia } from "lucia";
import { astro } from "lucia/middleware";
import { prisma } from "@lucia-auth/adapter-prisma";
import { PrismaClient } from "@prisma/client";

const client = new PrismaClient();

const auth = lucia({
	env: import.meta.env.DEV ? "DEV" : "PROD",
	middleware: astro(),
	adapter: prisma(client)
});
```

### Adapters for database drivers and ORMs

- [`better-sqlite3`](/database-adapters/better-sqlite3): SQLite
- [libSQL](/database-adapters/libsql): libSQL (Turso)
- [Mongoose](/database-adapters/mongoose): MongoDB
- [`mysql2`](/database-adapters/mysql2): MySQL
- [`pg`](/database-adapters/pg): PostgreSQL (including `@neondatabase/serverless`, `@vercel/postgres`)
- [`postgres`](/database-adapters/postgres): PostgreSQL
- [Prisma](/database-adapters/prisma): MongoDB, MySQL, PostgreSQL, SQLite
- [Redis](/database-adapters/redis): Redis
- [Unstorage](/database-adapters/unstorage): Azure, Cloudflare KV, Memory, MongoDB, Planetscale, Redis, Vercel KV

### Provider specific adapters

- [Cloudflare D1](/database-adapters/cloudflare-d1)
- [PlanetScale serverless](/database-adapters/planetscale-serverless)
- [Upstash Redis](/database-adapters/upstash-redis)

### Using query builders

- [Drizzle ORM](/guidebook/drizzle-orm)
- [Kysely](/guidebook/kysely)

## Set up types

In your `src/app.d.ts` file, declare a `Lucia` namespace. The import path for `Auth` is where you initialized `lucia()`.

```ts
// src/env.d.ts
/// <reference types="lucia" />
declare namespace Lucia {
	type Auth = import("./lib/lucia").Auth;
	type DatabaseUserAttributes = {};
	type DatabaseSessionAttributes = {};
}
```

## Set up middleware

This is optional but highly recommended. Create a new middleware that stores [`Auth`](/reference/lucia/interfaces/authrequest) to `locals.auth`.

```ts
// src/middleware.ts
import { auth } from "$lib/server/lucia";

import type { MiddlewareResponseHandler } from "astro";

export const onRequest: MiddlewareResponseHandler = async (context, next) => {
	context.locals.auth = auth.handleRequest(context);
	return await next();
};
```

Make sure to type `Locals` as well:

```ts
// src/env.d.ts
/// <reference types="lucia" />
declare namespace Lucia {
	// ...
}

/// <reference types="astro/client" />
declare namespace App {
	interface Locals {
		auth: import("lucia").AuthRequest;
	}
}
```

This allows us to share and access the same `AuthRequest` instance across multiple load times, which [results in better load times when validating requests](/basics/using-cookies#caching).

## Next steps

You can learn all the concepts and general APIs of Lucia by reading the [Basics](/basics/database) section in the docs. If you prefer writing code immediately, check out the [Starter guides](/starter-guides) page or the [examples in the repository](https://github.com/lucia-auth/lucia/tree/main/examples).

Remember to check out the [Guidebook](/guidebook) for tutorials and guides! If you have any questions, join our [Discord server](/discord)!

## Limitations

### Cloudflare

Please note that password hashing will not work on Free Bundled Workers; **the allocated 10ms CPU time is not sufficient for this**. Consider using unbound workers or paid bundled workers for hashing operations. This is not an issue when using OAuth.

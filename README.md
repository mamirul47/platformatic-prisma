# Platformatic Prisma

this repo is fork from the original https://github.com/kgajera/platformatic-prisma

to repo created to update the platformatic version 2.30.X and using prisma 6.X.X

[![npm version](https://badge.fury.io/js/platformaticdb-prisma.svg)](https://www.npmjs.com/package/platformaticdb-prisma)

Remove the friction to create migrations for your [Platformatic DB](https://oss.platformatic.dev/docs/reference/db/introduction) by generating them using a [Prisma schema](https://www.prisma.io/docs/concepts/components/prisma-schema).

- Use Prisma schema to manage your data models
- Generate Platformatic DB compatible up and down [migrations](https://oss.platformatic.dev/docs/reference/db/migrations) when changes are made to your Prisma schema
- Migrations will be [run](https://oss.platformatic.dev/docs/reference/db/migrations#how-to-run-migrations) by Platformatic DB

View the [example project](./example) to see it in action.

## Getting Started

Install `platformatic-prisma` and `prisma`:

```
npm install prisma platformaticdb-prisma -D
```

Create a `./prisma/schema.prisma` file with the following contents:

```prisma
datasource db {
  // Any provider supported by Platformatic
  // https://oss.platformatic.dev/docs/reference/db/introduction#supported-databases
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Table used by Platformatic/Postgrator to manage migrations
model Version {
  version BigInt    @id
  name    String?
  md5     String?
  run_at  DateTime? @db.Timestamptz(6)

  @@map("versions")
  @@ignore
}
```

Create an [`.env` file](https://www.prisma.io/docs/guides/development-environment/environment-variables#using-env-files) containing the database connection URL to connect to your database:

```
DATABASE_URL="postgresql://postgres:@localhost:5432/platformatic-prisma?schema=public"
```

## Generating Migrations

> **Note**
> If you are just starting with a new Platformatic DB project, run `npx platformatic db migrate` before generating your first migration. This is needed to allow Platformatic DB to initialize the database with it's migrations table.

When changes are made to your `schema.prisma` file, run `npx platformaticdb-prisma` to generate migrations. This will generate up and down migration files in the migrations directory that is specified in your Platformatic [configuration file](https://oss.platformatic.dev/docs/reference/db/configuration#configuration-file).

To run migrations, refer to [Platformatic's documentation](https://oss.platformatic.dev/docs/reference/db/migrations).

### Options

| Option          | Required | Description                                                       | Default                |
| --------------- | -------- | ----------------------------------------------------------------- | ---------------------- |
| `--description` | No       | Label to include in migration filename. Must not contain periods. |                        |
| `--no-down`     | No       | Prevents generation of the down migration file.                   | `false`                |
| `--no-up`       | No       | Prevents generation of the up migration file.                     | `false`                |
| `--schema`      | No       | Specifies the path to the `schema.prisma` file.                   | `prisma/schema.prisma` |


You also can create a script in package.json

```json
 "scripts": {
    "test": "npm run build:ts && tsc -p test/tsconfig.json && c8 node --test -r ts-node/register \"test/**/*.ts\"",
    "start": "npm run build:ts && fastify start -l info dist/app.js",
    "build:ts": "tsc",
    "watch:ts": "tsc -w",
    "dev": "npm run build:ts && concurrently -k -p \"[{name}]\" -n \"TypeScript,App\" -c \"yellow.bold,cyan.bold\" \"npm:watch:ts\" \"npm:dev:start\"",
    "dev:start": "fastify start --ignore-watch=.ts$ -w -l info -P dist/app.js",
    "pdb": "platformatic start",
    "pdbb": "platformatic compile",
    "pdbc": "rm -fr ./dist",
    "migrate": "platformaticdb-prisma"
  },

```

to import a platformatic db to your current fastify v5 project just add this file 

```json
{
    "$schema": "https://schemas.platformatic.dev/@platformatic/db/2.30.1.json",
    "db": {
      "connectionString": "{PLT_DATABASE_URL}",
      "graphql": true,
      "openapi": true,
      "schemalock": true
    },
    "migrations": {
      "dir": "migrations",
      "autoApply": "{PLT_APPLY_MIGRATIONS}"
    },
    "server": {
      "hostname": "{PLT_SERVER_HOSTNAME}",
      "port": "{PORT}"
    },
    "plugins": {
      "paths": [
        {
          "path": "./src/plugins",
          "encapsulate": false
        },
        {
          "path": "./src/routes"
        }
      ],
      "typescript": "{PLT_TYPESCRIPT}"
    },
    "types": {
      "autogenerate": true
    }
  }

```

then run this to install all dependency for enable platformatic db to your fastify v5 project

```bash
npm i platformatic@latest
npm i -D @platformatic/db
```
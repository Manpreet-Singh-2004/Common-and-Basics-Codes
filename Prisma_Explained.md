# Prisma

# Initialization

Install using

```bash
npm i prisma
```
and install client side as well

```bash
npm i @prisma/client
```

Init by

```bash
npx prisma init --datasource-provider postgresql
```

This is to be used if you are using postgreSQL

## Migration and Generation

In your Package.json under scripts add this

```json
    "prismagen": "npx prisma migrate dev && npx prisma generate"
```

then run `npm run prismagen` this command will execute the following commands

### Migrate (Making SQL)

```bash
npx prisma migrate dev --name init
```
Since this is the first migration it is named init and connects to the database.

Sync prisma schema with Database

### Generate new SQL

```bash
npx prisma generate
```

# schema.prisma

```ts
generator client {
  provider = "prisma-client"
}

datasource db {
  provider = "postgresql"
}

model User{
  id Int @id @default(autoincrement())
  name String
}
```
This file is auto-generated on init.

*Note: Prisma now uses prisma.config.ts file and in the datasource it takes the url from the env file, also prisma is going back to its default `provider = "prisma-client"` instead of prisma-client-js*
```ts
import { defineConfig, env } from "prisma/config";

// Rest of the code

  datasource: {
    url: env("DATABASE_URL"),
  },
```

It is mostly used with Neon DB, which uses PostgreSQL internally. Check the following resources for more information:

[Use with Neon DB](https://www.prisma.io/docs/orm/overview/databases/neon)

[New Rust-free compiler](https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/no-rust-engine#3-install-the-driver-adapter)

```ts
generator client { 
    provider = "prisma-client" 
    output = "../src/generated" 
}
```

Here i have used the default `prisma-client-js`

# Script

Since prisma is TS safe, make a script.ts

```ts
import { PrismaClient } from "@prisma/client"

const prisma = new PrismaClient()

async function main(){
    // const user = await prisma.user.create({data: {name: "Sally"}})
    // console.log(user)

    const users = await prisma.user.findMany()
    console.log(users)
}

main()
    .catch(e => {
        console.error(e.message)
    })
    .finally(async () =>{
        await prisma.$disconnect()
    })

```

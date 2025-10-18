# NextJS

# --> Todo Project

# Initialize

Install using

```bash
npx create-next-app@latest [project-name] [options]
```

## Troubleshooting

### Vercel Deployment
For deployment on **Vercel** you would get an error if you have seperately installed Oxide and Lightningcss in `Package.json`. That is because these dependencies are win32 x64 specific, and the vercel servers operate on Linux, hence to fix it we will shift those dependenceies to *optionalDependencies* this will basically skip the installation on non supported OS.

```bash
  "optionalDependencies": {
    "@tailwindcss/oxide-win32-x64-msvc": "latest",
    "lightningcss-win32-x64-msvc": "latest"
  }
```

# .env.local

```jsx
MONGO_URI=

NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
CLERK_WEBHOOK_SIGNING_SECRET=

```

# Database

## Mongo

In the MONGO_URI, after the port, add in your database name
mongodb://username:password@host:port/**databaseName**?authSource=admin

### Connect to Database

To connect to Database, we will create a new folder in the root of the project with the name `lib` in it we will create a file named `DBConnect.js`

```js
import mongoose from "mongoose";

const MONGO_URI = process.env.MONGO_URI;

export const DBConnect = async() =>{
    if(!MONGO_URI) throw new Error("MONGO_URI is not present in the env file");
    
    const readyState = mongoose.connection.readyState;
    if(readyState === 1){
        console.log("DB is connected");
        return;
    }
    try{
        await mongoose.connect(MONGO_URI);
        console.log("DB connected");
    } catch(error){
        console.log(`Mongo connection failed, error: ${error}`);
    }
}
```

In Mongoose, mongoose.connection.readyState is a property that tells you the current status of the connection to your database. It's not a boolean (true/false), but a number that represents a specific state.

The possible values are:

0: disconnected

1: connected

2: connecting

3: disconnecting

So, the line if (readyState === 1) is a very precise check that asks, "Is the database connection already successfully established and open?"

### Model

Create a new folder with the name `Models` then create a new file with the name `Todo.model.js` this is our Object data

```js
import mongoose from "mongoose";

const TodoSchema = new mongoose.Schema({
    done:{
        type: Boolean,
        default: false
    },
    content:{
        type: String,
        require: true
    },
    createdBy:{
        type: mongoose.type.ObjectId,
        ref: "User"
    }
}, {timestamps: true});

const Todo = mongoose.models.Todo || mongoose.model("Todo", TodoSchema);
export default Todo;
```
For the users, we will create another file with the name `User.model.js`

```js
import mongoose from "mongoose";

const UserSchema = new mongoose.Schema({
    clerkId:{
        type: String,
        required: true,
        unique: true
    },
    name:{
        type: String,
        required: true
    },
    email:{
        type: String,
        required: true
    },
    Todos:[{
        type: mongoose.Types.ObjectId,
        ref: "Todo",
        default: []
    }]
});

const User = mongoose.models.User || mongoose.model("User", UserSchema);
export default User;
```

to test out the DB, just add the code in a page.tsx and you will see the connection

```js
  await DBConnect();
```

### Webhooks

To send data from Auth to your database you need a webhook. Create a folder inside your app folder. Path -: `app/api/webhooks`  and create a file named `route.ts`

**Why do you need webhook?** Because clerk or any other Auth service will store the user data on its end, but we need the data on our database as well. This is where webhooks come in handy, they send the data from the Auth service to the endpoint.

Firstly you would need to deploy your project in order to get the `CLERK_WEBHOOK_SIGNING_SECRET=` from clerk. Before you deploy to vercel, just use any random string for this webhook signing secret. Then after deploying, copy paste the live link to clerk and you will get the signing secret. Then again go back to vercel and replace the random string with the correct secret.

After doing that, check the logs of your deployed app

```ts
// app/api/webhooks/route.ts

import { verifyWebhook } from '@clerk/nextjs/webhooks'
import { NextRequest } from 'next/server'
import { DBConnect } from '@/lib/DBConnect';
import User from '@/Models/User.model';

export async function POST(req: NextRequest) {
  try {
    const evt = await verifyWebhook(req)

    if (evt.type === 'user.created') {

      try{
        await DBConnect();
        console.log("Webhook Recieved: Creating user in MongoDB...")

        const {id, first_name, last_name} = evt.data;

        const existingUser = await User.findOne({clerkId: id});
        if(existingUser){
          console.log("User already exists in the database.");
          return new Response('User already exists', {status: 200});
        }

        await User.create({
          clerkId: id,
          name: `${first_name} ${last_name || ''}`.trim(),
          email: evt.data.email_addresses[0].email_address,
        });

        console.log("User created successfully in MongoDB.");
        return new Response('user Created', {status: 201});

      } catch(error){
        console.error("Error handling user.create webhook: ", error)
        return new Response("Error Occured during user creation in DB", {status: 500})
      }
}

if(evt.type==='user.deleted'){
  try{
    await DBConnect();
    console.log("Webhook recieved: Deleting user in MongoDB...");

    const {id} = evt.data;
    await User.findOneAndDelete({clerkId: id});
    console.log("User successfully deleted from MongoDB.")
    return new Response('User Deleted', {status: 200});

  } catch(error){
    console.log("Error handling user.delete webhook: ", error)
    return new Response("Error occured during user deletion in DB", {status: 500});
  }
}

  return new Response('Webhook received', { status: 200 })
  } catch (err) {
    console.error('Error verifying webhook:', err)
    return new Response('Error verifying webhook', { status: 400 })
  }
}
```

Since we are using clerk for this project, we need to go to clerk dashboard > Configure > webhooks


# Auth


## Clerk

### Middleware.ts

In the root of the project we will make a `middleware.ts`. Here we will be 

```ts
// @/middleware.ts
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server'

const isPublicRoute = createRouteMatcher(['/sign-in(.*)', '/sign-up(.*)',"/api/webhooks(.*)"])

export default clerkMiddleware(async (auth, req) => {
  if (!isPublicRoute(req)) {
    await auth.protect()
  }
})

export const config = {
  matcher: [
    // Skip Next.js internals and all static files, unless found in search params
    '/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)',
    // Always run for API routes
    '/(api|trpc)(.*)',
  ],
}
```
in the `isPublicRoute` add any routes that you **Want to be Open/Non-Protected**. Webhooks has to be in non-protected otherwise it won't be able to send the data back to your database.

### Sign in
Create a new folder in app folder with the name `sign-in/[[...sign-in]]` then make a `page.js` with the following code.
This is our dedicated sign-in page

```js
import { SignIn } from "@clerk/nextjs";

export default function Page() { 
    <div style={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '100vh' }}>
        <SignIn />
    </div>
 };
```
Check out `layout.tsx` to see how the protected routes are being used

# Layout.tsx
In our app/layout.tsx

```tsx
import { type Metadata } from 'next'
import {
  ClerkProvider,
  SignInButton,
  SignUpButton,
  SignedIn,
  SignedOut,
  UserButton,
} from '@clerk/nextjs'
import { Geist, Geist_Mono } from 'next/font/google'
import './globals.css';
import Navbar from '../components/Navbar';
import { ThemeProvider } from '@/components/ui/theme-provider'

const geistSans = Geist({
  variable: '--font-geist-sans',
  subsets: ['latin'],
})

const geistMono = Geist_Mono({
  variable: '--font-geist-mono',
  subsets: ['latin'],
})

export const metadata: Metadata = {
  title: 'Clerk Next.js Quickstart',
  description: 'Generated by create next app',
}

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode
}>) {
  return (
    <ClerkProvider>
      <html lang="en">
        <body className={`${geistSans.variable} ${geistMono.variable} antialiased`}>
          
          <ThemeProvider
            attribute="class"
            defaultTheme="system"
            enableSystem
            disableTransitionOnChange
          >
          <Navbar />
          {children}
          </ThemeProvider>
        </body>
      </html>
    </ClerkProvider>
  )
}
```
Here as we can see `ClerkProvider` wraps around the whole layout, meaning it will protect the whole page.
`Navbar` is also used at the top by default.

# Components
Now for the components, we will create a new folder named `components` in the root of the folder.

## theme-provider.jsx

Just install shadcn button and in the `ui` folder auto created by shadcn in components create another file with the name `theme-provider.jsx`

```jsx
// components/ui/theme-provider.jsx
"use client"

import * as React from "react"
import { ThemeProvider as NextThemesProvider } from "next-themes"

export function ThemeProvider({
  children,
  ...props
}){
  return <NextThemesProvider {...props}>{children}</NextThemesProvider>
}
```
Back in layout.tsx we can see how theme provider is covering both navbar and the {children}.

## ModeToggle.jsx

Then in components add a file called `ModeToggle.jsx`

```jsx
// components/ModeToggle.jsx
"use client"

import * as React from "react"
import { Moon, Sun } from "lucide-react"
import { useTheme } from "next-themes"

import { Button } from "@/components/ui/button"
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from "@/components/ui/dropdown-menu"

export function ModeToggle() {
  const { setTheme } = useTheme()

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="outline" size="icon">
          <Sun className="h-[1.2rem] w-[1.2rem] scale-100 rotate-0 transition-all dark:scale-0 dark:-rotate-90" />
          <Moon className="absolute h-[1.2rem] w-[1.2rem] scale-0 rotate-90 transition-all dark:scale-100 dark:rotate-0" />
          <span className="sr-only">Toggle theme</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end">
        <DropdownMenuItem onClick={() => setTheme("light")}>
          Light
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme("dark")}>
          Dark
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme("system")}>
          System
        </DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  )
}
```
## Navbar.jsx

This is also used in the layout.jsx just above {children} which means it will be on every page/

```jsx
import React from 'react';
import {
    SignedIn,
    SignedOut,
    UserButton,
} from '@clerk/nextjs'
import {ModeToggle} from './ModeToggle'
import Link from 'next/link'
import {NotebookPen} from 'lucide-react'
import {Button} from './ui/button'


const Navbar = ()=>{
    return(
        <div className='flex items-center justify-between'>
            <Link href={'/'} className='ml-10 flex gap-2'>
            <p><NotebookPen /></p>
            <p className='font-bold'>Notes</p>
            </Link>
            <div className='flex items-center py-7 px-5 justify-end gap-5'>
                <ModeToggle />
                <SignedOut>
                    <Link href={'/sign-in'}>
                        <Button>
                            Get Started
                        </Button>
                    </Link>
                </SignedOut>
                <SignedIn>
                    <UserButton />
                </SignedIn>
            </div>
        </div>
    )
}

export default Navbar
```
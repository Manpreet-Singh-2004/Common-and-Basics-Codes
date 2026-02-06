# NextJS

- [NextJS](#nextjs)
- [--> Todo Project](#-todo-project)
- [Initialize](#initialize)
   * [Troubleshooting](#troubleshooting)
      + [Vercel Deployment](#vercel-deployment)
- [.env.local](#envlocal)
- [Database](#database)
   * [Mongo](#mongo)
      + [Connect to Database](#connect-to-database)
      + [Model](#model)
      + [Webhooks](#webhooks)
- [Auth](#auth)
   * [Clerk](#clerk)
      + [Middleware.ts](#middlewarets)
      + [Sign in](#sign-in)
- [Layout.tsx](#layouttsx)
- [Actions](#actions)
- [Components](#components)
   * [theme-provider.jsx](#theme-providerjsx)
   * [ModeToggle.jsx](#modetogglejsx)
   * [Navbar.jsx](#navbarjsx)
   * [TodoUI](#todoui)
   * [EditFormMenu.jsx](#editformmenujsx)
   * [AddForm.jsx](#addformjsx)
   * [AddFloatingButton.jsx](#addfloatingbuttonjsx)
- [Page.tsx](#pagetsx)


# --> Todo Project

# Initialize

Install using

```bash
npx create-next-app@latest [project-name] [options]
```

## Troubleshooting

### Vercel Deployment
For deployment on **Vercel** you would get an error if you have seperately installed Oxide and Lightningcss in `Package.json`. That is because these dependencies are win32 x64 specific, and the vercel servers operate on Linux, hence to fix it we will shift those dependenceies to *optionalDependencies* this will basically skip the installation on non supported OS.

```json
"dependencies":{
  "@rollup/rollup-win32-x64-msvc": "^4.57.1",
},
"devDependencies":{
  "@tailwindcss/oxide-win32-x64-msvc": "latest",
  "lightningcss-win32-x64-msvc": "latest"
}
```
Then after installation move them to optional dependencies -:
```json
"optionalDependencies":{
  "@tailwindcss/oxide-win32-x64-msvc": "latest",
  "lightningcss-win32-x64-msvc": "latest"
}
```

To install the dependencies use the command -:
```bash
npm i lightningcss-win32-x64-msvc @tailwindcss/oxide-win32-x64-msvc --save-dev --force
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


Go to https://clerk.com/docs/guides/development/webhooks/syncing

To get your 

NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY

CLERK_SECRET_KEY

CLERK_WEBHOOK_SIGNING_SECRET

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

You can also add `username` field in clerk, so that on a new sign up clerk will always ask for a unique username and using webhooks you can send it to your database

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
  return(
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

# Actions

Make a folder with the name `actions` in it create a file with the name `User.actions.js` the path will be `app/actions/User.actions.js` 

This will have all the actions that a user can do with the server.

```js
"use server"

import { DBConnect } from "@/lib/DBConnect";
import Todo from "../../Models/Todo.model";
import User from "../../Models/User.model";
import { auth } from "@clerk/nextjs/server";
import { revalidatePath } from "next/cache";

export const addTodo = async(formData) =>{
    try{
        await DBConnect();
        const content = formData.get('content');
        if(!content){
            return {success: false, message:"Todo was empty"};
        }
        const CurrentUser = await auth();
        const FoundUser = await User.findOne({clerkId: CurrentUser.userId})
        if(!FoundUser){
            return console.log("User dosen't exists");
        }
        const CreatedTodo = await Todo.create({
            content: content,
            createdBy: FoundUser._id
        })
        FoundUser.Todos.push(CreatedTodo._id);
        await FoundUser.save();
        return {success: true, message: "Todo Created"}
    } catch(error){
        console.log(error);
    }
}

export const getTodos = async() =>{
    try{
        await DBConnect();
        const CurrentUser = await auth();
        const Todos = await User.findOne({clerkId:CurrentUser.userId}).populate('Todos');
        return Todos;
    } catch(error){
        console.log(error);
    }
}

export const CheckTodo = async(todoID)=>{
    try{
        await DBConnect();
        const FoundTodo = await Todo.findById({_id: todoID});
        FoundTodo.done = !(FoundTodo.done);
        await FoundTodo.save();
        revalidatePath('/');
    } catch(error){
        console.log(error);
    }
}

export const EditTodo = async(todoID, formData)=>{
    try{
        await DBConnect();
        const Newcontent = formData.get('content');
        await Todo.findByIdAndUpdate({_id: todoID}, {content: Newcontent});
        revalidatePath('/');
    } catch(error){
        console.log(error)
    }
}

export const DeleteTodo = async(todoID)=>{
    try {
        await DBConnect();
        await Todo.findByIdAndDelete({_id: todoID});
        revalidatePath('/');
        return {success: true, message:"Todo deleted"}
    } catch (err) {
        console.log(err);
    }
}

export const getSingleTodo = async(todoID)=>{
    try{
        await DBConnect();
        const FoundTodo = await Todo.findById({_id: todoID});
        return {success: true, Todo: FoundTodo};
    } catch(error){
        console.log(error);
    }
}
```

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

## TodoUI
make a file with the name TodoUI.jsx

```jsx
"use client"

import { CheckTodo, DeleteTodo } from '@/app/actions/User.actions'
import { Button } from '@/components/ui/button'
import { Checkbox } from '@/components/ui/checkbox'
import { Trash2 } from 'lucide-react'
import { toast } from 'sonner'
import EditformMenu from './EditformMenu'

const TodoUi = ({date, content, done, id})=>{
    const formattedDate = new Date(date).toLocaleDateString("en-US", {
        weekday: "long",
        month: "short",
        day: "numeric",
    });

    return(
        <div className='flex flex-col p-4'>
            <div className='bg-white dark:bg-slate-950 w-full max-w-lg rounded-xl shadow-xl border border-slate-200 dark:border-slate-700'>
                <div className='py-4 px-4 text-sm text-slate-700 dark:text-slate-300 border-b-2 border-dashed border-slate-300 dark:border-slate-600 flex items-center justify-between'>
                    <span className='font-medium'>{formattedDate}</span>
                    <div className='flex items-center gap-3'>
                        <Checkbox className='data-[state=checked]:bg-slate-600 data-[state=checked]:border-slate-600' onCheckedChange={() =>{
                            CheckTodo(id);
                        }}/>
                        <Button onClick = {async() =>{
                                const res = await DeleteTodo(id);
                                if(res.success) return toast.success(res.message);
                                toast.error('Something went wrong')
                            }}
                        variant="ghost"
                        size="sm"
                        className="h-8 w-8 p-0 text-slate-500 dark:text-slate-400 hover:text-red-500 dark:hover:text-red-400 hover:bg-slate-100 dark:hover:bg-slate-700 transition-colors"
                        >
                            <Trash2 className='h-4 w-4' />
                        </Button>
                        <EditformMenu todoId={id} content={content} />
                    </div>
                </div>
                <div className='py-4 px-4'>
                    <div className={`text-slate-800 dark:text-slate-100 text-base leading-relaxed min-h-[100px] ${done ? "line-through text-gray-50 dark:text-slate-500":""}`}>
                        {content}
                    </div>
                </div>
            </div>
        </div>
    )
}

export default TodoUi
```

## EditFormMenu.jsx

Make a file with the name EditformMenu.jsx

```jsx
import React from 'react'
import { Button } from './ui/button'
import {
  Dialog,
  DialogContent,
  DialogFooter,
  DialogHeader,
  DialogTrigger,
} from "@/components/ui/dialog"
import { DialogClose, DialogTitle } from '@radix-ui/react-dialog'
import { Input } from './ui/input'
import { SquarePen } from 'lucide-react'
import { EditTodo } from '@/app/actions/User.actions'

const EditformMenu = ({todoId, content}) =>{
    const handleSubmit = async(formData)=>{
        await EditTodo(todoId, formData);
    }
    return(
        <div>
            <Dialog>
                <DialogTrigger asChild>
                    <Button
                        variant="ghost"
                        size="sm"
                        className='h-8 w-8 p-0 text-slate-500 dark:text-slate-400 hover:text-green-500 dark:hover:text-green-400 hover:bg-slate-100 dark:hover:bg-slate-700 transition-colors'>
                        <SquarePen className='h-4 w-4' />
                    </Button>
                </DialogTrigger>
                <DialogContent className="sm:max-w-[425px]">
                    <form action={handleSubmit}>
                        <DialogHeader>
                            <DialogTitle>Edit Todo</DialogTitle>
                        </DialogHeader>
                        <div className='grid gap-4'>
                            <div className='grid gap-3'>
                                <Input id='content' name='content' defaultValue={content} className='my-5' />
                            </div>
                        </div>
                        <DialogFooter>
                            <DialogClose asChild>
                                <Button variant="outline">Cancel</Button>
                            </DialogClose>
                            <DialogClose asChild>
                                <Button type='submit' className='ml-2'>Add</Button>
                            </DialogClose>
                        </DialogFooter>
                    </form>
                </DialogContent>
            </Dialog>
        </div>
    )
}

export default EditformMenu
```

## AddForm.jsx

Make a file with the name AddForm.jsx

```jsx
"use client"
import React from "react"
import {
  DialogClose,
  DialogFooter,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog";
import { Button } from './ui/button';
import { Input } from './ui/input';
import { addTodo } from '@/app/actions/User.actions';
import { toast } from 'sonner';
import { useRouter } from 'next/navigation';

const AddForm = ()=>{
    const router = useRouter();

    const handleSubmit = async(formData)=>{
        const result = await addTodo(formData);
        if(result.success){
            toast(result.message);
            router.refresh('/');
        } else{
            toast.error(result.message);
        }
    }
    return(
        <form action={handleSubmit}>
            <DialogHeader>
                <DialogTitle>Add Todo</DialogTitle>
            </DialogHeader>
            <div className="grid gap-4">
                <div className="grid gap-3">
                    <Input id="content" name="content" placeholder="Add Content..." className='my-5' />
                </div>
            </div>
            <DialogFooter>
                <DialogClose asChild>
                    <Button variant="outline">Cancle</Button>
                </DialogClose>
                <DialogClose asChild>
                    <Button type="submit" className="ml-2">Add</Button>
                </DialogClose>
            </DialogFooter>
        </form>
    )
}

export default AddForm
```

## AddFloatingButton.jsx

Make a file with the name AddFloatingButton.jsx

```jsx
import React from 'react';
import { Button } from './ui/button'
import { Plus } from 'lucide-react';
import {
  Dialog,
  DialogContent,
  DialogTrigger,
} from "@/components/ui/dialog"
import AddForm from './AddForm'

const AddFloatingButton = () =>{
    return (
    <div>
         <Dialog>
      
        <DialogTrigger asChild>
          <Button
        className="fixed bottom-6 right-6 rounded-full w-14 h-14 p-0 
                   bg-white dark:bg-black 
                   text-gray-800 dark:text-gray-200 
                   border-2 border-gray-300 dark:border-gray-600
                   hover:bg-gray-50 dark:hover:bg-gray-900
                   hover:border-gray-400 dark:hover:border-gray-500
                   shadow-lg hover:shadow-xl 
                   transition-all duration-300 ease-in-out
                   hover:scale-105 active:scale-95"
      >
        <Plus className="w-6 h-6" />
      </Button>
        </DialogTrigger>
         <DialogContent className="sm:max-w-[425px]">
            <AddForm/>
           </DialogContent>
    </Dialog>
    </div>
  )
}

export default AddFloatingButton
```

# Page.tsx

```tsx
import AddFloatingButton from '../components/AddFloatingButton'
import TodoUi from '../components/TodoUI'
import { getTodos } from './actions/User.actions';
import { Notebook } from 'lucide-react';

interface ITodo {
  _id: { toString: () => string };
  content: string;
  done: boolean;
  createdAt: Date;
}

const page = async()=>{
  const data = await getTodos();
  const Todos = data?.Todos || [];

  return(
    <div className='relative min-h-screen'>
      {Todos.length > 0 ? (
        <div className='grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 p-4'>
          {Todos.map((Todo: ITodo)=>(
            <div key={Todo._id.toString()} className='transition-all'>
              <TodoUi 
                id={Todo._id.toString()}
                date={Todo.createdAt}
                content={Todo.content}
                done={Todo.done}
              />
            </div>
          ))}
        </div>
      ) : (
        <div className='flex flex-col items-center justify-center h-[70vh] text-center text-muted-foreground p-4'>
          <Notebook className='w-16 h-16 mb-4 text-primary' />
          <h2 className="text-2xl font-semibold mb-2">No Todos Yet</h2>
          <p className="text-sm">You haven&apos;t added any tasks. Start planning your day now!</p>
        </div>
      )}
      <AddFloatingButton />
    </div>
  );
};

export default page;
```

# Next.js

- [Next.js](#nextjs)
   * [Initializing](#initializing)
      + [Common Troubleshooting](#common-troubleshooting)
      + [Missing dependencies](#missing-dependencies)
      + [Turboflex](#turboflex)
   * [Components and Layouts](#components-and-layouts)
      + [Creating a `components` folder](#creating-a-components-folder)
      + [Creating local layout.tsx](#creating-local-layouttsx)
      + [Global layout.tsx](#global-layouttsx)
   * [Routing](#routing)
   * [Folders and naming convention](#folders-and-naming-convention)
      + [Using `()`](#using-)
      + [Using `[]`](#using--1)
      + [Nested Folders](#nested-folders)
      + [loading](#loading)
   * [Error handling](#error-handling)
      + [404 Page not found](#404-page-not-found)
      + [Local Error](#local-error)
      + [Global Error](#global-error)
   * [Database](#database)
      + [Demo using Arrays](#demo-using-arrays)
   * [APIs](#apis)
      + [Demo using jsonplaceholder](#demo-using-jsonplaceholder)
   * [SEO (MetaData)](#seo-metadata)
      + [Static Metadata](#static-metadata)
      + [Dynamic Metadata](#dynamic-metadata)


## Initializing

use the following command to create next app

```bash
npx create-next-app@latest
```
To run use the command

```bash
npm run dev
```

### Common Troubleshooting

### Missing dependencies
On windows systems, next sometimes dosnt install the following dependencies, which can cause error even before you make any changes to the code, to fix it simplly install the following dependencies -: 

```bash
npm i @tailwindcss/oxide-win32-x64-msvc

npm i lightningcss-win32-x64-msvc    
```

### Turboflex
Sometimes, turboflex can also cause errors if you use it to run the next app, for this, you can try disabling the turbopack to compile your project. go to your *package.json* and in your scripts

```bash
"dev": "next dev --turbopack"
```
remove the `--turbopack` so it becomes

```bash
"dev": "next dev"
```

*Note -: Disabling turbopack will result in longer time for project to run*

## Components and Layouts

### Creating a `components` folder

Create a separate folder, with the component and import it inside your files.

```tsx
// app/components/hello.tsx
"use client";

function Hello(){
    console.log("I am a client component");
    return(
        <div>
            <h1>Hello</h1>
        </div>
    );
}

export default Hello;
```
then in (root)/page.tsx
```js
<Hello/>
```

### Creating local layout.tsx

In your working folder make a file with the name layout.tsx, for eg -: (root)/layout.tsx and all files within (root) will auto-apply that layout.

*Note -: You can't change the position of layout.tsx in the page*

```tsx
import React from 'react'

const layout = ({children}: {children: React.ReactNode}) => {
  return (
    <div>
      <h1 className='text-3xl'>Navbar</h1>
      {children}
    </div>
  )
}

export default layout

```

### Global layout.tsx

Its in the app/layout.tsx

the global layout works the same way, but it’s at the very top of the hierarchy

Here’s how Next.js renders (root)/page.tsx:

```tsx
<app/layout>                 // Global layout (HTML, BODY, global fonts, etc.)
  <(root)/layout>            // Local layout (Navbar)
    <(root)/page />          // Your actual page content
  </(root)/layout>
</app/layout>
```

## Routing
Next.js uses a file-system based routing system — meaning that the folder structure inside your app directory automatically defines your routes.

For example:

If you create a folder named about

And inside it, add a file page.tsx

Then, the page will automatically be available at:
```bash
http://localhost:3000/about
```
You don’t need to manually configure any React routes — Next.js automatically handles it for you based on the folder and file names.

## Folders and naming convention

### Using `()`
Folders wrapped in parentheses, like (root) or (dashboard), are special layout or grouping folders in Next.js.
They are not converted into routes, meaning there will not be a URL like:
```bash
http://localhost:3000/root
http://localhost:3000/dashboard
```

**Files inside these folders** can still generate routes if they are in subfolders or are page files. For example:

```bash
app/(root)/about/page.tsx    →    http://localhost:3000/about
app/(root)/page.tsx          →    http://localhost:3000/  (renders as the main page)
```
Essentially, *(folderName)* is used for organizing your app structure without affecting the URLs.

### Using `[]`
Dynamic routing in Next.js allows you to create pages that adapt to different parameters — like showing multiple user profiles using a single page template.

For example, if you have a folder structure like this:
```bash
app/(dashboard)/dashboard/users/page.tsx
```

That route (/dashboard/users) will display a static page — it can only show one view.

To display different user profiles dynamically, you need to create a dynamic route using square brackets:

```bash
app/(dashboard)/dashboard/users/[id]/page.tsx
```
Here, [id] acts as a placeholder parameter, allowing URLs like:

```bash
http://localhost:3000/dashboard/users/1
http://localhost:3000/dashboard/users/2
```
Each one will automatically render the corresponding user profile page based on the id.


```js
// app/(dashboard)/dashboard/users/[id]/page.tsx

import React from 'react'

const page = ({params} : {params: {id: string}}) => {

    const {id} = params;
    return (
        <h1 className='text-3xl'>User profile: {id}</h1>
    )
}

export default page
```

Next.js automatically passes the params object to the page component, so you can easily access and display dynamic data like id.

### Nested Folders

you can have multiple folders inside of folders, for example -:
(dashboard)/dashboard/users 
http://localhost:3000/dashboard/users

and

(dashboard)/dashboard/analytics
http://localhost:3000/dashboard/analytics

*Note -: If (dashboard)/dashboard dosent have a page.tsx, then if you go to http://localhost:3000/dashboard you will get a 404 not  found page*

### loading

Put this file at the root of the app, app/loading.tsx

used to show loading states and is displayed automatically, that means no need to import it

```js
import React from 'react'

const loading = () => {
console.log("Loading Phase")

  return (
    <div>
      Loading...
    </div>
  )
}

export default loading
```
*Optional: You can also place loading.tsx inside subfolders like app/dashboard/loading.tsx to show a section-specific loading indicator*

## Error handling

### 404 Page not found
By default Next.js has its own 404 page not found, but you can create your own 404 page not found.
you have to create the file like this app/not-found.tsx

```js
export default function NotFound() {
  return (
    <div style={{ textAlign: 'center', marginTop: '100px' }}>
      <h1>404</h1>
      <p>Oops! Page Not Found.</p>
    </div>
  );
}
```
### Local Error

You can place this file inside a specific folder, such as: *app/(root)/error.tsx*


This creates a local error boundary for that route group.
If an error occurs within any page or component inside (root),
Next.js will automatically display the message:

“Something went wrong.”

This local error boundary takes priority over the global one and only affects errors within its own route group.

```tsx
// app/(root)/error.tsx

'use client' // Error boundaries must be Client Components
 
import { useEffect } from 'react'
 
export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    // Log the error to an error reporting service
    console.error(error)
  }, [error])
 
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button
        onClick={
          // Attempt to recover by trying to re-render the segment
          () => reset()
        }
      >
        Try again
      </button>
    </div>
  )
}
```
You can use

```tsx
throw new Error('Not Implemented')
```
in page.tsx to throw an error.

### Global Error

Make a file and name it *global-error.tsx* with the following content. and error will show up.

```tsx
'use client' // Error boundaries must be Client Components
 
export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    // global-error must include html and body tags
    <html>
      <body>
        <h2>Something went wrong!</h2>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
  )
}
```
*Note -: If you have both a global error file and a route-level error file,
the closest one to the error always takes priority.*

## Database

### Demo using Arrays

for this demo we will make a folder named api

inside we will make a file named **db.ts** with the following content

```ts
const books = [
    {id: 1, name: "Atomic Habbit"},
    {id: 2, name: "Deep Work"},
    {id: 3, name: "Games of thrown"},
]

export default books;
```

then make another folder in api with the name **books** and then a route.ts which is basically your GET and POST methods

this will now be api/books/route.ts

```ts
import books from '@/app/api/db';

export async function GET(){
    return Response.json(books);
}

export async function POST(request: Request) {
    const book = await request.json();
    books.push(book);

    return Response.json(books);
}
```
then go to http://localhost:3000/api/books to check GET

to test out **POST** you can use browser console and put this

```bash
fetch("/api/books", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ id: 4, name: "Rich Dad Poor Dad" })
})
  .then(res => res.json())
  .then(data => console.log(data));
```

to test out PUT and DELETE, we would need the IDs of the documents, hence we will create a new folder named *[id]* and inside again create a file named route.ts

now our path is api/books/[id]/route.ts

```ts
import books from '@/app/api/db';

export async function PUT(
    request: Request,
    context: {params: {id: string}},
) {
    const id = +context.params.id;
    const book = await request.json();

    const index = books.findIndex((b) => b.id == id);
    books[index] = book;
    return Response.json(books);
}

export async function DELETE(
    request: Request, 
    context: {params: {id: string}},
) {
    const id = +context.params.id;
    const index = books.findIndex((b) => b.id == id);
    books.splice(index, 1);
    return Response.json(books);
}
```
you don't need to change the link, just stay at the same URL http://localhost:3000/api/books and reload after making change.

In browser use the following commands to test PUT
``` bash
fetch('/api/books/2', {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ id: 2, name: 'Deep Work (Updated Edition)' })
})
  .then(res => res.json())
  .then(data => console.log(data));

```

and Delete
```bash
fetch('/api/books/3', {
  method: 'DELETE'
})
  .then(res => res.json())
  .then(data => console.log(data));
```
*Note -: The Change isnt permanent since we are its only happening at run time and the changes arent made to files or DB.*

## APIs

### Demo using jsonplaceholder

for this we are using https://jsonplaceholder.typicode.com/albums API.

making the function async to recieve data, then mapping it out and displaying its title and ID

```tsx
export default async function Home(){

  const response = await fetch("https://jsonplaceholder.typicode.com/albums");

  if(!response.ok) throw new Error("Failed to fetch data");

  const albums = await response.json();

  return(

    <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols">

      {albums.map((album: {id: number, title: string}) =>(
        <div key={album.id}>
          <h3>{album.title}</h3>
          <p>{album.id}</p>
        </div>
      ))}

    </div>
  )
}
```


## SEO (MetaData)

### Static Metadata
```js
export const metadata = {
    title: "Home | Next.js",
    description: "Generated by create next app",
    openGraph: {
        ...openGraphImage,
        title: 'Home',
    }
};
```
Make sure the static is used in the highest priority item, otherwise it will overwrite the default specified in the files. Hence for static its recomended to provide unique metadata for each route or rely on the metadata from the root component.

### Dynamic Metadata
```js
export async function generateMetadata({ params }){
    const {id} = params;
    const resource = await getResourceById({id});

    const title = resource.title + " | JS Mastery";
    const seoDescription = "Description...";
    
    return{
        title,
        description: seoDescription,
        other: {
            "og:title": title,
            "og:description": seoDescription,
            "og:image": resource.image,
        }
    }
}
```


## Auth

https://authjs.dev/getting-started/migrating-to-v5

OAuth providers

https://authjs.dev/getting-started/authentication/oauth


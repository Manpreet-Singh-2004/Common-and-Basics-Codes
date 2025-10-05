# Firebase

- [Firebase](#firebase)
   * [Initialization](#initialization)
   * [firebase-config.js OR config/firebase.js](#firebase-configjs-or-configfirebasejs)
   * [Authentication](#authentication)
   * [Database](#database)
      + [Query](#query)
         - [Comparision](#comparision)
         - [Ascending/Descending](#ascendingdescending)
      + [App.js](#appjs)
         - [Get](#get)
         - [Delete](#delete)
         - [Update](#update)
         - [Post](#post)
   * [Additional Notes](#additional-notes)

## Initialization
to make a project
```bash
firebase init hosting
```
to test locally
```bash
firebase server
```
to get a live working Link, but before that build your app
for example if using CRA, use the command

```bash
npm run build
```

then deploy using

```bash
firebase deploy
```

to check if the Firebase is connected you can do
```js
document.addEventListener("DOMContentLoaded", event  =>{
	const  app  =  firebase.app();
	console.log(app)
})
```

## firebase-config.js OR config/firebase.js

```js
import { initializeApp } from "firebase/app";
import {getAuth, GoogleAuthProvider} from 'firebase/auth'
import {getFirestore} from 'firebase/firestore'

const firebaseConfig = {
  apiKey: "",
  authDomain: "",
  projectId: "",
  storageBucket: "",
  messagingSenderId: "",
  appId: ""
};


const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const googleProvider = new GoogleAuthProvider();
export const db = getFirestore(app);
```
Here we are mostly using `getAuth` and since we are using google as the OAuth provide, `GoogleAuthProvider`.
For database we are using **firestore** hence `getFirestore`

## Authentication

```js
import { useState } from 'react';
import {auth, googleProvider} from '../config/firebase';
import {createUserWithEmailAndPassword, signInWithPopup, signOut} from 'firebase/auth';

export const Auth = () => {

    const [email, setEmail] = useState("");
    const [password, setPassword] = useState("");
    const [error, SetError] = useState("");

    // console.log(auth?.currentUser?.email) // Used to console log the current user
    // console.log(auth?.currentUser?.photoURL) // Used to console log the current user

    const signIn = async() =>{
        try{
            await createUserWithEmailAndPassword(auth, email, password);
            SetError("")

        } catch(e){
            console.log(e)
            SetError(e.message)
        }
    };

    const signInWithGoogle = async() =>{
        try{
            await signInWithPopup(auth, googleProvider);
            SetError("")
        }catch(e){
            console.log(e)
            SetError(e.message)
        }
    }

    const logout = async() =>{
        try{
            const signedOutUser = auth?.currentUser?.email
            await signOut(auth)
            console.log(`${signedOutUser} User Logged out Successfully`)
        }catch(e){
            console.log(e)
            SetError(e.message)
        }
    }

    return (
    <div>
        <input placeholder="Email" 
        onChange={(e) => setEmail(e.target.value)}
         />

        <input placeholder="Password" 
        type='password'
        onChange={(e) => setPassword(e.target.value)} 
        />

        <button onClick={signIn}>Sign in</button>

        <button onClick={signInWithGoogle}>Sign in with google</button>

        <button onClick={logout}>Logout</button>

        {error && <h2 style={{ color: "red" }}>{error}</h2>}

        <h1>Welcome {auth?.currentUser?.email}</h1>

    </div>
)}


// Rules for firestore database | Anyone can delete anything
/*
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      
      allow read: if true;

      allow create: if request.auth != null && request.auth.uid == request.resource.data.userId;
      allow update, delete: if request.auth != null;
    }
  }
}
*/
```

Exporting Auth compnent.

`auth?.currentUser?.email` is used to display email. The **?** is used to handle cases if there isnt any auth user. otherwise if userisnt there that would give undefined and hence crash the app.

`auth?.currentUser?.photoURL` is used to display the photo (if any)

`createUserWithEmailAndPassword` is used to enter custom user email and password.

`signInWithPopup` opens a pop-up and passes auth token.

logout is handled mainly by `signOut(auth)`

## Database
For this we are using firestone

Collection Name -: `posts`

Post ID -: `firstpost`

title of `firstpost`-: "My first Post"



```js
const  db  =  firebase.firestore();
const  myPost  =  db.collection('posts').doc('firstpost');

myPost.get()
	.then(doc  =>{
		if(doc.exists){
			const  data = doc.data();
			document.writeln(data.title)
			document.writeln(data.createdAt)
		} else{
			console.log("Docoument does not exist")
			document.writeln("Doc dosnt exist")
		}
})
```

For snapshot (Get notified as soon as the value changes) just put the code inside the `onSnapshot()` function like this

```js
myPost.onSnapshot(doc  =>{
	if(doc.exists){
		const  data  =  doc.data();
		document.writeln(data.title  +  `<br>`)
		document.writeln(data.createdAt)
	} else{
		console.log("Docoument does not exist")
		document.writeln("Doc dosnt exist")
}
})
```
that way the changes will happen in realtime.
for example you can add this to your html

```html
<h1  id="title">Display goes here</h1>
<input  type="text"  onchange="updatePost(event)">
```
and in your .js

```js
myPost.onSnapshot(doc  =>{
	if(doc.exists){
	const  data  =  doc.data();
	document.querySelector('#title').innerHTML  =  data.title
}

function  updatePost(e){
	const  db  =  firebase.firestore();
	const  myPost  =  db.collection('posts').doc('firstpost');
	myPost.update({ title:  e.target.value })
}
```
you have to press enter on inputfield to pass the changes.

### Query
#### Comparision
For this we are gonna create a new collection with the following data
Collection Name: Products
with 3 documents with auto ID
| Name (String) | Price (Number) |
|---|---|
| Hammer | 10 |  
| Chainsaw | 200 |
| Wrench | 25 | 

```js
const  db  =  firebase.firestore();
const  productsRef  =  db.collection('Products');
const  query  =  productsRef.where('price', '>', 10)

query.get()
	.then(produts  =>{
		produts.forEach(doc  =>{
		data = doc.data()
		document.writeln(`${data.name} at ${data.price} <br>`)
	})
})
```
query takes 3 arguments `price` the field, `'>'` Airthmetic operator and the `10` value. then we are iterating over each item and compairing it, then finally displaying the result

#### Ascending/Descending
used `orderBy`
```js
const  query  =  productsRef.orderBy('price', 'desc').limit(3)

query.get()
	.then(produts  =>{
		produts.forEach(doc  =>{
		data  =  doc.data()
		document.writeln(`${data.name} at ${data.price} <br>`)
	})
})
```
`.limit(3)` shows only 3 objects, its needed when working with a larger data-set
Compairing `price` and showing it in `desc` descending order.

---

**But in projects you might use it like this**

### App.js

```js
import { useEffect, useState } from 'react';
import './App.css';
import { Auth } from './components/auth';
import {db, auth} from './config/firebase';
import {getDocs, collection, addDoc, deleteDoc, doc, updateDoc} from 'firebase/firestore'

function App() {
  const [movieList, setMovieList] = useState([]);
  
  // New Movie state
  const [newMovieTitle, setNewMovieTitle] = useState("")
  const [newMovieRelease, setMovieRelease] = useState(0)
  const [isNewMovieOscar, setIsNewMovieOscar] = useState(false)
  const [updatedTitle, setUpdatedTitle] = useState("")

  const [error, SetError] = useState("");

  const movieCollection = collection(db, "movies")

  const getMovieList = async()=>{
    try {
      const data = await getDocs(movieCollection)
      const filteredData = data.docs.map((doc) =>({...doc.data(), id: doc.id}))
      setMovieList(filteredData)
    } 
    catch (error) {
      console.log(error)
  }}

  const deleteMovie = async(id) =>{
    try {
      const movieDoc = doc(db, "movies", id)
      await deleteDoc(movieDoc)
      getMovieList()
    } catch (error) {
      console.log(error)
      SetError(error.message)
    }
  }

  const updateMovieTitle = async(id) =>{
    try {
      const movieDoc = doc(db, "movies", id)
      await updateDoc(movieDoc, {title: updatedTitle})
      getMovieList()
    } catch (error) {
      console.log(error)
      SetError(error.message)
    }
  }

  useEffect(() =>{
    getMovieList()
  }, [])

  const onSubmitMovie = async() =>{
    try {
          await addDoc(movieCollection, {
            title: newMovieTitle, 
            releaseDate: newMovieRelease, 
            recievedAnOscar: isNewMovieOscar,
            userId: auth?.currentUser?.uid
          })
          getMovieList()
    } catch (error) {
      console.log(error)
      SetError(error.message)
    }
  }

  return <div className='App'>
    <Auth />

  {error && <h2 style={{ color: "red" }}>{error}</h2>}

  <div> 
    <input placeholder='Movie Title' onChange={(e) => setNewMovieTitle(e.target.value)} />
    <input placeholder='Release Date' type='number' onChange={(e) => setMovieRelease(Number(e.target.value))} />
    <input type='checkbox' checked = {isNewMovieOscar} onChange={(e) => setIsNewMovieOscar(e.target.checked)}/>
    <label>
      Recieved an Oscar
    </label>

    <button onClick={onSubmitMovie} > Submit </button>

  </div>

  <div>
    {movieList.map((movie)=>(
      <div>
        <h1 style={{color: movie.recievedAnOscar ? "green" : "red"}} > {movie.title} </h1>
        <p>Date: {movie.releaseDate}</p>
        <button onClick={() => deleteMovie(movie.id)}> Delete Movie </button>

      <input placeholder='New Title' onChange={(e) => setUpdatedTitle(e.target.value)} />
      <button onClick={() => updateMovieTitle(movie.id)}> Update Title </button>
      </div>
    ))}
  </div>
  </div>
}

export default App;

```

#### Get
`movieCollection` storing the collections in this const used as a reference.

`getDocs` is used to get elements from `movieCollection` collection

```js
    const data = await getDocs(movieCollection)
    const filteredData = data.docs.map((doc) =>({...doc.data(), id: doc.id}))
    setMovieList(filteredData)
```

the data contains a lot of unwanted meta data too, hence we used docs.map to get the filtered data

#### Delete

```js
    const movieDoc = doc(db, "movies", id)
    await deleteDoc(movieDoc)
	getMovieList()
```

delete requires Id, which is why we need to get it from the movie card component, hence rendered in each card

```js
    <button onClick={() => deleteMovie(movie.id)}> Delete Movie </button>
```
then used `getMovieList()` useEffect to remove the card from the front end

#### Update

```js
    const movieDoc = doc(db, "movies", id)
    await updateDoc(movieDoc, {title: updatedTitle})
```
for this again we took id from that card component, but we used `updatedTitle` useState to pass as the new title from input

```js
  const [updatedTitle, setUpdatedTitle] = useState("")
```

and in card component

```js
    <input placeholder='New Title' onChange={(e) => setUpdatedTitle(e.target.value)} />

    <button onClick={() => updateMovieTitle(movie.id)}> Update Title </button>
```

#### Post

```js
    await addDoc(movieCollection, {
        title: newMovieTitle, 
        releaseDate: newMovieRelease, 
        recievedAnOscar: isNewMovieOscar,
        userId: auth?.currentUser?.uid
    })
```

Formatting the data, then passing the values from useState to them.

```js
  const [newMovieTitle, setNewMovieTitle] = useState("")
  const [newMovieRelease, setMovieRelease] = useState(0)
  const [isNewMovieOscar, setIsNewMovieOscar] = useState(false)
```

```js
    <input placeholder='Movie Title' onChange={(e) => setNewMovieTitle(e.target.value)} />

    <input placeholder='Release Date' type='number' onChange={(e) => setMovieRelease(Number(e.target.value))} />

    <input type='checkbox' checked = {isNewMovieOscar} onChange={(e) => setIsNewMovieOscar(e.target.checked)}/>

    <label>
      Recieved an Oscar
    </label>

    <button onClick={onSubmitMovie} > Submit </button>
```

## Additional Notes

**onSubmitMovie()**

This function runs when you click the “Submit” button.
It does 3 things:

- Adds a new document to Firestore with the movie data.

- Calls getMovieList() to refresh the list.

- Runs only when you click, not when the component renders.

**deleteMovie(id)**

- This deletes a movie document from Firestore using its ID.
It:

- Finds the movie document (doc(db, "movies", id)).

- Deletes it using deleteDoc().

- Calls getMovieList() to refresh the displayed list.

**Why parentheses matter (())**

When you write:

```js
onClick={deleteMovie(movie.id)}
```


→ You’re calling the function immediately during rendering.
So it runs right away, not when you click.

When you write:

```js
onClick={() => deleteMovie(movie.id)}
```

→ You’re passing a function reference that runs only when clicked.

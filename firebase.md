# Firebase

## Initialization
to make a project
```bash
firebase init hosting
```
to test locally
```bash
firebase server
```
to get a live working Link
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


## Authentication

index.html
```html
<button onclick="googleLogin()">
	Login with google
</button>
```
app.js
```js
function  googleLogin(){

const  provider  =  new  firebase.auth.GoogleAuthProvider();
firebase.auth().signInWithPopup(provider)
	.then(result  =>{
		const  user  =  result.user;
		document.write(`Hello ${user.displayName}`)
		console.log(user)
	})
	.catch(error)
}
```

We use `GoogleAuthProvider`as the provider since we choose google as the OAuth provider, if we choose github then it would have been `GithubAuthProvider`

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

For snapshot (Get notified as soon as the value changes just put the code inside the `onSnapshot()` function like this
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


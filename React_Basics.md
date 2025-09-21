# Code
```jsx
import { useEffect, useState } from  'react'
import  './App.css'

const  Card  = ({ title }) =>{

	const [count, setCount] =  useState(0);
	const [hasLiked, setHasLiked] =  useState(false);

	useEffect(() =>{
		console.log(`${title} has been liked: ${hasLiked}`);
	}, [hasLiked]);

	return(
		<div  className='card'  onClick={() =>  setCount( count  +  1 )}>

			<h2>{title}  <br/>  {count  ||  null}</h2>

			<button  onClick={() =>  setHasLiked(!hasLiked) }>
				{hasLiked  ?  '❤️':  '🤍'}
			</button>
			
		</div>
)}

const  App =() =>{ 
	return(
		<div  className='card-container'>
			<Card  title="Star Wars"  />
			<Card  title  ="Avatar"/>
			<Card  title  ="Lord of the rings"/>
		</div>
	)}
export  default  App
```

# Explanation

## useState
```jsx

const [count, setCount] =  useState(0);
const [hasLiked, setHasLiked] =  useState(false);
```

Initialized the functions and then used in the return of `card` 

```jsx
<div  className='card'  onClick={() =>  setCount( count  +  1 )}>

	<h2>{title}  <br/>  {count  ||  null}</h2>
```
`setCount` is increased since we called it in the onClick button function.

As for the `setHasLiked` it is called in the button.

`count || null` will only show up if the count is more than 0

```jsx
<button  onClick={() =>  setHasLiked(!hasLiked) }>
	{hasLiked  ?  '❤️':  '🤍'}
</button>
```

`setHasLiked(!hasLiked)` Changes the state, if has liked is false, then it shows the option to make it true.

## useEffect

```jsx
useEffect(() =>{
	console.log(`${title} has been liked: ${hasLiked}`);
}, [hasLiked]);
```

here in the 3rd line `[hasLiked]` is a dependency, meaning that the component/console log will only happen if there is a change in hasLiked.

Otherwise when ever we click the card the message *Star Wars has been liked: false* will keep showing up.

With it, the component will only load up if `[hasLiked]` state changes.

## App

```jsx
const  App =() =>{ 
	return(
		<div  className='card-container'>
			<Card  title="Star Wars"  />
			<Card  title  ="Avatar"/>
			<Card  title  ="Lord of the rings"/>
		</div>
	)}
```

Calling `card` function again and again with different parameters *title*

```jsx
export  default  App
```

Finally we are `export default App` function to export the App function
# React Explanation

- [React Explanation](#react-explanation)
   * [ENV setup](#env-setup)
   * [main.jsx](#mainjsx)
   * [appwrite.js](#appwritejs)
      + [updateSearchCount](#updatesearchcount)
      + [getTrendingMovies](#gettrendingmovies)
   * [Components](#components)
      + [Spinner.jsx](#spinnerjsx)
      + [Search.jsx](#searchjsx)
      + [MovieCard.jsx](#moviecardjsx)
   * [App.jsx](#appjsx)
      + [APIs](#apis)
      + [useState](#usestate)
      + [Debouncer](#debouncer)
      + [Loading](#loading)
      + [Trending movies, usEffect and Debouncer](#trending-movies-useffect-and-debouncer)
      + [main body section](#main-body-section)
   * [State Management (Zustand)](#state-management-zustand)
      + [Nested States (Immer)](#nested-states-immer)

## ENV setup

```js
VITE_TMDB_API_KEY = 

VITE_APPWRITE_PROJECT_ID =
VITE_APPWRITE_PROJECT_NAME =
VITE_APPWRITE_ENDPOINT = "https://nyc.cloud.appwrite.io/v1"
VITE_APPWRITE_DATABASE_ID =
VITE_APPWRITE_TABLE_ID =
```

## main.jsx

```js
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.jsx'

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
)

```

`<StrictMode>` this runs the app 3 times for stress testing, disable this for production.

## appwrite.js

```js
import { Client, Databases, ID, Query } from 'appwrite'

const DATABASE_ID = import.meta.env.VITE_APPWRITE_DATABASE_ID;
const TABLE_ID = import.meta.env.VITE_APPWRITE_TABLE_ID;
const PROJECT_ID = import.meta.env.VITE_APPWRITE_PROJECT_ID;
const ENDPOINT = import.meta.env.VITE_APPWRITE_ENDPOINT;

const client = new Client()
    .setEndpoint(ENDPOINT)
    .setProject(PROJECT_ID)

const database = new Databases(client);

export const updateSearchCount = async(searchTerm, movie) =>{
    
    try{
        const result = await database.listDocuments(DATABASE_ID, TABLE_ID, [
            Query.equal('searchTerm', searchTerm),
        ])

        if(result.documents.length > 0){
            const doc = result.documents[0];

            await database.updateDocument(DATABASE_ID, TABLE_ID, doc.$id, {
                count: doc.count + 1,

            })
        } else{
            await database.createDocument(DATABASE_ID, TABLE_ID, ID.unique(), {
                searchTerm,
                count: 1,
                movie_id: movie.id,
                poster_url: `https://image.tmdb.org/t/p/w500${movie.poster_path}`,
            })
        }
    } catch (error){
        console.log(error);
    }

}

export const getTrendingMovies = async()=>{

    try {
        const result = await database.listDocuments(DATABASE_ID, TABLE_ID,
        [
            Query.limit(5),
            Query.orderDesc("count")
        ])
        return result.documents;
    } catch (e) {
        console.log(e);
    }
}
```

`import.meta.env.` is used to import the values from env file

### updateSearchCount

This function queries the database to see if the search term has been searched before, so you can either update its search count or add it as a new record.

```js
const result = await database.listDocuments(DATABASE_ID, TABLE_ID, [
    Query.equal('searchTerm', searchTerm),
])
```

searching for the term using the `searchTerm` usestate.

**if block**
```js
if(result.documents.length > 0){
    const doc = result.documents[0];
    await database.updateDocument(DATABASE_ID, TABLE_ID, doc.$id, {
        count: doc.count + 1,
})
}
```
`if(result.documents.length > 0)`

result.documents is the array of documents returned from the previous query.

.length > 0 means: "We found at least one document that matches this search term."

So this branch handles the case where the search term already exists in the database.

`const doc = result.documents[0];`
You take the first matching document.

Each document has a unique ID ($id) and fields like searchTerm and count.

```js
await database.updateDocument(DATABASE_ID, TABLE_ID, doc.$id, {
    count: doc.count + 1,
})
```

This updates that document.

It increases the count field by 1, meaning someone searched for this term again.

So the if part is: "The search term exists, so just increment its search count."

**else block**
```js
else{
    await database.createDocument(DATABASE_ID, TABLE_ID, ID.unique(), {
        searchTerm,
        count: 1,
        movie_id: movie.id,
        poster_url: `https://image.tmdb.org/t/p/w500${movie.poster_path}`,
    })
}
```
This runs if result.documents.length === 0, meaning no document was found for this search term.
ID.unique() generates a new unique ID for the document.

You create a new document with:

- searchTerm – the new search term.
- count: 1 – because this is the first time it’s searched.
- movie_id – stores the movie’s ID.
- poster_url – stores the movie poster URL.

So the else part is: "The search term doesn’t exist yet, so create a new record for it."

Then console logging any errors that may occur.

### getTrendingMovies

```js
export const getTrendingMovies = async()=>{
    try {
        const result = await database.listDocuments(DATABASE_ID, TABLE_ID,
        [
            Query.limit(5),
            Query.orderDesc("count")
        ])
        return result.documents;
    } catch (e) {
        console.log(e);
    }
}
```

This function is meant to fetch the top 5 most searched movies from your database.

This queries your Appwrite database collection (table).

Parameters:

- DATABASE_ID – your database ID.
- TABLE_ID – the specific collection storing search records.

Array of query filters:

Query.limit(5) → fetch only 5 documents.

Query.orderDesc("count") → sort the results in descending order by the `count` field (highest count first).

So this gets the top 5 search terms with the most searches.

`result.documents` is an array of document objects returned by Appwrite.

## Components

### Spinner.jsx

```js
import React from 'react'

const Spinner = () => {
  return (

<div role="status">
    <svg aria-hidden="true" className="w-8 h-8 text-gray-200 animate-spin dark:text-gray-600 fill-indigo-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
        <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
    </svg>
    <span className="sr-only">Loading...</span>
</div>

  )
}

export default Spinner

```

### Search.jsx

```js
import React from 'react'


const Search = ({searchTerm, setSearchTerm}) => {
  return (
    <div className='search'>
        <div>
            <img src="/search.svg" alt="search" />
            <input type="text"
            placeholder='Search through thousand of movies'
            value={searchTerm}
            onChange={(e) =>setSearchTerm(e.target.value)}
            />
        </div>
    </div>
  )
}

export default Search

```

### MovieCard.jsx

```js
import React from 'react'

const MovieCard = ({movie: 
    {title, vote_average, poster_path, release_date, original_language}
}) => {
  return (
    <div className='movie-card'>
        <img src={poster_path ? `https://image.tmdb.org/t/p/w500/${poster_path}` : '/no-movie.png'} alt={title} />

        <div className='mt-4'>
            <h3>{title}</h3>
            <div className='content'>
                <div className='rating'>
                    <img src="star.svg" alt="Star Icon" />
                    <p>{vote_average ? vote_average.toFixed(1) : `N/A`}</p>
                </div>
                <span>⚪</span>
                <p className='lang'>{original_language}</p>
                <span>⚪</span>
                <p className='year'>{release_date ? release_date.split('-')[0] : `N/A`}</p>
            </div>
        </div>

    </div>
  )
}

export default MovieCard

```

## App.jsx

```jsx
import React, { useEffect, useState } from 'react'
import Search from './component/Search'
import Spinner from './component/Spinner';
import MovieCard from './component/MovieCard';
import { useDebounce } from "@uidotdev/usehooks";
import { getTrendingMovies, updateSearchCount } from './appwrite';

const API_BASE_URL = 'https://api.themoviedb.org/3/';

const API_KEY = import.meta.env.VITE_TMDB_API_KEY;

const API_OPTIONS = {
  method: 'GET',
  headers:{
    accept: 'application/json',
    Authorization: `Bearer ${API_KEY}`
  }
}


const App = () => {

  const[searchTerm, setSearchTerm] = useState('')

  const [errorMessage, setErrorMessage] = useState(null);

  const [movieList, setMovieList] = useState([])

  const [isLoading, setIsLoading] = useState(false)

  const [trendingMovies, setTrendingMovies] = useState([])

  const debouncedSearchTerm = useDebounce(searchTerm, 300);

  const fetchMovies = async(query = '') =>{

    setIsLoading(true);
    setErrorMessage('');

    try{
        const endpoint = query
        ? `${API_BASE_URL}search/movie?query=${encodeURI(query)}`
        : `${API_BASE_URL}discover/movie?sort_by=popularity.desc`;

        const response = await fetch(endpoint, API_OPTIONS)
        
        if(!response.ok){
          throw new Error(`Failed to fetch movies`)
        }

        const data = await response.json();

        if(data.Response == 'False'){
          setErrorMessage(Error || 'Failed to fetch movie');
          setMovieList([])
          return
        }

        setMovieList(data.results || [])

        if(query && data.results.length > 0){
          await updateSearchCount(query, data.results[0]);
        }
    } 
      catch(error){
        console.log(`Error fetching movies: ${error}`);
        setErrorMessage('Error fetching movies, please try again later');
      } finally{
        setIsLoading(false)
      }
  }

  const loadTrendingMovies = async() =>{
    try {
      const movies = await getTrendingMovies();

      setTrendingMovies(movies)

    } catch (error) {
      console.log(`Error fetching trending movies ${error}`)
    }
  }

  useEffect(() =>{
    fetchMovies(debouncedSearchTerm);

  }, [debouncedSearchTerm])

  useEffect(() =>{
    loadTrendingMovies();
  }, [])

  return (
    <main>
        <div className='pattern'>

          <div className='wrapper' >
            <header>
              <img src="/hero.png" alt="Hero Banner" />
              <h1>Find <span className='text-gradient'>movies</span> you'll enjoy without the hassel</h1>

              <Search searchTerm={searchTerm} setSearchTerm={setSearchTerm}/>
            </header>
            
            {trendingMovies.length > 0 && (
              <section className='trending'>
                <h2>Trending Movies</h2>

                <ul>
                  {trendingMovies.map((movie, index) => (
                    <li key={movie.$id}>
                      <p>{index + 1}</p>
                      <img src={movie.poster_url} alt={movie.title} />
                    </li>
                  ))}
                </ul>
              </section>
            )}

            <section className='all-movies'>
              {debouncedSearchTerm && (
                  <div className="search-result-message">
                    <h3>
                      {movieList.length > 0 
                        ? `Result for: ${debouncedSearchTerm}` 
                        : `No results for: ${debouncedSearchTerm}`}
                    </h3>
                  </div>
                )}

              <h2 className='mt-[40px]'>All Movies</h2>
              <h3></h3>

              {isLoading ? (
                <Spinner />
              ) : errorMessage ? (
                <p className='text-red-500'>{errorMessage}</p>
              ) : (
                <ul>
                  {movieList.map((movie) => (
                    <MovieCard key={movie.id} movie={movie} />
                  ))}
                </ul>
              )}
            </section>

          </div>

        </div>
    </main>
  )
}

export default App
```

### APIs

```js
const API_BASE_URL = 'https://api.themoviedb.org/3/';

const API_KEY = import.meta.env.VITE_TMDB_API_KEY;

const API_OPTIONS = {
  method: 'GET',
  headers:{
    accept: 'application/json',
    Authorization: `Bearer ${API_KEY}`
  }
}
```

### useState

```js
  const[searchTerm, setSearchTerm] = useState('')

  const [errorMessage, setErrorMessage] = useState(null);

  const [movieList, setMovieList] = useState([])

  const [isLoading, setIsLoading] = useState(false)

  const [trendingMovies, setTrendingMovies] = useState([])
```

`searchTerm` Default value should be empty.

`errorMessage` is optional but used to manage all the error messages.

`movieList` is getting data from the API so its also initialised to empty array.

`isLoading` is the state, to check if the application is loading or if it has fetched the data from API

`trendingMovies` is taking data from the app write using -:

```js
Query.limit(5),
Query.orderDesc("count")
```

which checks and display the movies with the highst count.

### Debouncer

```js
  const debouncedSearchTerm = useDebounce(searchTerm, 300);
```

### Loading

```js
const fetchMovies = async(query = '') =>{

    setIsLoading(true);
    setErrorMessage('');

    try{
        const endpoint = query
        ? `${API_BASE_URL}search/movie?query=${encodeURI(query)}`
        : `${API_BASE_URL}discover/movie?sort_by=popularity.desc`;

        const response = await fetch(endpoint, API_OPTIONS)
        
        if(!response.ok){
          throw new Error(`Failed to fetch movies`)
        }

        const data = await response.json();

        if(data.Response == 'False'){
          setErrorMessage(Error || 'Failed to fetch movie');
          setMovieList([])
          return
        }

        setMovieList(data.results || [])

        if(query && data.results.length > 0){
          await updateSearchCount(query, data.results[0]);
        }
    } 
      catch(error){
        console.log(`Error fetching movies: ${error}`);
        setErrorMessage('Error fetching movies, please try again later');
      } finally{
        setIsLoading(false)
      }
  }
```

initalizing the values for setisLoading to true and setErrorMessage to empty string.

then setting up API endpoints

then checking if the response is okay or not. if response is okay then pass it into data using `await response.json()`

checking `data.Response` if its false, then we were not able to fetch, and setMovieList to empty array to avoid any errors.

`setMovieList(data.results || [])` data.results is expected to be an array of movies returned from the API call.

The **|| []** part is a fallback. It ensures that if data.results is undefined or null (for example, if the API doesn't return any results), setMovieList will still be called with an empty array instead of undefined.

`if(query && data.results.length > 0)`

query: This is the search term the user typed.
data.results.length > 0: Checks that the API returned at least one movie.

So this condition means: “If the user searched for something and the search returned results…”

`updateSearchCount` is an async function used back in *appwrite.js*.

It is being called with:

The search term (query)

The first movie from the results (data.results[0])

then at the end we use

```js
finally{
        setIsLoading(false)
      }
```

to exit the spinner and displat the movies fetched.

### Trending movies, usEffect and Debouncer

```js
  const loadTrendingMovies = async() =>{
    try {
      const movies = await getTrendingMovies();

      setTrendingMovies(movies)
    } catch (error) {
      console.log(`Error fetching trending movies ${error}`)
    }}

  useEffect(() =>{
    fetchMovies(debouncedSearchTerm);
  }, [debouncedSearchTerm])

  useEffect(() =>{
    loadTrendingMovies();
  }, [])
```

this function will try to fetch the `getTrendingMovies` back in *appwrite.js* and `setTrendingMovies` to the *movies* array.

then in useEffect

```js
  useEffect(() =>{
    fetchMovies(debouncedSearchTerm);
  }, [debouncedSearchTerm])
```

searchTerm → updates every time the user types in the search box.

useDebounce(searchTerm, 300) → creates a debounced version of searchTerm.
in the lower line 
```js
  }, [debouncedSearchTerm])
```
debouncer is used as a dependency.
The array [debouncedSearchTerm] tells React:

“Run this effect whenever debouncedSearchTerm changes.”

and then the `loadTrendingMovies` is run once to fetch trending movies.

### main body section

```js
<header>
  <img src="/hero.png" alt="Hero Banner" />
  <h1>Find <span className='text-gradient'>movies</span> you'll enjoy without the hassel</h1>

  <Search searchTerm={searchTerm} setSearchTerm={setSearchTerm}/>
</header>
```

Displays a hero image at the top.

Shows a heading with gradient-styled text for emphasis.

Includes the `<Search />` component:

Controlled input using searchTerm state.

Updates state via setSearchTerm when user types.


```js
{trendingMovies.length > 0 && (
  <section className='trending'>
    <h2>Trending Movies</h2>
    <ul>
      {trendingMovies.map((movie, index) => (
        <li key={movie.$id}>
          <p>{index + 1}</p>
          <img src={movie.poster_url} alt={movie.title} />
        </li>
      ))}
    </ul>
  </section>
)}
```
Conditional rendering: only show this section if trendingMovies is not empty.

Loops through trendingMovies with .map().

Displays a numbered list of trending movies with poster images.

key={movie.$id} → unique key for React to efficiently render lists.

```js
{debouncedSearchTerm && (
  <div className="search-result-message">
    <h3>
      {movieList.length > 0 
        ? `Result for: ${debouncedSearchTerm}` 
        : `No results for: ${debouncedSearchTerm}`}
    </h3>
  </div>
)}
```

Shows a message about the search results.

Conditional rendering: only shows if the user typed something (debouncedSearchTerm).

Ternary operator checks if movieList has results:

If yes → Result for: `<search term>`

If no → No results for: `<search term>`

```js
<h2 className='mt-[40px]'>All Movies</h2>

{isLoading ? (
  <Spinner />
) : errorMessage ? (
  <p className='text-red-500'>{errorMessage}</p>
) : (
  <ul>
    {movieList.map((movie) => (
      <MovieCard key={movie.id} movie={movie} />
    ))}
  </ul>
)}
```

Heading “All Movies” displayed for the main movie list.

Conditional rendering handles three states:

Loading → show `<Spinner />` component.

Error → show error message in red text.

Success → map through movieList and render `<MovieCard />` for each movie.

key={movie.id} → unique key for React list rendering.

`<MovieCard /> ` → custom component that displays each movie’s info (poster, title, etc.).


## State Management (Zustand)

for this i am using another example code. A simple add to cart ecommerce code

// app/page.jsx

```jsx
"use client"

import Cart from "../components/Cart"
import ProductList from "../components/ProductList"
import {PRODUCTS} from "../components/products"


export default function Home(){
  

  return(
    <div className="">
      <h3>Welcome to store</h3>
      <ProductList products={PRODUCTS} />
      <Cart />
    </div>
  )
}
```

in the components file these are the products data

components/products.jsx

```jsx
export const PRODUCTS = [
  {
    id: 1,
    name: "Phone",
    description: "A modern mobile phone with a smart display, latest version",
  },
  {
    id: 2,
    name: "Laptop",
    description: "Lightweight laptop with 16GB RAM and long battery life",
  },
  {
    id: 3,
    name: "Wireless Headphones",
    description: "Over-ear bluetooth headphones with noise cancellation",
  },
  {
    id: 4,
    name: "Smartwatch",
    description: "Water-resistant smartwatch with fitness tracking and GPS",
  },
  {
    id: 5,
    name: "Tablet",
    description: "10-inch tablet with high-resolution display and stylus support",
  },
  {
    id: 6,
    name: "Coffee Maker",
    description: "Programmable drip coffee maker with timer and thermal carafe",
  },
  {
    id: 7,
    name: "Electric Kettle",
    description: "Rapid-boil electric kettle with temperature control",
  },
  {
    id: 8,
    name: "Bluetooth Speaker",
    description: "Portable speaker with deep bass and 12-hour battery life",
  },
  {
    id: 9,
    name: "Gaming Controller",
    description: "Ergonomic wireless controller compatible with multiple platforms",
  },
  {
    id: 10,
    name: "External SSD",
    description: "Fast USB-C external SSD with hardware encryption support",
  },
  {
    id: 11,
    name: "Desk Lamp",
    description: "Adjustable LED desk lamp with touch controls and dimmer",
  },
  {
    id: 12,
    name: "Backpack",
    description: "Durable laptop backpack with multiple compartments and rain cover",
  },
];

```

Now for the main files

Before we use the products, we need to have a centralized state management solution for which we will use zustand

we'll make a sort of "schema". make a newfolder with the name store and file cartStore.jsx

```jsx
import { create } from "zustand";

export const useCartStore = create((set) => ({
    cart: [],

    addToCart: (product) => set((state) => ({cart: [...state.cart, product]})),

    removeFromCart: (productId) => 
        set((state) => ({
            cart: state.cart.filter(product => product.id !== productId)
        })),

    clearCart: () => set({cart: []}) ,
}))
```

Notice how we are doing `cart: [...state.cart, product]` in the addToCart thats because we cant just append it to list. we are creating a new list with the same name and `...state.cart` pushes the element from the old cart into the new cart and the `, product` the new product is added in as well.

Then we have Cart.jsx in components

```jsx
"use client"
import { useCartStore } from '../store/cartStore'

const Cart = () =>{

    const {cart, removeFromCart, clearCart} = useCartStore()

    return(
        <div>
            <h2>Cart</h2>
            {cart.map((product) =>(
                <div key={product.id}>
                    <span>{product.name}</span>
                    <button
                    onClick={() => removeFromCart(product.id)}>
                        Remove
                    </button>
                </div>
            ))}
            {cart.length > 0 && (
                <button onClick={clearCart}>Clear Cart</button>
            )}
        </div>
    )
}

export default Cart;
```

See we didnt used useState here, we imported the functions and pass in the values when the button is clicked.

for the product cards
make ProductList.jsx in component folder

```jsx
"use client"

import { useCartStore } from '../store/cartStore'

const ProductList = ({products}) =>{

    const addToCart = useCartStore((state) => state.addToCart)

    return(
        <div>
            {products?.map((product) =>(
                <div key={product.id}>
                    <h3>{product.name}</h3>
                    <p>{product.description}</p>
                    <button onClick={() => addToCart(product)}>
                        Add to cart
                    </button>
                </div>
            ))}
        </div>
    )
}

export default ProductList;
```

Again here we are passing in the state and mapping over the Products

### Nested States (Immer)

When we use nested data, it becomes harder to push the elements into the list. like before, we need to create a new list, add the elements from old list to the new list and then add the new element. If the data is nested then you have to do this for every row and nested list.

The solution for that is to use Immer (produce) function

```jsx
import { create } from "zustand";
import { produce } from "immer";

const initialState = {
    user: {
        id: "user123",
        friends: ["jack", "jessica", "paul"],
        profile: {
            name: "John Doe",
            email: "john.doe@example.com",
            address: {
                street: "123 Main St",
                city: "NewYork",
                zipcode: "123456",
            },
        },
    },
};

export const useStore = create((set) =>({
    ...initialState,
    updateAddressStreet: (street) =>
        set(
            produce((state) =>{
                state.user.profile.address.street = street;
            })
        )
}));
```

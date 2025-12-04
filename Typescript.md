# Typescript

TS compiler compiles code to JS {This step is called transpilation}

medium to large projects.

# Table of contents

- [Typescript](#typescript)
- [Table of contents](#table-of-contents)
- [Type Safety basic](#type-safety-basic)
- [Data Types](#data-types)
   * [Primitives and References](#primitives-and-references)
   * [Tuples](#tuples)
   * [Enums](#enums)
   * [Any, unknown, Void, Null, Undefined, Never](#any-unknown-void-null-undefined-never)
- [Type Inference](#type-inference)
- [Type Annotation](#type-annotation)
- [Interfaces](#interfaces)
   * [Extending Interfaces](#extending-interfaces)
- [Type Aliases](#type-aliases)
- [Union and Intersection Types](#union-and-intersection-types)
   * [Difference between Intersection and interfaces](#difference-between-intersection-and-interfaces)
      + [**Interface Advantages**](#interface-advantages)
      + [**Type Advantages**](#type-advantages)
   * [**Which should YOU use?**](#which-should-you-use)
- [Classes](#classes)
   * [How is it different from Interfaces or Type?](#how-is-it-different-from-interfaces-or-type)
      + [**interface/type**](#interfacetype)
      + [**class**](#class)
   * [Constructor](#constructor)
   * [Access Modifiers](#access-modifiers)
      + [Public](#public)
      + [Private](#private)
- [Using in React](#using-in-react)
   * [Type props](#type-props)
      + [Arrays and Tuples](#arrays-and-tuples)
      + [React.CSSProperty](#reactcssproperty)
      + [Records](#records)
   * [Function](#function)
   * [Children and React.node](#children-and-reactnode)
   * [Interfaces (When to use what)](#interfaces-when-to-use-what)
   * [ComponentProps](#componentprops)
      + [WithRef and WithoutRef](#withref-and-withoutref)
   * [Interface Extending and Type &](#interface-extending-and-type-)
   * [Event Handlers](#event-handlers)
   * [Hooks](#hooks)
      + [useState](#usestate)
      + [useState Objects](#usestate-objects)
      + [useEffect](#useeffect)
      + [useRef](#useref)
   * [as const](#as-const)
   * [Omit](#omit)

to install typescript

```jsx
	npm install -g typescript
```

to compile ts file manually use the

```jsx
tsc index.ts
```

to make tsconfig file just use

```jsx
tsc --init
```

if you are using a src folder, then just type

```jsx
tsc
```

to compile the code . to keep the code running, use the command

```jsx
tsc --watch
```

# Type Safety basic

```jsx
let age: number = 20;

let name: string = 'Manpreet Singh'
```

# Data Types

## Primitives and References

when you have an array lets say 

a = [1,2,3,4,5]

and b = a

but we want to remove the last value of b array. So b should be [1,2,3,4] but our `a` will also be 1,2,3,4

`Any value that dosent have [], {}, or () is Primitive`

we can directly copy this

```jsx
var a = 12
var b = a
```

but for reference we can’t directly copy them

```jsx
var a = [1,2,3,4,5]
var b = a
```

Here b is a reference to a. so any change to B will also change A. 

```jsx
b.push(6)
console.log(b)
console.log(a)
```

we pushed item in b but the same item will also appear in a.

## Tuples

This is where we have a fixed array

```jsx
const a: [string, number] = ["Manpreet", 21]

console.log(a)
console.log(typeof(a))
```

## Enums

Here we create objects and then we can assign them values

```jsx
enum UserRoles{
    ADMIN = 'admin',
    GUEST = "guest"
}

console.log(UserRoles.ADMIN)
```

## Any, unknown, Void, Null, Undefined, Never

```jsx
let a;

console.log(typeof(a))
```

Here we get error saying that the type of a is `undefined` this is what we **never** want hence we always do this

But in this code

```jsx
let a: unknown

a=12
a = "Manpreet"

console.log(a)
console.log(typeof(a))
console.log(a.toUpperCase())
```

this code will work for any String but fail for number. Hence it gives error that we have to define the type of a

`unknown` Lets you work, but kinda keeps reminding you that you have to set its value in the future.

This function returns void output, and typescript automatically treats it as a return type of void

```jsx
function abc(){
    console.log('Hey')
}

abc()
```

For example this function will give me error because here i have defined the function to give a boolean value

```jsx
function abc(): boolean{
    console.log('Hey')
}

abc()
```

But this wont give me any error

```jsx
function abc(): boolean{
    return true
}

abc()
```

For null, this will give us an error because tsc expects a to be null

```jsx
let a: null
a = 12
```

hence the only argument it will receive is going to be `a=null`

to resolve that issue you can do this

```jsx
let a: number | null
a = 12
```

either it can be a number or it can be a null value

Never

```jsx
function abc(){
    while(true){
        console.log("hello")
    }
}
console.log("While loop Start")

abc()

console.log("While loop ended")
```

Since we know that this function will never stop, and will keep running we will use `never` here

```jsx
function abc(): never{
    while(true){
        console.log("hello")
    }
}
```

It isnt used that much, because after this type is used anything after the function will not run.

# Type Inference

This happens automatically, like here the type of a is automatically given number

```jsx
let a = 12
```

# Type Annotation

This is where we basically give the type to variable

```jsx
let a: number | string
a = 12
a = "Manpreet"
```

# Interfaces

interfaces mean telling the shape of object

```jsx
interface User{
    name:string,
    email: string,
    password: string
}

function getUserData(obj: User): void{
    console.log(obj.name)
}
getUserData({name: "Manpreet"})
```

Here i will get error because `getUserData`function expects all the values that are defined in interface User

therefore i have to pass in everything

```jsx
interface User{
    name:string,
    email: string,
    password: string
}

function getUserData(obj: User): void{
    console.log(obj.name)
}
getUserData({name: "Manpreet", email: "Manpreet@gmail.com", password: "abcd"})
```

If you have a variable that can be empty like gender, in that case use `?:`

```jsx
    gender?: string
```

## Extending Interfaces

In the same previous example we extend Admin as a User

```jsx
interface User{
    name:string,
    email: string,
    password: string
    gender?: string
}

interface Admin extends User{
    admin: boolean
}

function abc(obj: Admin){
    obj.admin
}
abc({name: "Manpreet", email: "Manpreet@gmail.com", password: "abcd", gender: "male", admin: true})
```

function `abc` can access obj.admin but function `getUserData` cannot access obj.admin

*Note -: If you have 2 interfaces with the same name then both of them will merge*

# Type Aliases

Here you can create your own new data types by merging different datatypes

```jsx
type password = string | number | null
let a: password = "1234Manpreet"
console.log(a)
console.log(typeof(a))
```

here we created a new type of data which is `password` and that datatype is a mix of string, number and null. then we simply used it in a.

# Union and Intersection Types

Union is basically `|` we used, like this

```jsx
let a = number | string
```

That **&** is an **intersection**
It literally **merges** both types into one bigger type.

```jsx
type User = {
    name: string,
    email: string,
}
type Admin = User & {
    getDetails(user: string): void
}

function abc(obj: Admin){
    obj.getDetails
}
```

So the new Admin becomes :

```jsx
{
  name: string,
  email: string,
  getDetails(user: string): void
}
```

## Difference between Intersection and interfaces

### **Interface Advantages**

- Can be **merged automatically** (declaration merging)
- More structured for object shapes
- Better for extending multiple parent interfaces
- Preferred for class-type definitions

### **Type Advantages**

- Can represent **anything** (union, primitive, tuple, function type)
- Intersection (&) allows combining multiple types flexibly
- More powerful in advanced type manipulations

## **Which should YOU use?**

If you’re defining the shape of an object?

→ **Use interfaces. They scale better.**

If you’re mixing types or doing complex unions/intersections?

→ **Use types. They’re more powerful.**

# Classes

When multiple vars have the same things.

```jsx
class Device{
    name = 'LG';
    price = 12000;
    category = 'Digital';
}
let d1 = new Device();

console.log(d1)
```

## How is it different from Interfaces or Type?

### **interface/type**

- Only describe **shape**
- Don’t create objects
- No runtime existence
    
    (Meaning they disappear after compile)
    

### **class**

- Creates objects at runtime
- Has actual properties + methods
- Can have constructors, inheritance, logic
- Exists in real JavaScript

## Constructor

Here its just inserted the values and its not really that useful. think of class like it provides a structure and constructor provides the data.

**Note -: whenever you define constructor in a class, the constructor runs first**

```jsx
class BottleMaker{
    constructor(public CN: string, public price: number, public matterial: string = "Unknown"){
        console.log(`Bottle created: ${this.CN} with the price ${this.price} made from matterial ${this.matterial}`)
    }
}
let b1 = new BottleMaker("Milton", 1200, "metal")
let b2 = new BottleMaker("Ocean World", 30) // Explicitely says "Unknown" matterial, since we didnt pass it
```

## Access Modifiers

### Public

This type of value can be changed everywhere, suppose in the above `BottleMaker` we are using `public CN` this means that after defining b1, if we wrote anywhere that [b1.CN](http://b1.CN) = “Ocean World”, its company Name will change to Ocean world since its a publicly accessed variable

### Private

The variable defined with a private can be accessed ONLY in the class. Hence to use this we have to create a separate function.

```jsx
class BottleMaker{
    constructor(private CN: string){
        console.log(`Bottle created: ${this.CN}`)
    }
    changeCompany(){
        this.CN = "Jameson"
        console.log(`Changed name: ${this.CN}`);
    }
}
let b1 = new BottleMaker("Milton")
b1.changeCompany()

// b1.CN = "Ocean World" // Throw error in TS
```

# Using in React

## Type props

Suppose you have a button component like this, and passing the props like this

```jsx
<Button backgroundColor="red" fontSize={50} pillShape={true} />
```

we first should make a type and then tell it what type the data is and then import them in the function

```jsx
import React from "react";

type ButtonProps = {
    backgroundColor: string, 
    fontSize: number, 
    pillShape?: boolean
}

export default function Button( {backgroundColor, fontSize, pillShape} : ButtonProps){

    return(
        <button className={`px-4 py-2 ${pillShape ? "rounded-full" : "rounded"}`} 
            style={{backgroundColor, fontSize}}
        >
            Click me
        </button>
    )
}
```

You can also make a different type if you are using it for 2 elements

```jsx
type Color = "red" | "blue" | "green" | "white"

type ButtonProps = {
    backgroundColor: Color, 
    fontSize: number, 
    pillShape?: boolean,
    textColor: Color
}

// then pass textColor in style like this
style = {{color: textColor}}
```

### Arrays and Tuples

Suppose you are taking an array of values like this, focus on padding2

```jsx
    padding: [number, number, number],
    padding2: number[]
```

here we can pass any number of values like padding2=[12,34,12,43,4,52,3,53,23,43,23,32] which becomes problematic. To resolve this we use Tuples, here it has a fixed length.

Take a look at `padding` it only has 3 numbers for input.

### React.CSSProperty

we can just pass a style object and refer that to React.CSSProperties this accepts all the style elements

```jsx
type ButtonProps = {
    style: React.CSSProperties
}
```

and this is how you will pass in the props in, Since we are using the style `<button style={style}>` we have to pass the props name according to that

```jsx
      <Button style={{
        backgroundColor: 'blue',
        fontSize: 25,
        color: 'white',
        padding: '1rem 2rem',
        borderRadius: 8
      }} />
```

### Records

These are the key value pairs, for example you have this

```jsx
      <Button 
        borderRadius={{
          topLeft: 5,
          topRight: 5
        }}
      />
	      
// component you have this
type ButtonProps = {
    borderRadius: {
        topLeft: number,
        topRight: number
    }
}
```

But a better way to do this is by using records

```jsx
// In components use this
type ButtonProps = {
    borderRadius: Record<string, number>
}
```

## Function

Here, we have to pass the type in the function. 

React **always passes a MouseEvent**, whether you want it or not. and MouseEvent is not a String

If you want your function to receive YOUR value (a string). You ***cannot*** do this:

```tsx
<button onClick={func}>
```

Because React will pass an event.

You must **wrap it** so YOU control what gets passed:

```tsx
<button onClick={() => func("hello")}>
```

Now React calls the wrapper (no problem),

and the wrapper calls your function with a string.

```jsx
function App() {
    const func = (test: string) => {
      console.log(test)
      return 5
    }
  return (
    <>
      <Button func={func}
      />
    </>
  )
}

// In the component file do this
type ButtonProps = {
    func: (test: string) => number;
}

export default function Button( {func}: ButtonProps){

    return(
        <button onClick={() => func("Test String")}>
            Click me
        </button>
    )
}
```

## Children and React.node

When you have to take the value of the button from them main app, and pass it into the component that is where you have to use `React.ReactNode` 

```jsx
// In the main app, you'll have to pass value the other way
<Button>Check this out</Button>

// In the component file do this

type ButtonProps = {
    children: React.ReactNode
}

export default function Button( {children}: ButtonProps){
    return(
        <button> {children} <button>
    )
}
```

## Interfaces (When to use what)

We were doing all of the code above in Types as of now, but Interface is also important, like mentioned above you can extend interfaces but you cannot use Unions in it

Use `interface` for objects and props.

Use `type` for everything else.

## ComponentProps

when you are passing the props, some elements have their own values as well like <a> has the url, or <image> has the src or <button> has the auto Focus and type {submit, reset}. that will be automatically available when you use ComponentProps.

This mean that this will work **Only** when its a **button** since you are declaring type and autoFocus

```jsx
<Button type='submit' autoFocus={true} />

// In the component

interface ButtonProps {
    type: "submit" | "reset" | "button";
    autoFocus?: boolean
}

export default function Button( {type, autoFocus}: ButtonProps){
    return(
        <button >
            Click here
        </button>
    )
}
```

But a better way is to use componentProps which automatically pick it.

Notice how we can directly access type and autoFocus without initializing them

```jsx
<Button type='submit' autoFocus={true} defaultValue="Test" />

// In the component

type ButtonProps = React.ComponentPropsWithoutRef<'button'>

export default function Button( {type, autoFocus, ...rest }: ButtonProps){

    return(
        <button type={type} autoFocus={autoFocus} {...rest}>
            Click here
        </button>
    )
}
```

`{…rest}` picks up all the other attributes and passes it into the button, without you mentioning it, like here we mentioned type={type}.

### WithRef and WithoutRef

if you are passing any ref, then use withRef and if not use withoutRef. its better to explicitly say if you are using ref or not. Although the code should also work just fine without explicitly mentioning it.

## Interface Extending and Type &

In Type if you want to ***extend** an already existing type you will use &. For example*

```jsx
type ButtonProps ={
    type: "button" | "submit" | "reset";
    color: "red" | "blue" | "green"
}
type SuperButtonProps = ButtonProps &{
    size: "md" | "lg"
}
```

If we have to do the same thing using interfaces we will use the keyword extends

```jsx
interface ButtonProps {
    type: "button" | "reset" | "submit",
    color: "red" | "blue" | "green"
}
interface SuperButtonProps extends ButtonProps{
    size: "md" | "lg"
}
```

Want to know how to take out instead of extending, check out -: [Omit](https://www.notion.so/Typescript-2be56ff8855c802b8da5ef1bac930a85?pvs=21)

## Event Handlers

In specific cases like when you are making onClick functions you can pass in the the event function inline. you **don’t** have to specifically write the type. If you **Hover over the e you will get that TS has automatically given it the type e: React.MouseEvent<HTMLButtonElement, MouseEvent>**

This happened because TS in this case has the context

```jsx
button onClick={(e) => console.log("Clicked")}
```

But if you are making the function outside of the onClick then you have to give it the Type. Like this

```jsx
    const handleClick = (event: React.MouseEvent<HTMLButtonElement, MouseEvent>) =>{
        console.log('Clicked')
    }
    return(
        <button onClick={handleClick}>
            Click here
        </button>
    )
```

## Hooks

### useState

For this you can hover over the element like setCount and that will give you the type of element.

```jsx
function App() {
  const [count, setCount] = useState(0)
  return (
    <>
    <div>Count: {count}</div>
      <Button setCount={setCount} />
    </>
  )
}

// in the component

type ButtonProps = {
    setCount: React.Dispatch<React.SetStateAction<number>> //Type of state
}

export default function Button( {setCount}: ButtonProps){

    return(
        <button onClick={() => {setCount (c => c+1)}}>
            Click here
        </button>
    )
}
```

Now this is simple, because when we initialize, it takes that value

### useState Objects

Lets say we have an object like user {name, age}, so to use that object, we first need to make it into a type/interface

then use that type in useState. Since the initial value in an object can be null, we used `<User | null>`

and then `name = user?.name` meaning that it is not required, and if its not required it means that even if the name is undefined or null, it wont crash the whole app

```jsx
    type User ={
        name: string,
        age: number
    }
    const [user, setUser] = useState<User | null>(null)
    
    const name = user?.name;
```

### useEffect

You don’t really need to use any types here like come on man

### useRef

```jsx
    const ref = useRef<HTMLButtonElement>(null)

    return(
        <button ref={ref}>
            Click here
        </button>
```

## as const

Here we told TypeScript that this array is **read-only** and contains those specific strings, not just any strings. That makes your typing bulletproof

```jsx
const buttonTextOptions = [
    "Click me",
    "Click me again!",
    "Click one last time"
] as const

export default function Button(){
    return(
        <button>
            {buttonTextOptions.map((option)=>{
                return option;
            })}
        </button>
    )
}
```

## Omit

Used when we want to use the “extend” but want to take out instead of adding in.

```jsx
type User = {
    name: string,
    sessionId: string
}
type Guest = Omit<User, "name">
```

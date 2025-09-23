# Express backend

## Table of Contents

- [Express backend](#express-backend)
  - [Table of Contents](#table-of-contents)
  - [Initialize Express app](#initialize-express-app)
  - [.env](#.env)
  - [Server.js](#server.js)
  - [Routes](#routes)
    - [home.js](#home.js)
    - [notesRouter.js](#notesrouter.js)
    - [user.js](#user.js)
    - [Images.js (Optional)](#images.js-(optional))
  - [Controllers](#controllers)
    - [notesController.js](#notescontroller.js)
    - [userController.js](#usercontroller.js)
    - [Image/fileController.js (Optional)](#image/filecontroller.js-(optional))
  - [Model](#model)
    - [notesModel.js](#notesmodel.js)
  - [userModel.js](#usermodel.js)
    - [Image/fileController.js](#image/filecontroller.js)
  - [Middleware](#middleware)
    - [requireAuth.js](#requireauth.js)
    - [multerConfig.js (Optional)](#multerconfig.js-(optional))

## Initialize Express app

```bash
npm init -y
```
This will install Node and package.json type files

```bash
npm install express
```

This will install express

```bash
npm run dev
```
to run 

```bash
npm install
```
to install all the dependencies

## .env
For this file we will keep it in this format

```js
PORT=
MONGOURI=
SECRET=
```

## Server.js

This file is the main entry point for your backend code and files

```js
require('dotenv').config()
const  express  =  require('express')
const  mongoose  =  require('mongoose')

// Routes
const  homePage  =  require('./routes/home')
const  notesRoute  =  require('./routes/notesRouter')
const  userRoute  =  require('./routes/user')

// Express App
const  app  =  express()

// Middleware
app.use(express.json())

app.use((req, res, next) =>{
	console.log(req.path, req.method)
	next()
})

// Routes
app.use('/', homePage)
app.use('/', userRoute)
app.use('/notes', notesRoute)

mongoose.connect(process.env.MONGOURI)

	.then(() =>{
		app.listen(process.env.PORT, () =>{
		console.log('Connected to DB and Listening on port ', process.env.PORT)
		console.log('http://localhost:4000/')
		})
	})

	.catch((error) =>{
		console.log(error)
	})
```
`routes` will add routes, and then you can point it towards your custom path, like notes are at `/notes` path meaning at http://localhost:4000/notes . but for APIs its better to use a api/notes to ensure the safety of app.

**Note -:** in `.env` file i have specified a specific port to use and that port is 4000 in my case, the default port of express is 3000.

`process.env.MONGOURI` or `process.env.PORT` will take the value from .env file and just process them.

## Routes

To create routes we will use a seperate folder

### home.js

```js
const express = require('express')
const router = express.Router()

router.get('/', (req, res) => {
    res.json({message: "Welcome to the Home Page"})
})

module.exports = router
```

A basic route to display the name

### notesRouter.js

```js
const express = require('express')
const {
    getNotes,
    getNote,
    createNote,
    deleteNote,
    updateNote
} = require('../controllers/notesController')

// Importe middleware Auth
const requireAuth = require('../middleware/requireAuth')

const router = express.Router()

// Auth for routes
router.use(requireAuth)

// Get all notes
router.get('/', getNotes)

// Get a single note
router.get('/:id', getNote)

// Create note
router.post('/', createNote)

// Delete note
router.delete('/:id', deleteNote)

// Update note
router.patch('/:id', updateNote)

module.exports = router
```

for this route, we are using controllers and then using it in the routes and specifing the functions.

### user.js

```js
const express = require('express')
const router = express.Router()

// Controller function
const {signupUser, loginUser} = require('../controllers/userController')

// Login
router.post('/login', loginUser)
// Signup
router.post('/signup', signupUser)

module.exports = router
```

### Images.js (Optional)

Optional File for image/file handling route

```js
const express = require('express')

const {
        getImages,
    getImage,
    createImage,
    deleteImage,
    updateImage
} = require('../controllers/ImageController')

const upload = require('../middleware/multerConfig')
const requireAuth = require('../middleware/requireAuth')

const router = express.Router()


router.get('/welcome', (req, res) => {
    res.json({msg: 'Welcome to the Momentus API, here you will see all the images'})
})

// protecting routes
router.use(requireAuth)

router.get('/', getImages)

router.get('/:id', getImage)

router.post('/', upload.single('image'), createImage);

router.delete('/:id', deleteImage)

router.patch('/:id', updateImage)

module.exports = router
```

## Controllers
Again making a new folder for the `controllers`, this is the functions that the `routes` folder is using

### notesController.js

```js
const {json} = require('express')
const Note = require('../models/notesModel')
const mongoose = require('mongoose')

// Get all notes
const getNotes = async(req, res) =>{
    const user_id = req.user._id
    const notes = await Note.find({user_id}).sort({createdAt: -1})

    res.status(200).json(notes)
}

// Getting a single Note
const getNote = async(req, res) =>{
    const {id} = req.params
    
    if(!mongoose.Types.ObjectId.isValid(id)){
        return res.status(404).json({error: "Not a valid key"})
    }

    const note = await Note.findById(id)
    
    if (!note){
        return res.status(404).json({error: "No suck note exist"})
    }

    res.status(200).json(note)
}

// Creating a new Note
const createNote = async(req, res) => {
    const {title, description, pinned } = req.body
    let emptyFields = []

    if(!title){
        emptyFields.push('title')
    }

    // Only added title, will add more stuff in app

    if(emptyFields.length>0){
        return res.status(400).json({error: "Please Fill all the necessary titles", emptyFields})
    }

    // Adding note to DB
    try{
        const user_id = req.user._id
        const note = await Note.create({title, description, pinned, user_id})

        res.status(200).json({
          message: "Note Added",
          note: note
        })
    } catch(error){
        res.status(400).json({error: error.message})
    }
}

// Delete a note
const deleteNote = async(req, res) => {
    const {id} = req.params

    if(!mongoose.Types.ObjectId.isValid(id)){
        return res.status(404).json({error: 'Invalid ID'})
    }

    const note = await Note.findOneAndDelete({_id: id})
    
    if(!note){
        return res.status(404).json({error: 'No such note exist'})
    }

    res.status(200).json({
        message: "Note deleted",
        note: note
    })
}

// Updating a note
const updateNote = async(req, res) =>{
    const {id} = req.params

    if(!mongoose.Types.ObjectId.isValid(id)){
        return res.status(404).json({error: 'Invalid ID'})
    }

    const note = await Note.findByIdAndUpdate(id, {
        ...req.body
    }, {new: true})

    if(!note){
        return res.status(404).json({error: 'No such note exist'})
    }

    res.status(200).json({
        message: "Note Updated",
        note: note
    })
}

module.exports = {
    getNotes,
    getNote,
    createNote,
    deleteNote,
    updateNote
}
```

This is first gonna take the notes model because based on it defines the schema of our notes database. Almost all if not all functions in this is going to be async in nature.

- const user_id = req.user._id → gets the logged-in user’s id to link notes to them.

- ...req.body → spread operator, copies all fields from request body into the update (handy, but risky if user sends unwanted fields).

- mongoose.Types.ObjectId.isValid(id) → checks if id is a valid MongoDB ObjectId before querying.

- find({user_id}).sort({createdAt:-1}) → fetch user’s notes, newest first.

- findById(id) → get one note by id.

- create({...}) → add a new note.

- findOneAndDelete({_id:id}) → delete a note and return it.

- findByIdAndUpdate(id,{...},{new:true}) → update a note and return updated version.

- emptyFields.push('title') → collects missing fields so you can tell the frontend what’s missing.


### userController.js

```js
const User = require('../models/userModel')
const jwt = require('jsonwebtoken')

const createToken = (_id) =>{
    return jwt.sign({_id}, process.env.SECRET, {expiresIn: '3d'})
}

// Login the user
const loginUser = async(req, res)=>{
    const {email, password} = req.body

    try{
        const {user, message} = await User.login(email, password)

        // creating token
        const token = createToken(user._id)
        res.status(200).json({
            message,
            userID: user._id,
            email, 
            token
        })

    }catch(error){
        res.status(400).json({error: error.message})
    }
}

// Signup the user
const signupUser = async(req, res)=>{
    const {email, password} = req.body

    try{
        const user = await User.signup(email, password)

        // creating token
        const token = createToken(user._id)
        res.status(200).json({
            message: "User signed up",
            userID: user._id,
            email, 
            token
        })

    }catch(error){
        res.status(400).json({error: error.message})
    }
}

module.exports = {signupUser, loginUser}
```

- createToken(_id) → generates a JWT signed with user’s _id, secret key, and expires in 3 days.

- loginUser → takes email + password, calls User.login (custom static method on model), and if valid, returns a JWT plus user info.

- signupUser → takes email + password, calls User.signup (custom static method), and if successful, returns a JWT plus user info.

- Both functions wrap logic in try/catch for error handling.


*Note -: If you want to add user name into it then you can do the following*

```js
const signupUser = async (req, res) => {
    const { name, email, password } = req.body 

    try {
        const user = await User.signup(name, email, password)
        const token = createToken(user._id)

        res.status(200).json({ name: user.name, email: user.email, token })
    } catch (error) {
        res.status(400).json({ error: error.message })
    }
}
```

### Image/fileController.js (Optional)

imageController.js manages the multer files, bellow is an example of it

```js
const { json } = require('express')
const mongoose = require('mongoose')
const Image = require('../model/imageModel')

// Get all images
const getImages = async(req, res)=>{

    const images = await Image.find({ user: req.user._id });
    res.status(200).json(images);
}

// Get a single Image
const getImage = async(req, res)=>{

    const{ id } = req.params
    if(!mongoose.Types.ObjectId.isValid(id)){
        return res.status(404).json({
            error: "ID cannot exist",
            message: "This ID cannot exist in Mongo database"
        })
    }
    const image = await Image.findById(id)

    if(!image){
        return res.status(404).json({
            error: "No such Image is found",
            message: "The ID you are looking for is not present in the database"
        })
    }

    res.status(200).json(image)

}

// Create new Image
const createImage = async (req, res) => {
  try {
    // Multer stores the file info here
    if (!req.file) {
      return res.status(400).json({
        error: 'Image file is required',
        message: 'No image file was uploaded',
      });
    }

    const { caption } = req.body;

    const newImage = await Image.create({
      filename: req.file.filename,
      originalName: req.file.originalname,
      url: `/uploads/${req.file.filename}`,
      caption: caption || '',
      user: req.user._id,
    });

    res.status(201).json({
      message: 'Image Posted',
      image: newImage,
    });
  } catch (error) {
    res.status(500).json({
      error: error.message,
      message: 'Encountered Server Error',
    });
  }
}

// Deleating an image
const deleteImage = async(req, res)=>{

    const { id } = req.params

    if(!mongoose.Types.ObjectId.isValid(id)){
        return res.status(404).json({ 
            error: 'Invalid ID',
            message: "The ID given is invalid, please provide a valid mongoose ID"
        })
    }
    const deletedImageID = await Image.findByIdAndDelete(id);
    if(!deletedImageID){
        return res.status(404).json({
            error: 'Image not in Database',
            message: 'The Image you selected is not present in our database'
        })
    }
    res.status(202).json({
        message: "Image is deleted",
        image: deletedImageID
    })
}

// Updating an image
const updateImage = async(req, res)=>{
    const { id } = req.params
    const { caption } = req.body

    if (!mongoose.Types.ObjectId.isValid(id)) {
        return res.status(404).json({ 
            error: 'Invalid ID',
            message: "The ID given is invalid, please provide a valid mongoose ID"

        });
    }
    const updatedImageID = await Image.findByIdAndUpdate(
        id,
        {caption},
        {new: true}
    )
    if(!updatedImageID){
        return res.status(404).json({ 
            error: 'Image not found',
            message: 'Image is not found in our database, cannot update'
        })
    }
    res.status(200).json({
        message: 'Caption Updated',
        image: updatedImageID
    })
}

module.exports = {
    getImages,
    getImage,
    createImage,
    deleteImage,
    updateImage
}
```

`req.file` → added by multer (upload.single('image')). Contains details of the uploaded file (e.g. filename, originalname, mimetype, size).

`req.body.caption` → extra text data sent along with the file (like an image caption).

`req.user._id` → from your requireAuth middleware, links the image to the logged-in user.

`filename` → generated by multer (unique timestamp name), stored in DB so you can find it later.

`originalName` → the original file name from the user’s computer.

`url` → custom field you build (/uploads/< filename >) so the frontend knows where to fetch the image.

`id` (from req.params) → used in getImage, deleteImage, and updateImage to identify the specific image document in MongoDB.

`caption` → optional description field you allow users to update later.

`Image` → the Mongoose model for storing and querying uploaded images in MongoDB.

## Model

Create a folder witht the name `models`

### notesModel.js

```js
const mongoose = require('mongoose')

const Schema = mongoose.Schema

const notesSchema = new Schema({
    title:{
        type: String,
        required: true
    },
    description:{
        type: String,
        required: false
    },
    user_id:{
        type: String,
        required: true
    },
    pinned:{
        type: Boolean,
        default: false
    }
}, {timestamps: true})

module.exports = mongoose.model('Note', notesSchema)
```

Defines a Note model with fields:

- title → string, required.

- description → string, optional.

- user_id → string, required (owner of note).

- pinned → boolean, defaults to false.

- {timestamps: true} → auto-adds createdAt and updatedAt fields.

Other useful field types you can add in Mongoose:

- String (with options: minlength, maxlength, match for regex).

- Number (with min, max).

- Boolean (true/false).

- Date (custom date values).

- Array (e.g. tags: [String]).

- ObjectId (to reference another model).

- Mixed / Map (for flexible JSON-like objects).

- Enum (restrict string/number to specific values).


## userModel.js

```js
const mongoose = require('mongoose')
const bcrypt = require('bcrypt')
const validator = require('validator')

const Schema = mongoose.Schema

const userSchema = new Schema({
    name: {
        type: String,
        required: false,
        default: ''
    },
    email:{
        type: String,
        required: true,
        unique: true
    },
    password:{
        type: String,
        required: true
    }
})

//  static validations and method
// Login
userSchema.statics.login = async function(email, password){
    if(!email || !password){
        throw Error('All Fields must be filled')
    }
    
    const user = await this.findOne({email})

    if(!user){
        throw Error('Incorrect email')
    }
    const match = await bcrypt.compare(password, user.password)
    if(!match){
        throw Error('Incorrect password')
    }
    return {message: "User Loggedin successfully", user}
}


// Signup
userSchema.statics.signup = async function(email, password) {
    
    if(!email || !password){
        throw Error('All Fields must be filled')
    }
    if(!validator.isEmail(email)){
        throw Error('Email is not valid')
    }
    if(!validator.isStrongPassword(password)){
        throw Error('Password is not string enough')
    }

    const exist = await this.findOne({email})

    if (exist){
        throw Error('Email already exists')
    }

    // Hashing
    const salt = await bcrypt.genSalt(10)
    const hash = await bcrypt.hash(password, salt)

    const user = await this.create({email, password: hash})

    return {message: "User signed up successfully" , user}
}

module.exports = mongoose.model('User', userSchema)
```

Defines a User model with fields:

- name → optional string, defaults to empty.

- email → required, unique.

- password → required.

login static method:

- Checks if both fields exist.

- Finds user by email.

- Uses bcrypt.compare to check hashed password.

- Throws errors if invalid, else returns user + success message.

signup static method:

- Validates required fields.

- Uses validator to check email format and strong password.

- Ensures email is unique.

- Hashes password with bcrypt before saving.

- Creates new user and returns it with success message.

*Note -: if you want to add the name as well check out the code bellow*

```js
userSchema.statics.signup = async function(name, email, password) {
    if(!name || !email || !password){
        throw Error('All fields must be filled')
    }
    if(!validator.isEmail(email)){
        throw Error('Email is not valid')
    }
    if(!validator.isStrongPassword(password)){
        throw Error('Password is not strong enough')
    }
    
    const exist = await this.findOne({email})
    if (exist){
        throw Error('Email already in use')
    }

    const salt = await bcrypt.genSalt(10)
    const hash = await bcrypt.hash(password, salt)

    const user = await this.create({name, email, password: hash})
    return user
}
```

### Image/fileController.js
Optional model

```js
const mongoose = require('mongoose');

const ImageSchema = new mongoose.Schema({
  filename: {
    type: String,
    required: true,    
  },
  originalName: {
    type: String,
    required: true,
  },
  url: {
    type: String,
    required: true,
  },
  caption: {
    type: String,
    default: '',
  },
  uploadedAt: {
    type: Date,
    default: Date.now,
  },
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "User",
    required: true,
    },
});

module.exports = mongoose.model('Image', ImageSchema);
```

## Middleware

Create a folder with the name `middleware`

### requireAuth.js

```js
const jwt = require('jsonwebtoken')
const User = require('../models/userModel')

const requireAuth = async (req, res, next) =>{
    
    // verification auth
    const {authorization} = req.headers
    if(!authorization){
        return res.status(401).json({error: "Auth token is required"})
    }
    // Spliting Bearer token
    const token = authorization.split(' ')[1]

    try{
        const {_id} = jwt.verify(token, process.env.SECRET)

        req.user = await User.findOne({_id}).select('_id')
        next()
    }
    catch(error){
        console.log(error)
        res.status(401).json({error: 'Request is not authorized'})
    }
}

module.exports = requireAuth
```

- Pulls the Authorization header from the request (Bearer < token > format).

- If missing → returns 401 Unauthorized.

- Splits the header to extract the token string.

- Verifies the token with jwt.verify using your secret key.

- Extracts the user’s _id from the token and looks up the user in DB (only selects _id).

- Attaches the user to req.user so other controllers know who is logged in.

- Calls next() to continue if valid, else sends 401 Unauthorized.

*Note -: remember in `routes/notesRouter.js` we used*
```js
const requireAuth = require('../middleware/requireAuth')
```
*at the top of the routes, thats so that the requireAuth can run first and protect the api/notes routes from unauth requests*

### multerConfig.js (Optional)

middleware for app to use storage

```js
const multer = require('multer');
const path = require('path');

// Set storage engine to save files in /uploads with unique timestamp names
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, 'uploads/');
  },
  filename: function (req, file, cb) {
    const uniqueName = Date.now() + path.extname(file.originalname);
    cb(null, uniqueName);
  }
});

// Filter only image files
function fileFilter(req, file, cb) {
  const allowedTypes = /jpeg|jpg|png|gif/;
  const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
  const mimetype = allowedTypes.test(file.mimetype);

  if (mimetype && extname) {
    return cb(null, true);
  } else {
    cb(new Error('Only images are allowed!'));
  }
}

// Export the multer upload middleware
const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 } // max 5MB
});

module.exports = upload;
```

- Uses multer.diskStorage → saves files in /uploads with unique names (Date.now() + original extension).

- fileFilter → only accepts images (jpeg|jpg|png|gif) by checking MIME type + file extension.

- limits → restricts file size to 5MB.

- Exports upload so you can use it in routes (e.g. upload.single('image')).


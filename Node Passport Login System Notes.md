# Node.js Passport Login System

## BASIC TEMPLATE

*Initialize project `npm init -y`

*Install dependencies `npm i express ejs`

*Install Dev dependencies `npm i -D nodemon dotenv`

*Create a .env file and a .gitignore
(Add .env and node_modules to the gitignore file)

*In the package.json add the start script for nodemon to run
(`"devStart": "nodemon server.js"`)

*Create a server.js file

*In the console start the dev server: `npm run devStart`

*In the serverjs create a basic express server

```
const express = require('express')

const app = experss()

app.get("/", (res, req )=> {
    res.render("index.ejs")
})

app.listen(3000)
```
*Create a views folder in your project and add the index.ejs and add the following: `<h1>Hello</h1>`

*Navigate to the localhost:3000 and ensure that the h1 hello is there 

*To use EJS syntax the middleware must be added so in the server.js file add `app.use('view-engine', 'ejs')

* In the html add the ejs syntax to pull the variable name:
`<h1>Hello <%= name %></h1>`

*In the app.get function add the following:
`res.render("index.ejs", {name: "Joshua"} )`

*Navigate to the localhost:3000 and ensure that the h1 hello Joshua is there 


## ADDING ROUTES

*Create routes in the views folder for register and login

*Create a route in the server.js file for the login and register page
```
app.get("/login", (req, res ) => {
    res.render("login.ejs")
})

app.get("/register", (req, res ) => {
    res.render("register.ejs")
})

```

*Test the routes by adding the text login in the login page and register in the register page. Localhost:3000/login | Localhost:3000/register

*Add the following in register.ejs
```
<h1>Register</h1>
<form action="/register" method="POST">
    <div>
        <label for="name">NAME:</label>
        <input type="text" id="name" name="name" required>
    </div>

    <div>
        <label for="email">EMAIL:</label>
        <input type="text" id="email" name="email" required>
    </div>

    <div>
        <label for="password">PASSWORD:</label>
        <input type="text" id="password" name="password" required>
    </div>

    <button type="submit">Register</button>
</form>

<a href="/login">Login</a>
```

Add the following to login.ejs
```
<h1>Login</h1>
<form action="/login" method="POST">
    <div>
        <label for="email">EMAIL:</label>
        <input type="text" id="email" name="email" required>
    </div>

    <div>
        <label for="password">PASSWORD:</label>
        <input type="text" id="password" name="password" required>
    </div>

    <button type="submit">Login</button>
</form>

<a href="/register">Register</a>
```

*In the server.js add a post route for register and login
```
app.post("/register", (req, res ) => {

})
```

*In the server.js we need to add middleware to encode our url since we are using a form. This tells the the app to all us to access them in our req (request) POST calls ie: req.body.name 

`app.use(express.urlencoded({extended:false}))`

*Create a local variable of users so we do not have to access them via a database 
```
const users = [
    {
        
    }
]
```

*We need to hash users passwords with bcrypt add the dependency 
`npm i bcrypt` and import it at the top of the server.js `const bcrypt = require("bcrypt")`

*Lets create a new user with a hash password by adding try catch since we will be using asyncronus code:
```
app.post("/register", async (req, res) => {

    try {
        const hashedPassword = await bcrypt.hash(req.body.password, 10)
        users.push({
            id: Date.now(),
            name: req.body.name,
            email: req.body.email,
            password: req.body.hashedPassword
        })
        res.redirect("/login")
    } catch {
        res.redirect('/register')
    }
    console.log(users)
})
```
In the try block we will create a variable named hashPassword and allow bcrypt to hash the password that is passed through the form. Afterwards it will push a new user to the DATABASE (users variable) with a new id name email and password. The user will be redirected to the login page.

If there are any errors then it will redirect the user to the /register again.

(Users will be logged in the console to ensure information is correct. If you restart the server you will lose the user info)


*Lets start the login process with Passportjs and use the stratagy pass port local by adding the following in the terminal:
`npm i passport passport-local` this will help us with the process of logging in locally. We will also use other dependancies to store and presist users on multiple pages and allow messages to show if we fail to login `express-session and express-flash`

*Lets create a new file called passport-config.js so we dont over load the server.js

*Lets export a function that initializes passport related information. 

*Inside of our server we will require passport and require the initializePassport function and add passport (;ibrary) as the argument:

```
const passport = require("passport")

const app = express()

const initializePassport = require("./passport-config")
initializePassport(passport)
```

* Now we should be able to do all of our passport configurations inside of our initialize function. 

*To use the local stratagy for passport we need to require it ontop of the initializePassport function. `const LocalStrategy = require('passport-local').Strategy`


*Inside of the initialize function we will use the `passport.use` and pass in the new LocalStrategy and pass the usernameField option which is our email. we will create a function called authenticateUser and pass it. `    passport.use(new LocalStrategy({ usernameField: "email" }),
        authenticateUser)`

*Inside of the initialize function we will create an authenticateUser function that will take an email password and done arg. `const authenticateUser = (email, password, done) =>{}`

*We need to serialize (to store in the session )and deserialize our user by passing ` passport.serializeUser((user, done) => { })
    passport.deserializeUser((id, done) => { })` the serializing will take the user information and the deserialize will take the id since it will be created after the user is logged in

*Inside of the authenticateUser function we will create a variable named user and set it to getUserEmail and pass email as an arg. We will create an if statement that if the user is null then a message will show saying that the email is not existent. 

*We then will do a try catch statement to check if the password is correct or not by awaiting bcrypt and compare passport to our userPassword else the password is incorrect. If there are any errors we will show this in the catch method.

*We then need to export this module. The code in whole should look like this:

```
const LocalStrategy = require('passport-local').Strategy
const bcrypt = require(bcrypt)

function initialize(passport, getUserByEmail) {
    const authenticateUser = (email, password, done) => {
        const user = getUserEmail(email)
        if (user == null) {
            return done(null, false, {message: "No user with that email"})
        }
        try {
            if(await bcrypt.compare(passport, user.password)){

            }else{
                return done(null, false, {message: "Password incorrect"})
            }
        } catch (err) {
            return done(err)
        }
    }

    passport.use(new LocalStrategy({ usernameField: "email" }),
        authenticateUser)
    passport.serializeUser((user, done) => { })
    passport.deserializeUser((id, done) => { })
}

module.exports = initialize
```

*Back into our server.js we shoud navigate to the initializePassport and pass the function through email and add users.find where the email.email is equal to the email we passed.

```
initializePassport(
    passport, 
    email => users.find(user => user.email === email)
    )
```

*We need to ensure our app knows how to use passport by adding middle ware byt adding our flash messages and sessions. require express-flash and express-sessions at the top of the server.js. 
```
const flash = require("express-flash")
const session = require("express-session")

...

app.use(flash())
app.use(session())
```

*Session takes in options. the first on is secret which can be called by our enviornment variable. 
```
app.use(session({
    secret: process.env.SESSION_SECRET
}))
```

*Lets not forget to set our application to load the enviornment vatiables but going to the top of the file and adding the following: `if (process.env.NODE_ENV !== "production"){require("dotenv").config()}` This will load in all of our enviornment variables and set them in the process.env

*Back to the session options we add resave and set it to false if nothing is changed in the session then everything stays the same. We also add saveUninitialized which does not allow an empty value in the session.  

```
app.use(session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false
}))
```

*Now that we set up flash and sessions we can use passport by adding `app.use (passport.initialized)` and `app.use(passport.session())`

*We can now navigate to the post login function and remove the req, res and replace it with passport authentication middleware and have it use the local stratagy. we will pass it options such as redirect which goes to root and if it fails it will go back to failure and set the failure flash to true.
```
app.post("/login", passport.authenticate("local", {
    successRedirect: '/',
    failureRedirect: '/login',
    failureFlash: true
}))
```

*In order to have a flash message shown we need to navigate to the login.ejs and add the following at the top: 

```
<h1>Login</h1>
<% if (messages.error) { %>
    <%= messages.error %>
<% } %>
<form action="/login" method="POST">
...
```

*You should see some errors to which I believe is due to the calls made in passport config file. We created an await in the bcrypt.compare but did not use async on the authenticateUser we then need to go to fix the localStrategy that passes the authenticateUser to the Localstategy and not the passport.use. The fixed code should look like this:

```
const LocalStrategy = require('passport-local').Strategy
const bcrypt = require("bcrypt")

async function initialize(passport) {
    const authenticateUser = async (email, password, done) => {
        const user = getUserEmail(email)
        if (user == null) {
            return done(null, false, {message: "No user with that email"})
        }
        try {
            if(await bcrypt.compare(password, user.password)){

            }else{
                return done(null, false, {message: "Password incorrect"})
            }
        } catch (err) {
            return done(err)
        }
    }

    passport.use(new LocalStrategy({ usernameField: "email" },
        authenticateUser))
    passport.serializeUser((user, done) => { })
    passport.deserializeUser((id, done) => { })
}

module.exports = initialize
```

*We see another issue when testing this application out. When the password is entered we never serialized or deserialized the user 

*Inside of the serialize user we need to add the followng `    passport.serializeUser((user, done) => done(null, user.id))` which will save the users id to our session and null is for our error

*Inside of the deserialize user we need to add the followng `passport.deserializeUser((id, done) => {done(null, getUserById(id))})` which will get the user by id to deserrialize from our session. getUserById also needs to be passed in our initialize function as an argument. `async function initialize(passport, getUserByEmail, getUserById)`

* Lets then go to our server.js and add it to our initializePassport and pass in the id:
```
initializePassport(
    passport,
    email => users.find(user => user.email === email),
    id => users.find(user => user.id === id)
)
```

*Test logging in, It shoud show the default index.ejs page but we want to pass the actual name to the page. We can simply navigate to the `app.get('/')` portion of our code and change the name result from "Joshua" to req.user.name

*Now that we have this user information dynamic we need to protect our routes. Anyone can access the '/' route without being logged in. 

We need to create protected routes for authenticated users. 

*At the bottom of our server.js we can create a function called checkAuthenticated and it will check if the user is authenticated else if will redirect them to the login page. 



```
function checkAuthenticated(req, res, next){
    if(req.isAuthenticated()) {
        return next()
    }
    res.redirect('/login')
}
```

*In order to use this we can navigate to our login function and apply our authentication check to be called before rendering a request or response.

```
app.get("/", checkAuthenticated, (req, res) => {
    res.render("index.ejs", { name: req.user.name })
})

```



*We also need to make sure that the logged in user can have access to the login page if they are logged in so let check if they are not authenticated by creating a fucntion called checkNotAuthenticated we can apply this to the /login, /register so logged in users will be redirected to the root path

```

function checkNotAuthenticated(req, res, next){
    if(req.isAuthenticated()){
        return res.redirect('/')
    }
   next()
}

```

*Now we need to make a way to log out.

*We need to create an app.delete request that will call /logout and will call req.logout() and redirect the user to the login page. the logout function comes from the passport library

```
app.delete('/logout', (req, res) => {
    req.logout()
    res.redirect('/login')
})
```


*since html forms does not handle the delete request we need to use a dependancy called method override `npm i method-override` this will allow us to use the delete method .

*Lets include this at the top of our server.js and add this to our middleware
```
const methodOverride = require('method-override')
...
app.use(methodOverride())

* in the `app.use(methodOverride())` we need to pass in _method which will pass as our method
```
app.use(methodOverride('_method'))
```

*In our index.ejs we can add the following:
```
<h1>Hi <%= name %></h1>
<form action="/logout?_method=DELETE" method="POST">
  <button type="submit">Log Out</button>
</form>
```

This will allow is to make a POST method in our form but our action will be a request that is made to _method delete. 


*Now when we login, we will see a logout button that will allow us to log out and redirect us to the login page.

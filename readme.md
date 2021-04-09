## Authentication 
--

From Beginning to End

- Why Authentication?
    - Users will start generating data once your app starts to be utilized
    - In order to associate the Data we create with the user that created it, we need to create an account
    - We need authentication to password protect our accounts
    -  Restrict Access to a certain content is another reason why Authentication is needed
    - 

- Authentication can be done in a number of ways.
    - There are currently 6 levels of Security starting from the weakest to the strongest
    
- Setup
    - Copy Starter Files to Working Directory
    - npm init -y  
    - npm i express ejs body-parser
    - Once app.js looks like the code below, we can nodemon app.js to confirm it works
    - http://localhost:3000/

```
//jshint esversion:6
const express = require("express");
const bodyParser = require("body-parser");
const ejs = require("ejs");

const app = express();

app.use(express.static("public"));
app.set('view engine', 'ejs');
app.use(bodyParser.urlencoded({
    extended:true
}));

app.get("/", function(req, res){
    res.render("home")
})

app.get("/login", function(req, res){
    res.render("login")
})

app.get("/register", function(req, res){
    res.render("register")
})

app.listen(3000, function(){
    console.log("Successfully Connected to Port 3000")
})

```




## Level 1 Authentication
---

Register Users with a Username and Password

1. This is level 1, lowest level of security
2. Creating an account for our user and storing the pw in our DB


- We need to add a DB first. 
- Lets stop nodemon and npm i mongoose and create the const
- Lets call mongoose and connect to our db next
         ``mongoose.connect("mongodb://localhost:27017/userDB", {useNewUrlParser: true})``

- If we call upon nodemon app.js now we will get a failed to connect to server error
- We need to start up the mongoDB server to interact with it.
    - Run the command below to create the mongod alias (need to do it often with latest macOS)
    - ``alias mongod="sudo mongod --dbpath /Users/mcooper/data/db"``
    - mongod
- To interact with our DB we need to create a schema for our users
    - This needs to be an object with two properties
    ```
    const userSchema = {
    email: String,
    password: String
    };
    ```
- Next we need to create the User Model to interact with our Schema
    - Specify User in the singular form with the capital U
    - Create using the user Schema
    ``const User = new mongoose.model("User", userSchema)``

- Lets create app.post next with the register route. 
    - create a const newUser = new User({
        email: req.body.username,
        password: req.body.password
    })
    - notice username and password are the names of the input fields of our form
    - save the new user inside our post.
        - Add datavalidation(callback inside the save method) to make sure the save goes though

    ```
     newUser.save(function(err){
        if (err){
            console.log(err)
        } else {
            res.render("secrets")
        }
    });
    ```

    - Lets save and test our code out now
        - Use a simple email: 1@2.com as we have no data validation yet
        - Use a simple password, such as 123
        - If it renders your Secrets page, congratulations. You did it correctly so far.
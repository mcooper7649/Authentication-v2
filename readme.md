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
3. This stores the User Profile information in plaintext on the DB


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

- Lets create app.post next with the login route. 
    - This will take the information we put into the login input fields and check to see if we have a user with that profile
    - set username and password to the value of the input fields
    - Tap into the User collection and call findOne 
        - search parameters  email: username
        - Use a Data Validation callback to error handle the username
        - if foundUser, compare foundUser.password to the password input
        - if password matches res.render secrets


    ```
    app.post("/login", function (req, res){
    const username = req.body.username;
    const password = req.body.password;

        User.findOne({email: username}, function(err, foundUser){
            if (err){
                console.log(err);
            } else {
                if (foundUser){
                    if (foundUser.password === password){
                        res.render("secretes")
                    }
                }
            }
        })
    })
    ```
            
- Test your login now and seee if our registered user can login


## Level 2 Encryption
---


 As you can see from our previous module, we can see our passwords if we access the DB from robo3t. That's nice and all but you are suspectible to hackers. In our next module we will learn how to encrypt our data.

 Encryption has been used throughout to hide messages. This type of technology has evolved over the years.


 ### Installation
 ---

(Mongoose-Encryption)[https://preview.npmjs.com/package/mongoose-encryption] - NPM Package for Mongoose Encryption

1. Lets install Mongoose Encryption to get started 
    - Stop nodemon
    - npm install mongoose-encryption
    - ``const encrypt =  require("mongoose-encryption")``
    
2. If you view the Setup on the documentation, you will notice the encrypt plugin needs the userSchema to be a new mongoose.Schema

```
const userSchema =  new mongoose.Schema ({
    email: String,
    password: String
});
```
3. The documentation says we have two methods to which we can encrypt our DB
    - with an encryptionKey and a signingKey
    - OR with a secret (a long string) to encypt

4. Using the secret method we can set a password
    - call upon the encypt plugin for the mongoose schema userSchema, passing secret:secret
```
const secret = "Thisisourlittlesecret."
userSchema.plugin(encrypt, {secret: secret})
```

5. This is great if we are trying to encrypt all the user data, ie username and password but we want to just encrypt the password
    - Lets specify what field using the encryptedFields property
    - ``userSchema.plugin(encrypt, {secret: secret, encryptedFields: ["password"]})``
    - If you wanted to encrypt more fields you can just add more field names to the array with password

6. Mongoose Encryt is very nice because on the backend it automatically encrypts and decrypts for us.
    - When you call save() it will automatically encrypt
    - When you call fine() it will automatically decrypt

7. Test it out
    - Create another user a@b.com for example
    - Create another pw qwerty for example
    - We should be able to seee this new User Email
    - Password though is a very long binary string
    - Try to login using this user profile and confirm the findOne is succesffully decrypting


### Environment Variables
---

(.ENV-NPM-OFFICIAL)[https://www.npmjs.com/package/dotenv]

1. While you may have noticed, our const secret is stored on the servers code. So if someone gets ahold of this password they can use the same package to decrypt our DB and expose our users password.

2. Environment Variables let store secrets, apiKeys, or anything we want to store and keep them hidden from public repositories!
    - This can be crawled by bots
        - Bots look for AWS and other cloud platform dev keys so they can mine BTC.
    - This can then be used to decrypto your DB

3. dotenv is a NPM package that lets us utilize .ENV files
    - npm i dotenv
    - require ('dotenv').config()
        - This needs to be at the top of your app.js
        - This doesn't need an a const declared

4. How do we create a .env file?   
    - In our ROOT directory we need to create the .env
    - touch .env from command line

5. How to add environment variables to .env file?
    - Format for new entries is NAME = VALUE 
    ```
    DB_HOST=localhost
    DB_USER=root
    DB_PASS=s1mp13
    ```

6. Thats it!! But how to access?
    - process.env now has the keys and values you defined 
    

7. Lets give it a shot
    - Lets remove our secret and add to the .env file
        - Follow the same format
            - const secret = "Thisisourlittlesecret." // OLD
            - SECRET=Thisisourlittlesecret. // NEW
    - We should now be able to tap into our environment variables
        - console.log(process.env.SECRET)
            - You should now see two things in the console
                - The secret password
                - secret is not defined error

8. Lets fix the code for a .env file
    - line 29, try to tap into the secret password of the .env file
``userSchema.plugin(encrypt, {secret: process.env.SECRET, encryptedFields: ["password"]})``

9. 

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

9. ONE MORE THING. These files are great and all but they still don't hide themselves from being uploaded and viewed by Git.

(Git-IGNORE-TEMPLATE-FILES-NODE)[https://github.com/github/gitignore/blob/master/Node.gitignore]

    - We need to add a .gitignore file
        - In project root, touch .gitignore
        - Add template data from .gitnore node public repo, linked above
        
10. Secure your Data with .ENV and .GITIGNORE
    - It's best practice to create these files first
    

## Level 3 Hashing Passwords
---


Previosly we learned about encryption, taking a password and securing it using an encryption key. Then using a modern cypher method, like aes. Now we have a generated a ciphertext that will make it hard to decipher the DB. 

The problem with using just encryption is we have to have a encryption key. This will need to be stored somewhere and create a weak surface area for hackers to exploit. This is where Hashing comes to play.

- If there is no encryption key, how do we convert it back into plaintext? 
    - You don't
    - Hashed pws are designed to hash one-way so its virtually impossible, to hash a pw the other way
    
- How does it work?
    - Upon account creation and login the account credentials are hashed and encrypted/decrypted. 
    - This never stores the actual plaintext pw on the server
    - This doesn't allow admin to see the original PWs

### Installation
---

(npmjs-md5-home)[https://www.npmjs.com/package/md5]

1. Lets run install the npm package and configure by using the code snippet below

```
npm i md5
```

2. Next we need to remove the encryption configuration we setup on previous modules.

    - ``const encrypt =  require("mongoose-encryption")``
    - ``userSchema.plugin(encrypt, {secret: process.env.SECRET, encryptedFields: ["password"]})``

3. Now we can require md5 as per the documentation
    - const mpd5 = require('md5');

4. Then we need to add the md5 hashing function to to the register route
    - password: md5(req.body.password)
    - This converts it to an irreversable hash

5. Lets add to the login route next 
    - const password = md5(req.body.password);



### Hacking PWs 101
---

(haveibeenpwned)[https://www.haveibeenpwned.com]
(hackertyper)[hackertyper.net]

If a company lets you do a pw reset and they email you your pw back or generate a new pw that means they are either using the insecure encyrption key method or plaintext. 

When you hash a PW it will always end up with the same hash if the pw is the same. Hackers thought of a way of creating HASH tables, of the most commonly used pws.

Then a hacker can use their hash table and brute force it against a hashed DB of users.

- A popular and effective Hashtable consists of 
    - All the words from a dictionary
    - All the numbers from a telephone book
    - All combinations of characters up to 6 places

- This is about 19 billion pws, but computers can proccess them very fast

- Md5 is the most common type of hash table you will see on the web. You can even copy md5 hashes into google and you can find what pw it translates to, IF it's fairly common.

- Strong PWs are long and contain numbers and special characters
    - Using these techniques makes it rarely found on a hashtable



## Level 4 Salting and Hashing PWs with Bcrypt
---

- Salting allows us to avoid these types of Hash/Dictionary Attacks.
    - In addition to the pw, salting generates a random set of characters
    - These characters are then added to the pw then hashed
    - Users that have the same pw will have different hashes now because each salt is different
    - The user doesn't need to remember the salt
        - It will be stored in the DB
        - In DB, wouldn't store the actual PW, just the salt and the hash

    - MD5 can still be hashed at 20billion pws a second using some of the latest HW technology today.
    - BCRYPT algorythm can only be hashed at 17,000 pws a second.

    - Salt Rounds allow us to specify another of rounds to salt a pw
        - It will run a normal round of salt and create a new hash
        - Then it will take that Hash and salt it again with the same salt.
        - It will continue for as many rounds specified
        - This is convienient if you want to change your hash periodically, but don't want to change your password or modify your code.

### Bcrypt Setup
---

(BCRYPT-NPM)[https://www.npmjs.com/package/bcrypt]

``npm i bcrypt``  // Root Directory

- MD5 will no longer be used as BCRYPT replaces it. So we need to remove md5 references and add BCRYPT
    - removed const md5 = require("md5")

## REGISTER ROUTE
---    

- We will used Technique 2 (auto-gen a salt and hash)
    - This needs to be on Register Post Route
    ```
    const hash = bcrypt.hash(myPlaintextPassword, saltRounds);
    // Store hash in your password DB.
    ```
- Next We will need to add the new User into the callback
- Then replace the md5 with hash, like below
    - password: md5(req.body.password)
    - password: hash


    
```
app.post("/register", function (req, res){
    const hash = bcrypt.hash(req.body.password, saltRounds, function(err, hash){
        if (err){
            console.log(err)
        }else {
        const newUser = new User({
            email: req.body.username,
            password: hash
        });
        newUser.save(function(err){
            if (err){
                console.log(err)
            } else {
                res.render("secrets")
            }
        });
    }});

   
})
```

- If you search for the generated HASH now you shouldn't be able to find it on any search engine or hash table.


## LOGIN ROUTE
---

- We will need to remove the md5 code from the login first

OLD CODE

```

app.post("/login", function (req, res){
    const username = req.body.username;
    const password = md5(req.body.password);

    User.findOne({email: username}, function(err, foundUser){
        if (err){
            console.log(err);
        } else {
            if (foundUser){
                if (foundUser.password === password){
                    res.render("secrets")
                }
            }
        }
    })
})

```

- Next we will add the bcrypt compare method to the foundUser code in the login route.
    - our first option password is pulled from req.body.password
    - our 2nd option is the foundUser.password
    
- Bcrypt Compare method uses the callback err and res (for result) but that callback is the same as our res.render
    - Lets rename the callback Bcrypt uses by default to avoid any errors and confusion
- Try to login now and see if it works!



## Level 5 Cookies and Sessions
---

Cookies are data that are stored in the local browser about the user's interection with our platform. Thinks like session id, token, items in cart are all common cookies.

Cookies persist in your local browser unless you clear them.

Cookies can tell the server what html/css/js to render to the client as something like an item in cart, may be unique to their previous visit.

- There are lots of different types of cookies.
   - One common type of Cookies is the sesssion
        - This keeps track the moment you login, the cookie is created
        - Inside that Cookies, you will have your credentials and say you've successfully authenticated.
        

- What is Passport?
    - Passport is authentication middleware for Node.js
        - Can be dropped into any Express-based Web App
        - A comprehensive set of strategies support authentication using
            - Username/PW
            - Facebook, Twitter and more social logins

- Passport Installation and Setup
    - npm install
        - passport
        - passport-local
        - passport-local-mongoose
        - express-session      NOT express-sessions

- Remove BCRYPT and app.post login/register code

- BCrypt removed install code
```
const bcrypt = require('bcrypt');
const saltRounds = 10;
```

- Code for app.post login
```
 const username = req.body.username;
 const password = req.body.password;

    User.findOne({email: username}, function(err, foundUser){
        if (err){
            console.log(err);
        } else {
            if (foundUser){
                bcrypt.compare(password, foundUser.password, function(err, results){
                    if (err){
                        console.log(err)
                    } else {
                        res.render("secrets")
                    }
                })
                    
                }
            }
        }
    )
```

- Code for app.post register
```
const hash = bcrypt.hash(req.body.password, saltRounds, function(err, hash){
        if (err){
            console.log(err)
        }else {
        const newUser = new User({
            email: req.body.username,
            password: hash
        });
        newUser.save(function(err){
            if (err){
                console.log(err)
            } else {
                res.render("secrets")
            }
        });
    }});


```

- Now we can configure the installed packages
    - Express Session Setup
        - const session = require('express-session')
        - app.use(session({
        secret: "Our little Secret.",
        resave: "false",
        saveUninitialized: false
        }))
    - Passport Setup
        - const passport = require("passport")
        - app.use(passport.initialize());  // Needed for passport to function; Read the documentation if confused.
        - app.use(passport.session()); // Tells passport to use the session we previously configured
    - Passport Local Setup
        - const passportLocalMongoose = require("passport-local-mongoose")
        - This needs to be added as a PLUGIN, similiar to the encryption plugin
        - userSchema.plugin(passportLocalMongoose);
        - This taps into the UserSchema so it needs to be placed after
        
    - Passport uses SERIALIZE AND DESERIALIZE to encrypt and decrypt the session cookie
        - passport.serializeUser(User.serializeUser());
        - passport.deserializeUser(User.deserializeUser());
            - Needs to be added after the User model is created
        - Fix Deprecation Warning
            - mongoose.set("useCreateIndex", true);
        - passport.use(User.createStrategy());

    - Setup Register Post Route
        - passport-local-mongoose package to acheive this
        - Check documentation for setup
        ```
        app.post("/register", function (req, res){
            User.register({username: req.body.username}, req.body.password, function(err, user){
                if (err){
                    console.log(err)
                    res.redirect("/register")
                } else {
                    passport.authenticate("local")(req, res, function(){
                        res.redirect("/secrets")
                    })
                    
                }
            })
        
        })
        ```
    - add secrets route too as passport will need to redirect
            ```
            app.get("/secrets", function(req, res){
                if (req.isAuthenticated()){
                    res.render("secrets");
                } else {
                    res.redirect("login")
                }
            })
            ```


    - Setup Login Post Router
        ```
        app.post("/login", function (req, res){
        const user = new User({
            username: req.body.username,
            password: req.body.password
        })

        req.login(user, function(err){
            if (err){
                console.log(err)
            } else {
                passport.authenticate("local")(req, res, function(){
                    res.redirect("/secrets")
                })
            }
        })

        })
        ```
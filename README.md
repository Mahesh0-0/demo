<!-- dashboard.html -->
<!DOCTYPE html>
<html>
<head>
  <title>Dashboard</title>
</head>
<body>
  <h1>Welcome to the Dashboard</h1>
  <h2>Username: <%= username %></h2>
  <a href="/logout">Logout</a>
</body>
</html>


<!-- signup.html-->
<!DOCTYPE html>
<html>
<head>
  <title>Register</title>
</head>
<body>
  <h1>Register</h1>
  <form action="/register" method="POST">
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required><br>
    <label for="firstName">First Name:</label>
    <input type="text" id="firstName" name="firstName" required><br>
    <label for="lastName">Last Name:</label>
    <input type="text" id="lastName" name="lastName" required><br>
    <label for="password">Password:</label>
    <input type="password" id="password" name="password" required><br>
    <button type="submit">Register</button>
  </form>
  <p>Already have an account? <a href="/login">Login</a></p>
</body>
</html>


<!-- login.html -->
<!DOCTYPE html>
<html>
<head>
  <title>Login</title>
</head>
<body>
  <h1>Login</h1>
  <form action="/login" method="post">
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required><br>
    <label for="password">Password:</label>
    <input type="password" id="password" name="password" required><br>
    <button type="submit">Login</button>
  </form>
  <p>Don't have an account? <a href="/register">Register</a></p>
</body>
</html>


const express = require('express');
const fs = require('fs');
const path = require("path");
const bodyParser = require('body-parser');
const session = require('express-session');

const app = express();

app.use(bodyParser.urlencoded({ extended: true }));
// app.use(express.static('public'));
app.use(session({
    secret: 'secret-key',
    resave: false,
    saveUninitialized: false
}));
app.get('/login', (req, res) => {
    res.render(__dirname + '/login.ejs');
});
app.get('/signup', (req, res) => {
    res.render(__dirname + '/signup.ejs');
});
app.post('/register', (req, res) => {
    const { email, firstName, lastName, password } = req.body;
    const user = {
        email: email,
        firstName: firstName,
        lastName: lastName,
        password: password
    };
    const users = JSON.parse(fs.readFileSync('users.json', 'utf8'));
    const existingUser = users.find((u) => u.email === email);
    if (existingUser) {
        res.redirect('/register');
    } else {
        users.push(user);
        fs.writeFileSync('users.json', JSON.stringify(users));
        res.redirect('/login');
    }
});
app.post('/login', (req, res) => {
    const { email, password } = req.body;
    const users = JSON.parse(fs.readFileSync('./users.json', 'utf8'));
    const user = users.find((u) => u.email === email);

    if (user && user.password === password) {
        req.session.userId = user.email;
        res.redirect('/dashboard');
    } else {
        res.redirect('/login');
        console.log("Invalid");
    }
});
app.get('/dashboard', (req, res) => {
    if (req.session.userId) {
        const users = JSON.parse(fs.readFileSync('users.json', 'utf8'));
        const user = users.find((u) => u.email === req.session.userId);
        res.render('../dashboard.ejs', { username: user.firstName + ' ' + user.lastName });
    } else {
        res.redirect('/login');
    }
});
app.get('/logout', (req, res) => {
    req.session.destroy();
    res.redirect('/login');
});
app.listen(3000, () => {
    console.log('Server started on port 3000');
});

================================================================================================

<---routes--userroutes.js>
const express = require("express");
const route = express.Router();
route.use(express.urlencoded({extended:false}));
route.use(express.json());

const url = "mongodb+srv://u1161:u1161@cluster0.xgs5l6r.mongodb.net/?retryWrites=true&w=majority";

const client=require("mongodb").MongoClient;
let dbInstance;

client.connect(url).then(async (database)=>{
    console.log("Db connected");
    dbInstance =  await database.db("Account").collection("User");
}).catch((err)=>{
    console.log("Error in connecting"+err);
})

async function read(obj,res){
    let arr = await dbInstance.find(obj).toArray();
    if(arr.length==0)
    {
        console.log("Not founded");
        res.render("login",{msg:"Username Not found"});
    }
    else
    {
        console.log("Founded");
        res.render("login",{msg:"Successfully Login"});
    }
}
async function search(){
    let res = await dbInstance.find({}).toArray();
    console.log("Search compl");
}
async function insert(obj)
{
    let response = await dbInstance.insertOne(obj)
    console.log("Data inserted"+response);
}
async function delte(obj)
{
    let res = await dbInstance.deleteMany(obj);
    console.log("Delted");
}

async function update(obj)
{
    let res = await dbInstance.updateOne(
        {
            username:obj.username
        },
        {
            $set:{pwd:obj.pwd}
        }
    );
    console.log("Delted");
}


async function check(obj,res){
    let arr = await dbInstance.find(obj).toArray();
    if(arr.length==0)
    {
        insert(obj);
        res.render("signup",{msg:"User created"});
    }
    else
    {
        console.log("User already present");
        res.render("signup",{msg:"User already present"});
    }
}
route.get("/act1",(req,res)=>{
    console.log(req.query);
    let obj = {};
    obj.username = req.query.username;
    obj.pwd = req.query.pwd;
    console.log(obj);
    read(obj,res);
})

route.get("/replace",(req,res)=>{
    res.render("update");
})


route.get("/destroy",(req,res)=>{
    res.render("delete");
})

route.get("/login",(req,res)=>{
  console.log(req.query);
    res.render("login",{msg:""});
})


route.get("/signup",(req,res)=>{
  console.log(req.query);
    res.render("signup",{msg:""});
})
route.post("/act2",(req,res)=>{
    console.log(req.body);
    let obj = {};
    obj.username = req.body.username;
    obj.pwd = req.body.pwd;
    console.log(obj);
    check(obj,res);
})
route.get("/delete",(req,res)=>{
    let obj = {};
    obj.username = req.query.username;
    obj.pwd = req.query.pwd;
    console.log(obj);
    delte(obj);
    res.end("User deleted");
})
route.get("/update",(req,res)=>{
    let obj = {};
    obj.username = req.query.username;
    obj.pwd = req.query.pwd;
    console.log(obj);
    update(obj);
    res.end("User updated");
})
module.exports = route;

<-------views--------->
//delete.ejs
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <!-- <link href="delete.css" type="text/css" rel="stylesheet"> -->
</head>
<body>
    <div id="title">
        <h1>Delete Page</h1>
    </div>

    <div id="main">
        <form action="/delete" method="get">
            <!-- Username -->
            <p>Username</p>
            <input type="text" name="username" placeholder="Enter username">

            <!-- Password -->
            <p>Password</p>
            <input type="password" name="pwd" placeholder="Enter password">

            <!-- Submit -->
            <p>
                <input type="submit">
            </p>
            
        </form>
    </div>

</body>
</html>

//home.ejs

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <!-- <link href="style.css" type="text/css" rel="stylesheet"> -->
</head>
<body>
    <div id="title">
        <h1>Welcome to mongodb</h1>
    </div>
    

    <div id="main">
        <div id="left">
            <a href="/login"><input type="button" value="Login" id="login"></a>
        </div>

        <div id="right">
            <a href="/signup"><input type="button" value="Sign Up" id="signup"></a>
        </div>

        <div>
        <a href="/destroy">Delete User</a>
        </div>

        <div>
        <p>--------------</p>
        </div>

        <div>
        <a href="/replace">Update User</a>
        </div>
    </div>
</body>
</html>

//login.ejs
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <!-- <link href="login.css" type="text/css" rel="stylesheet"> -->
</head>
<body>
    <div id="title">
        <h1>Login Page</h1>
    </div>

    <div id="main">
        <form action="/act1" method="get">
            <p>Username</p>
            <input type="text" name="username" placeholder="Enter username">
            <p>Password</p>
            <input type="password" name="pwd" placeholder="Enter password">
            <p>
                <input type="submit">
            </p>

            <p>
            <%=msg%>
            </p>
            
        </form>
    </div>

</body>
</html>

//signup.ejs
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <!-- <link href="signup.css" type="text/css" rel="stylesheet"> -->
</head>
<body>
    <div id="title">
        <h1>Sign Up Page</h1>
    </div>

    <div id="main">
        <form action="/act2" method="post">
            <!-- Username -->
            <p>Username</p>
            <input type="text" name="username" placeholder="Enter username">

            <!-- Password -->
            <p>Password</p>
            <input type="password" name="pwd" placeholder="Enter Password">

            <!-- Submit -->

            <p>
                <input type="submit">
            </p>

            <p>
            <%=msg%>
            </p>
        </form>
    </div>

</body>
</html>
  
const express = require('express');
const path = require('path');
const multer=require('multer');
// const upload=multer({dest:"uploads/"}) if we use this file gets corrupted
const storage=multer.diskStorage({
    destination:function(req,file,cb) {
        return cb(null,'./uploads');
    },
    filename:function(req,file,cb){
        return cb(null,`${Date.now()}-${file.originalname}`)
    }
});
const upload=multer({storage})

const app=express();
app.use(express.static('public'));
app.use(express.urlencoded({extended:false}))

app.get('/',(req,res)=>{
    // res.sendFile(path.resolve(__dirname,'./public/home.ejs'));
    res.render(path.resolve(__dirname,"./public/home.ejs"));      
})

app.post('/upload',upload.single("profileImage"),(req,res)=>{
    console.log(req.body);
    console.log(req.file);
    console.log(req.file.path);
    res.redirect("/");
})

app.listen(3000,(err)=>{
    if(err){
        console.log(err);
    }
    else{
        console.log("Server started successfully");
    }
})

//update.ejs
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <!-- <link href="update.css" type="text/css" rel="stylesheet"> -->
</head>
<body>
    <div id="title">
        <h1>Update Page</h1>
    </div>

    <div id="main">
        <form action="/update" method="get">
            <p>Username</p>
            <input type="text" name="username" placeholder="Enter username">
            <p>New Password</p>
            <input type="password" name="pwd" placeholder="New password">
            <p>
                <input type="submit">
            </p>
            
        </form>
    </div>

</body>
</html>

//main.ejs;
const port = 3000;

const express = require("express");

const app = express();

// import userroute

const userroute = require("./routes/userroutes.js");

// using userroute

app.use(userroute);

// Setting ejs
app.set("view engine","ejs");

// Using body parser

app.use(express.urlencoded({extended:false}));
app.use(express.json());

// Maintinag route

app.get("/",(req,res)=>{
    res.render("home");
})

// Handling static file of ejs
app.use(express.static("views"));

app.listen(port,(err)=>{
    if(err){
        console.log("Error in starting database");
    }
    else{
        console.log("Sever started");
    }
})


================================================================
->Routes
    //userroute.js
const express = require("express");
const Users = require("./userModel");
const route = express.Router();
route.use(express.urlencoded({extended:false}));
route.use(express.json());
const mongoose = require("mongoose");
// const userModel = require("./models/userModel");
const url = "mongodb+srv://u1161:<>@cluster0.uqtm8fg.mongodb.net/?retryWrites=true&w=majority";
mongoose.connect(url)
.then((database)=>{
    console.log("Connected to database");
}).catch((err)=>{
    console.log("Error in connecting database"+err);
})

route.get("/login",(req,res)=>{
    res.render("login");
})

route.get("/signup",(req,res)=>{
    res.render("signup",{msg:""});
})

route.post("/login",(req,res)=>{
    let obj  = {};
    obj.username = req.body.username;
    obj.pwd = req.body.pwd;
    console.log(obj);
    res.end("Data submitted");
})

route.post("/signup",(req,res)=>{
    let obj  = {};
    obj.username = req.body.username;
    obj.email  = req.body.username;
    obj.pwd = req.body.pwd;
    obj.data = req.body.date;
    obj.userAgent = req.body.userAgent;
    console.log(obj);

    let user = new Users(obj);
    user
      .save()
      .then(async (result) => {
        console.log(result);
        res.end("Submitted");
      })
      .catch(async (err) => {
        console.log("Error in saving " + err);
        res.end("Error");
      });
    

})

module.exports = route;

    //userModel.js
const mongoose=require("mongoose");
// const userModel=mongoose.Schema({
//     email:{
//         type:String,
//         required:true}
//         ,
//     password:{
//         type:String,
//         default:'password'
//     },
//     firstName:String,
//     lastName:String

// })

const userModel=mongoose.Schema({
    username:String,
    email:String,
    password:String,
    date:Date,
    UserAgent:String
})
// userModel.virtual("FullName").get(()=>{
//     return this.firstName+" "+this.lastName;
// })

module.exports=mongoose.model("users",userModel)

->views
    //style.css
#title{
    height: 100px;
    margin: auto;
    display: flex;
    background-color: black;
    color: white;
    text-align: center;
    justify-content: center;
}


    //home.ejs
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="style.css" type="text/css" rel="stylesheet">
    <title>Document</title>
</head>
<body>
    <div id="title">
        <h1>Mongo DB</h1>
    </div>

    <div id="main">
        <div id="left">
            <a href="/login">Login</a>
        </div>

        <div id="right">
            <a href="/signup">SignUp</a>
        </div>
    </div>
</body>
</html>
    
    //login.ejs
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="title">
        <h1>Login  Page</h1>
    </div>

    <div id="main">
        <form action="login" method="post">
            UserName<input type="text" name="username"/><br/>
            Paswword<input type="password" name="pwd"/><br/>
            <input type="submit"/>
        </form>
    </div>
</body>
</html>

    //signup
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="title">
        <h1>SignUp Page</h1>
    </div>

    <div id="main">
        <form action="/signup" method="post">
            UserName<input type="text" name="username"/><br/>
            Email<input type="email" name="email"/><br/>
            Paswword<input type="password" name="pwd"/><br/>
            LoginHistroy
            <br/>
            Data<input type="date" name="date"/><br/>
            UsrAgent<input type="text" name="UserAgent"/><br/>
            <input type="submit"/>
            <p><%=msg%></p>
        </form>
    </div>
</body>
</html>


--index.js
const express = require("express");

const port = 3000;

const app = express();

app.set("view engine","ejs");

const userroute = require("./routes/userroute");

app.use(userroute);

app.use(express.static("views"));

app.use(express.urlencoded({extended:false}));

app.use(express.json());

app.get("/",(req,res)=>{
    res.render("home");
})
app.listen(port,(err)=>{
    if(err){
        console.log("Error in connecting to database");
    }else{
        console.log("Server Started");
    }
})
 ==================================================================================================
  <!DOCTYPE html>
<html>
<head>
  <title>Login</title>
</head>
<body>
  <h1>Login</h1>
  <form method="POST" action="/login">
    <label for="username">Username:</label>
    <input type="text" id="username" name="username" required><br><br>
    <label for="password">Password:</label>
    <input type="password" id="password" name="password" required><br><br>
    <button type="submit">Log In</button>
  </form>
</body>
</html>

  
  const express = require('express');
const session = require('express-session');

const app = express();

// Set up session middleware
app.use(
  session({
    secret: 'your_secret_key',
    resave: false,
    saveUninitialized: false
  })
);

// Mock user database
const users = [
  { id: 1, username: 'user123', password: 'password123' },
  { id: 2, username: 'anotheruser', password: 'testpass' }
];

// Middleware
const authMiddleware = (req, res, next) => {
  if (req.path === '/dashboard' && !req.session.userid) {
    return res.redirect('/login'); // Redirect to login page if user is not logged in
  }
  next();
};

// Apply middleware to endpoint
app.use(authMiddleware);

// Login route
app.post('/login', (req, res) => {
  const { username, password } = req.body;

  // Find user in the mock database
  const user = users.find(u => u.username === username && u.password === password);

  if (!user) {
    return res.send('Invalid credentials');
  }

  // Set session variables
  req.session.userid = user.id;

  res.send('Logged in successfully!');
});

// Dashboard route
app.get('/dashboard', (req, res) => {
  const userId = req.session.userid;

  // Fetch user data from the database based on the userId

  res.send(`Welcome to the dashboard, User ${userId}!`);
});

// Logout route
app.get('/logout', (req, res) => {
  req.session.destroy();
  res.send('Logged out successfully!');
});

// Start the server
app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
==========================================================

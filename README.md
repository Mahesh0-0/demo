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





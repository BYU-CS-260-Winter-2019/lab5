# Part 4: Authentication -- Back End

Users need to register for an account and login. We'll create the back end
portion of this here.

We will use the same code from our activity on user authentication, with some
small modifications. Please see that activity for explanations of this code.

The primary change to this code is to extend our user model to include a real
name. You'll notice a change to the schema and to the export statement.

The second change is to use middleware that validates a user's account,
even if the token is valid. For example, we may have issued a token for a login,
but then deleted the account before the token expired. You can find this in
the `verify` static method that is defined on the user schema.

## users.js

Create a file called `server/users.js` and put the following there:

```
const mongoose = require('mongoose');
const bcrypt = require("bcrypt");
const express = require("express");
const router = express.Router();
const auth = require("./auth.js");

const SALT_WORK_FACTOR = 10;

const userSchema = new mongoose.Schema({
  username: String,
  password: String,
  name: String,
  tokens: [],
});

userSchema.pre('save', async function(next) {
  // only hash the password if it has been modified (or is new)
  if (!this.isModified('password'))
    return next();

  try {
    // generate a salt
    const salt = await bcrypt.genSalt(SALT_WORK_FACTOR);

    // hash the password along with our new salt
    const hash = await bcrypt.hash(this.password, salt);

    // override the plaintext password with the hashed one
    this.password = hash;
    next();
  } catch (error) {
    console.log(error);
    next(error);
  }
});

userSchema.methods.comparePassword = async function(password) {
  try {
    const isMatch = await bcrypt.compare(password, this.password);
    return isMatch;
  } catch (error) {
    return false;
  }
};

userSchema.methods.toJSON = function() {
  var obj = this.toObject();
  delete obj.password;
  delete obj.tokens;
  return obj;
}

userSchema.methods.addToken = function(token) {
  this.tokens.push(token);
}

userSchema.methods.removeToken = function(token) {
  this.tokens = this.tokens.filter(t => t != token);
}

userSchema.methods.removeOldTokens = function() {
  this.tokens = auth.removeOldTokens(this.tokens);
}

// middleware to validate user account
userSchema.statics.verify = async function(req, res, next) {
  // look up user account
  const user = await User.findOne({
    _id: req.user_id
  });
  if (!user || !user.tokens.includes(req.token))
    return res.clearCookie('token').status(403).send({
      error: "Invalid user account."
    });

  req.user = user;

  next();
}

const User = mongoose.model('User', userSchema);

// create a new user
router.post('/', async (req, res) => {
  if (!req.body.username || !req.body.password || !req.body.name)
    return res.status(400).send({
      message: "Name, username, and password are required."
    });

  try {

    //  check to see if username already exists
    const existingUser = await User.findOne({
      username: req.body.username
    });
    if (existingUser)
      return res.status(403).send({
        message: "That username already exists."
      });

    // create new user
    const user = new User({
      username: req.body.username,
      password: req.body.password,
      name: req.body.name
    });
    await user.save();
    login(user, res);
  } catch (error) {
    console.log(error);
    return res.sendStatus(500);
  }
});

// login
router.post('/login', async (req, res) => {
  if (!req.body.username || !req.body.password)
    return res.status(400).send({
      message: "Username and password are required."
    });

  try {
    //  lookup user record
    const existingUser = await User.findOne({
      username: req.body.username
    });
    if (!existingUser)
      return res.status(403).send({
        message: "The username or password is wrong."
      });

    // check password
    if (!await existingUser.comparePassword(req.body.password))
      return res.status(403).send({
        message: "The username or password is wrong."
      });

    login(existingUser, res);

  } catch (error) {
    console.log(error);
    return res.sendStatus(500);
  }
});

async function login(user, res) {
  let token = auth.generateToken({
    id: user._id
  }, "24h");

  user.removeOldTokens();
  user.addToken(token);
  await user.save();

  return res
    .cookie("token", token, {
      expires: new Date(Date.now() + 86400 * 1000)
    })
    .status(200).send(user);
}

// Logout
router.delete("/", auth.verifyToken, User.verify, async (req, res) => {
  req.user.removeToken(req.token);
  await req.user.save();
  res.clearCookie('token');
  res.sendStatus(200);
});

// Get current user if logged in.
router.get('/', auth.verifyToken, User.verify, async (req, res) => {
  return res.send(req.user);
});

module.exports = {
  model: User,
  routes: router,
}
```

## auth.js

Create a file called `server/auth.js` and put the following there:

```
const mongoose = require("mongoose");
const jwt = require("jsonwebtoken");

// We define a random secret here to use for signing JWTs
// You should NOT do this normally. You don't want to hard code
// secret values into your code.
let secret = "RANDOMSECRETCHANGETHIS";

// Instead, you should define the value in a file called ".env".
// Then call "source .env" to put this into the environment
// This file should have in it:
// export jwtSecret="RANDOMSECRETCHANGETHIS"
// We would read this secret with the lne below:

// let secret = process.env.jwtSecret;

if (secret === undefined) {
  console.log("You need to define a jwtSecret environment variable to continue.");
  mongoose.connection.close();
  process.exit();
}

// Generate a token
const generateToken = (data, expires) => {
  return jwt.sign(data, secret, {
    expiresIn: expires
  });
};

// Verify the token that a client gives us.
// This is setup as middleware, so it can be passed as an additional argument to Express after
// the URL in any route. This will restrict access to only those clients who possess a valid token.
const verifyToken = (req, res, next) => {
  const token = req.cookies["token"];
  if (!token) return res.status(403).send({
    message: "No token provided."
  });
  try {
    const decoded = jwt.verify(token, secret);
    // save user id
    req.user_id = decoded.id;
    req.token = token;
    next();

  } catch (error) {
    console.log(error);
    return res.status(403).send({
      message: "Failed to authenticate token."
    });
  }
}

const removeOldTokens = (tokens) => {
  return tokens.filter(token => {
    try {
      jwt.verify(token, secret);
      return true;
    } catch (error) {
      return false;
    }
  });
}

module.exports = {
  generateToken: generateToken,
  verifyToken: verifyToken,
  removeOldTokens: removeOldTokens,
};
```

## server.js

Finally, modify `server/server.js` so it contains the following, before
`app.listen`.

```
const users = require("./users.js");
app.use("/api/users", users.routes);
```

Go to [Part 5](/tutorials/part5.md).

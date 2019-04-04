# Part 3: Server Setup

We need to setup a server for the back end of this project. In the top level of
this project, install some packages we'll use:

```
npm install express mongoose bcrypt jsonwebtoken cookie-parser multer
```

Make a directory called `server`, and in that directory, create a file
called `server.js`. Add the following content to `server/server.js`:

```
const express = require('express');
const bodyParser = require("body-parser");

const app = express();
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({
  extended: false
}));

const mongoose = require('mongoose');

// connect to the database
mongoose.connect('mongodb://localhost:27017/photobomb', {
  useNewUrlParser: true
});

const cookieParser = require("cookie-parser");
app.use(cookieParser());

app.listen(3001, () => console.log('Server listening on port 3001!'));
```

This sets up Express, the body parser middleware, and Mongo. See previous
activities for explanations of these.

You also need to create a file in the top level of this project called `vue.config.js`, containing the following:

```
module.exports = {
  devServer: {
    proxy: {
      '^/api': {
        target: 'http://localhost:3001',
      },
    }
  }
}
```

This lets the webpack development server that is started by `npm run serve` proxy the requests for the API and send them to your node server.

Note we are using a different port this time. This will ensure it does not
conflict with your previous projects.

Go to [Part 4](/tutorials/part4.md).

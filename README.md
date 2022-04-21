# MERN Stack Full Tutorial | User authentication, JWT

- Development = Node.js server + React server

- Production = Node.js server + static react files

## Install React app

- `npx create-react-app client`

- Remove git init from CRA

  - ```bash
    cd client
    rm -rf .git
    ```

## Install Node app

- ```bash
  cd server
  npm init -y

  npm install express
  npm install nodemon --save-dev
  ```

- On `package.json`

  - ```json
    "scripts": {
      "dev": "nodemon index.js"
    }
    ```

- `npm run dev`

- On `/server/index.js`

  - ```js
    const express = require('express');
    const app = express();

    app.get('/hello', (req, res) => {
      res.send('hello world!');
    });

    app.listen(1337, () => {
      console.log('Server started on 1337 port');
    });
    ```

- You can see `hello world!` on browser, `localhost:1337/hello`

## Initialize Git

- `git init`

- Create `/.gitignore`

  - ```
    /server/node_modules/

    # From CRA
    # dependencies
    /client/node_modules
    /client/.pnp
    /client/.pnp.js

    # testing
    /client/coverage

    # production
    /client/build

    # misc
    /client/.DS_Store
    /client/.env.local
    /client/.env.development.local
    /client/.env.test.local
    /client/.env.production.local

    /client/npm-debug.log*
    /client/yarn-debug.log*
    /client/yarn-error.log*
    ```

## Register Page

- On `client/src/App.js`

  - ```js
    import { useState } from 'react';
    import './App.css';

    function App() {
      const [name, setName] = useState('');
      const [email, setEmail] = useState('');
      const [password, setPassword] = useState('');

      async function registerUser(event) {
        event.preventDefault();
        const response = await fetch('http://localhost:1337/api/register', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            name,
            email,
            password,
          }),
        });
        const data = await response.json();

        console.log(data);
      }

      return (
        <div>
          <h1>Register</h1>
          <form onSubmit={registerUser}>
            <input
              value={name}
              onChange={(e) => setName(e.target.value)}
              type='text'
              placeholder='Name'
            />
            <br />
            <input
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              type='email'
              placeholder='Email'
            />
            <br />
            <input
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              type='password'
              placeholder='Password'
            />
            <br />
            <input type='submit' value='Register' />
          </form>
        </div>
      );
    }

    export default App;
    ```

- On `server/index.js`

  - ```js
    ...
    const cors = require('cors');

    app.use(cors());          // to solve blocked by CORS
    app.use(express.json());  // body data is json

    app.post('/api/register', (req, res) => {
      console.log(req.body);
      res.json({ status: 'ok' }); // the result can see on Network / register / Preview of devtool
    });

    ...
    ```

### Error fix

- Uncaught (in promise) TypeError: Failed to execute 'fetch' on 'Window': Request with GET/HEAD method cannot have body.

  - Add `method: 'POST',` to the `fetch` on `client/src/App.js`

- Access to fetch at 'http://localhost:1337/api/register' from origin 'http://localhost:3000' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

  - Add `cors` to `server/index.js`

    - `npm install cors` on server

    - ```js
      const cors = require('cors');
      app.use(cors());
      ```

    - `cors` is a node.js package for providing a Connect/Express middleware that can be used to enable CORS with various options.

    - [CORS (Cross-Origin Resource Sharing)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) is an HTTP-header based mechanism that allows a server to indicate any origins (domain, scheme, or port) other than its own from which a browser should permit loading resources. CORS also relies on a mechanism by which browsers make a "preflight" request to the server hosting the cross-origin resource, in order to check that the server will permit the actual request. In that preflight, the browser sends headers that indicate the HTTP method and headers that will be used in the actual request.

## Install MongoDB

- [Download MongoDB Installer](https://www.mongodb.com/try/download/community?tck=docs_server) - the Community `.msi` installer from the link and execute the file.

  - Uncheck "Install MongoD as a Service" to do not configure MongoDB as a Windows service because I will run it locally without network connectivity.

- Set up the MongoDB environment

  - MongoDB requires a data directory to store all data. MongoDB’s default data directory path is the absolute path \data\db on **the drive from which you start MongoDB**. Create this folder by running the following command **in a Command Prompt**:

    - `md \mongodb\data`

    - Copy the location of mongo.exe, `C:\Program Files\MongoDB\Server\5.0\bin`

    - To make it easier to run server in future, press search and type `environment variables`

      - Click `Edit the system environment variables`

      - Push `Environment Variables` button

      - Under `System variables`, double click on `path`

      - Click `New` and paste the location

      - Hit `Ok`

- Start MongoDB: `mongod.exe` on cmd

- Connect MongoDB: `mongo.exe` on new cmd

  - To disconnect MongoDB, `> quit()`

- You can use MongoDB Compass for management it

  - Compass is an interactive tool for querying, optimizing, and analyzing your MongoDB data.

## Register User to MongoDB

- ```bash
  cd server
  npm install mongoose
  ```

- Create `server/models/user.model.js`

  - ```js
    const mongoose = require('mongoose');
    const User = new mongoose.Schema(
      {
        name: { type: String, required: true },
        email: { type: String, required: true, unique: true },
        password: { type: String, required: true },
        quote: { type: String },
      },
      // `user-data` is collection name
      { collection: 'user-data' }
    );

    const model = mongoose.model('UserData', User);

    module.exports = model;
    ```

- On `server/index.js`

  - ```js
    ...
    const mongoose = require('mongoose');
    const User = require('./models/user.model');

    ...
    // `mern-user-auth` is database name
    mongoose.connect('mongodb://localhost:27017/mern-user-auth');

    app.post('/api/register', async (req, res) => {
      try {
        await User.create({
          name: req.body.name,
          email: req.body.email,
          password: req.body.password,
        });
        res.json({ status: 'ok' });
      } catch (error) {
        res.json({ status: 'error', error: 'Duplicate email' });
      }
    });

    app.listen(1337, () => {
      console.log('Server started on 1337 port');
    });
    ```

## Check if user exists on MongoDB

- On `server/index.js`

  - ```js
    ...

    app.post('/api/login', async (req, res) => {
      const user = await User.findOne({
        email: req.body.email,
        password: req.body.password,
      });

      if (user) {
        return res.json({ status: 'ok', user: true });
      } else {
        return res.json({ status: 'error', error: 'Duplicate email' });
      }
    });

    ...
    ```

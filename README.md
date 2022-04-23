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
        return res.json({ status: 'error', user: false });
      }
    });

    ...
    ```

## Create Login Page and Register Page

- Move `client/src/App.js` to `client/src/pages/Register.js`

- Create `client/src/pages/Login.js`

  - ```js
    import { useState } from 'react';

    function Login() {
      const [email, setEmail] = useState('');
      const [password, setPassword] = useState('');

      async function loginUser(event) {
        event.preventDefault();
        const response = await fetch('http://localhost:1337/api/login', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            email,
            password,
          }),
        });
        const data = await response.json();

        console.log(data);
      }

      return (
        <div>
          <h1>Login</h1>
          <form onSubmit={loginUser}>
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
            <input type='submit' value='login' />
          </form>
        </div>
      );
    }

    export default Login;
    ```

- Install `react-router-dom` on `client`

  - `npm i react-router-dom`

- Create `client/src/App.js`

  - ```js
    import React from 'react';
    import { BrowserRouter, Routes, Route } from 'react-router-dom';
    import Register from './pages/Register';
    import Login from './pages/Login';

    const App = () => {
      return (
        <div>
          <BrowserRouter>
            <Routes>
              <Route path='/login' element={<Login />} />
              <Route path='/register' element={<Register />} />
            </Routes>
          </BrowserRouter>
        </div>
      );
    };

    export default App;
    ```

- Modify `client/src/index.js`

  - ```js
    import React from 'react';
    import ReactDOM from 'react-dom/client';
    import App from './App';

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(
      <React.StrictMode>
        <App />
      </React.StrictMode>
    );
    ```

## Use JWT for User's Authentication

- ```bash
  cd server
  npm i jsonwebtoken
  ```

- On `server/index.js`

  - ```js
    ...
    const jwt = require('jsonwebtoken');

    ...

    app.post('/api/login', async (req, res) => {
      ...

      if (user) {
        const token = jwt.sign(
          {
            name: user.name,
            email: user.email,
          },
          'secret123'
        );
        return res.json({ status: 'ok', user: token });

    ...
    ```

  - The token will be `AAAAAA.BBBBBBBB.CCCCCC`. You can check the user info on the Console of devtool

    - `atob("BBBBBBBB")` => `'{"name":"tester","email":"test@email.com","iat":1650596116}'`

    - `new Date(1650596116000)` => `Thu Apr 21 2022 22:55:16 GMT-0400 (Eastern Daylight Time)`

## Set Route and Save token to LocalStorage

- On `client/src/pages/Register.js`

  - ```js
    import { useNavigate } from 'react-router-dom';

    function Register() {
      const navigate = useNavigate();
      ...
        if (data.status === 'ok') {
          navigate('/login');
        }
    ```

- On `client/src/pages/Login.js`

  - ```js
    import { useNavigate } from 'react-router-dom';

    function Login() {
      const navigate = useNavigate();
      ...
        if (data.user) {
          localStorage.setItem('token', data.user);
          alert('Login Successful');
          navigate('/dashboard');
        } else {
          alert('Please check you email and password');
        }
    ```

## Use JWT for Getting User's Quote and the Saving

- On `server/index.js`

  - ```js
    app.get('/api/quote', async (req, res) => {
      const token = req.headers['x-access-token'];
      try {
        const decoded = jwt.verify(token, 'secret123');
        const email = decoded.email;
        const user = await User.findOne({ email: email });

        return res.json({ status: 'ok', quote: user.quote });
      } catch (error) {
        console.log(error);
        res.json({ status: 'error', error: 'invalid token' });
      }
    });

    app.post('/api/quote', async (req, res) => {
      const token = req.headers['x-access-token'];
      try {
        const decoded = jwt.verify(token, 'secret123');
        const email = decoded.email;
        await User.updateOne(
          { email: email },
          { $set: { quote: req.body.quote } }
        );
        return res.json({ status: 'ok' });
      } catch (error) {
        console.log(error);
        res.json({ status: 'error', error: 'invalid token' });
      }
    });
    ```

- On `client/src/App.js`

  - ```js
    import Dashboard from './pages/Dashboard';

    const App = () => {
      ...
              <Route path='/dashboard' element={<Dashboard />} />
    ```

- ```bash
  cd client
  npm i jwt-decode
  ```

- Create `client/src/pages/Dashboard.js`

  - ```js
    import React, { useEffect, useState, useCallback } from 'react';
    import jwt_decode from 'jwt-decode';
    import { useNavigate } from 'react-router-dom';

    const Dashboard = () => {
      const navigate = useNavigate();
      const [quote, setQuote] = useState('');
      const [tempQuote, setTempQuote] = useState('');

      // get user quote
      async function populateQuote() {
        const req = await fetch('http://localhost:1337/api/quote', {
          headers: {
            'x-access-token': localStorage.getItem('token'),
          },
        });

        const data = await req.json();
        if (data.status === 'ok') {
          setQuote(data.quote);
        } else {
          alert(data.error);
        }
      }

      useEffect(() => {
        const token = localStorage.getItem('token');
        if (token) {
          const user = jwt_decode(token);
          if (!user) {
            localStorage.removeItem('token');
            navigate('/login');
          } else {
            populateQuote();
          }
        }
      }, []);

      // update user quote
      async function updateQuote(event) {
        event.preventDefault();
        const req = await fetch('http://localhost:1337/api/quote', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'x-access-token': localStorage.getItem('token'),
          },
          body: JSON.stringify({
            quote: tempQuote,
          }),
        });

        const data = await req.json();
        if (data.status === 'ok') {
          setQuote(tempQuote);
          setTempQuote('');
        } else {
          alert(data.error);
        }
      }

      return (
        <div>
          <h1>Your quote: {quote || 'No quote found'}</h1>
          <form onSubmit={updateQuote}>
            <input
              type='text'
              placeholder='Quote'
              value={tempQuote}
              onChange={(e) => setTempQuote(e.target.value)}
              autoFocus
              ref={(inputElement) => {
                if (inputElement) {
                  inputElement.focus();
                }
              }}
            />
            <input type='submit' value='Update quote' />
          </form>
        </div>
      );
    };

    export default Dashboard;
    ```

### Fix Errors

- When I tried to use jsonwebtoken with latest version of create-react-app, it issued some errors as below.

  - I tried to fix the errors by adding code like `"browser": { "stream": false, "util": false, "buffer": false }` to package.json of the node_modules. The errors were disappeared, but the project was not working correctly.

  - `Module not found: Error: Can't resolve 'buffer' in 'D:\study_program\mern\user_auth\client\node_modules\buffer-equal-constant-time'`

  - `Module not found: Error: Can't resolve 'crypto' in 'D:\study_program\mern\user_auth\client\node_modules\jwa'` and `Can't resolve 'util'`

  - `Module not found: Error: Can't resolve 'stream' in 'D:\study_program\mern\user_auth\client\node_modules\jws\lib'`, `Can't resolve 'util'`, and `Can't resolve 'buffer'`

  - add `"browser": { "stream": false, "util": false, "buffer": false }`

  - `Module not found: Error: Can't resolve 'buffer' in 'D:\study_program\mern\user_auth\client\node_modules\safe-buffer'`

- Solution: using jwt-decode instead of jsonwebtoken

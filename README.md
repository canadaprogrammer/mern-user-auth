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

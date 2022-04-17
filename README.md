# MERN Stack Full Tutorial | User authentication, JWT

- Development = Node.js server + React server

- Production = Node.js server + static react files

## Install React app

- `npx create-react-app client`

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

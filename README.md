# MERN Stack Docker


### API-1. Create api directory and make initial Dockerfile in the api directory
1) Dockerfile
   ```Dockerfile
      FROM node:20.5.1-bookworm-slim
      USER node
      WORKDIR /app
   ```

2) Then, in the api terminal run
   ```bash
      docker build -t api .
   ```

3) Enter into the docker container
   ```bash
      docker run -it -v $(pwd):/app --name api-1 api sh   
   ```

4) In the container, type ```whoami```, it will show node username since we put ```USER node``` in the second line of the Dockerfile. Without this, the user will be ```root```. ```USER node``` will ensure to prevent user permission issues while coding afterwards.

5) Continue setup in the container.
    ```
      npm init -y
      npm i express
      npm i -D nodemon
    ```

6) Make index.js file
   ```javascript
      const express = require('express')
      const app = express()

      app.get('/', (req, res)=>{
         res.status(200).json({
            status: 'success',
            message: 'Hello World'
         })
      })

      app.listen(3000, ()=>{
         console.log('app is listening on port 3000')
      })
   ```

7) Add the following "dev" script in the package.json file
   ```json
      "scripts": {
         "test": "echo \"Error: no test specified\" && exit 1",
         "dev": "nodemon index.js"
      },
   ```

8) If you followed correctly up to this point, if you run ```npm run dev``` in the container, it will show the ```nodemon``` is listening on port 3000. Press Ctrl+C to stop ```nodemon```

9) Exit from the container


### API-2. Update the Dockerfile in the api directory to the final version
1) Dockerfile
   ```Dockerfile
      FROM node:20.5.1-bookworm-slim
      USER node
      WORKDIR /app
      COPY --chown=node:node package*.json ./
      RUN npm ci
      COPY --chown=node:node ./ ./
      CMD [ "npm", "run", "dev" ]
   ```

   From above ```--chown=node:node``` changes the permission of the files from the root user to node user.

2) Add .dockerignore file.
   ```.dockerignore
      node_modules
   ```

3) Build the api docker image again. 
   ```bash
      docker build -t api .
   ```

4) Run the container, and test if it is working
   ```bash
      docker run -d -p 3000:3000 -v $(pwd):/app -v /app/node_modules --name api-1 api
      curl localhost:3000
   ```

   It should show ```json message``` on the cli.


### Front-1. Create front directory and make initial Dockerfile in the front directory
1) Dockerfile
   ```Dockerfile
      FROM node:20.5.1-bookworm-slim
      USER node
      WORKDIR /app
   ```

2) Then, in the front directory terminal run
   ```bash
      docker build -t front .
   ```

3) Enter into the docker container
   ```bash
      docker run -it -v $(pwd):/app --name front-1 front sh   
   ```

   In the container, initialize the react vite. But if there is any file in the directory where the react vite is installed, it emits error. So, tentantively move the Dockerfile fromt the front directory to the MERN home directory in the separate host terminal.

   ```bash
      # in the host directory
      ~/MERN $ cd front
      ~/MERN/front $ mv Dockerfile

      # in the container
      $ npm create vite@latest
      # in the project name question, type "." so that the react vite is installed in the front directory. And then, choose appropriate framework and its variant
      $ npm install
      $ npm run dev      
   ```

   If you followed correctly up to this point, if you run ```npm run dev```, it should show running message of react vite. Press Ctrl+C to stop react vite.

4) In the host terminal, move back the Dockerfile to the front directory
   ```bash
      # in the host directory
      ~/MERN $ cd Dockerfile ./front
   ```

5) Exit from the front container


### FRONT-2. Update the Dockerfile in the front directory to the final version
1) Dockerfile
   ```Dockerfile
      FROM node:20.5.1-bookworm-slim
      USER node
      WORKDIR /app
      COPY --chown=node:node package*.json ./
      RUN npm ci
      COPY --chown=node:node ./ ./
      CMD [ "npm", "run", "dev" ]
   ```

   From above ```--chown=node:node``` changes the permission of the files from the root user to node user.

2) Add .dockerignore file.
   ```.dockerignore
      node_modules
   ```

3) In the vscode front directory, change vite.config.js as follows
   ```javascript
      import { defineConfig } from 'vite'
      import react from '@vitejs/plugin-react'

      // https://vitejs.dev/config/
      export default defineConfig({
         plugins: [react()],
         server: {
            watch: {
               usePolling: true
            },
            host: true,
            strictPort: true,
            port: 5173
         }
      })
   ```

4) Build the api docker image again. 
   ```bash
      docker build -t front .
   ```

5) Run the container, and test if it is working
   ```bash
      docker run -d -p 5173:5173 -v $(pwd):/app -v /app/node_modules --name front-1 front
      curl localhost:5173
   ```

   It should show ```json message``` on the cli.


### Docker Compose-1. 
1) Set api and front into docker-compose.yml
   ```docker-compose.yml
      version: '3.9'
      services:
         api:
            build: 
               context: ./api
            ports:
               - 3000:3000
            volumes:
               - ./api:/app
               - /app/node_modules

         front:
            build: 
               context: ./front
            ports:
               - 5173:5173
            volumes:
               - ./front:/app
               - /app/node_modules
   ```

2) Check if it is working the same as before
   ```bash
      curl localhost:3000
      curl localhost:5173
   ```
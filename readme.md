# Docker-Compose project 

### Init setup

```java
npm init -y
```

```java
npm i typescript
```

```java
npx tsc --init
```

change 

- rootDir ⇒ ./src
- outDir ⇒ ./dist

package-json ⇒ modified script 

```python
"scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "tsc && node dist/index.js",
  }
```

express

```python
npm i express
or
npm install express @types/express
```

- create index.ts in src folder
- Index.js code
    
    ```python
    import express  from "express";
    const app = express();
    
    app.get("/", (req, res) => {
        res.send("Hello World");
    })
    
    app.post("/", (req, res) => {
        res.send("Hello World2");
    })
    
    app.listen(3000, () => {    
        console.log("Server is running on port 3000");
    })
    
    ```
    

check on post man 

```python
[http://localhost:3000/](http://localhost:3000/) 
```

get and post request

```java
npm i prisma
or
npm i prisma express @types/express
```

```java
npx prisma init
```

![image.png](image.png)

install "postgresql"

ENV file :

```python
 DATABASE_URL="postgresql://postgres: mysecretpassword@localhost:5432/postgres"
```

Terminal : 

```python
docker run -e POSTGRES_PASSWORD=mysecretpassword -d -p 5432:5432 postgres
```

- Index.js code (Update)
    
    ```python
    import { PrismaClient } from "@prisma/client"
    import express from "express"
    
    const app = express()
    const prismaClient = new PrismaClient()
    
    app.get("/", async (req, res) => {
      const data = await prismaClient.user.findMany()
      res.json({
        data,
      })
    })
    
    app.post("/", async (req, res) => {
      await prismaClient.user.create({
        data: {
          username: Math.random().toString(),
          password: Math.random().toString(),
        },
      })
      res.json({
        message: "post endpoint",
      })
    })
    
    app.listen(3000)
    ```
    

if this is complicated , just go to neon DB and paste the URI

```java
npx prisma migrate dev
npx prisma generate 
npm run dev
npm run start
```

this migration i have to do it once , after that docker-compose will automate it 

### Manual setup img

![image.png](image%201.png)

```python
git remote 
git init 
vi .gitignore 
	# check node_modules & .env is present 
	:wxxxx
git add .
git commit -m "Added initial part"
git remote add origin URL
git push origin HEAD

```

[https://github.com/100xdevs-cohort-3/week-27-docker-compose](https://github.com/100xdevs-cohort-3/week-27-docker-compose)

DOCKER FILE

```python
FROM node:20-alpine

WORKDIR /app

COPY ./package.json ./package.json
COPY ./package-lock.json ./package-lock.json

RUN npm install

COPY . . 

ENV DATABASE_URL = postgresql://postgres:mysecretpassword@localhost:5432/postgres
ENV npx prisma migerate dev
ENV npx prisma generate
RUN npm run dev

CMD ["npm", "start"]
```

### Docker Steps img

```python
docker network create user_project
```

```python
docker run -e POSTGRES_PASSWORD=mysecretpassword -d -p 5432:5432 postgres
```

```python
docker build -t user-project .
```

```python
docker run -e DATABASE_URL=postgresql://postgres:mysecretpassword@postgres:5432/postgres --network user_project -p 3000:3000 user-project
```

moving `npx prisma migrate dev` from docker file to script

now it is docker dev script bcz  user_app only work when postgres container start , but before starting contianer , we are migrating 

inshort : u have 2 container , 1st is building and 2nd is DB which is UP, while building 1st and talking to 2nd same time is hard , we do it another way like host mode , so extracted this step  

WHY : this step happen when the container start , and when the container start DB is up 

```python
"scripts": {
    "build": "tsc -b",
    "start": "node dist/index.js",
    "dev:docker": "npx prisma migrate dev && node dist/index.js"
  },
```

and replace docker file 

```python
CMD ["npm", "run", "dev:docker"]
```

- complete Dockerfile
    
    ```python
    FROM node:20-alpine
    WORKDIR /app
    COPY ./package.json ./package.json
    COPY ./package-lock.json ./package-lock.json
    RUN npm install
    
    COPY . .
    
    RUN npx prisma generate
    RUN npm run build
    
    EXPOSE 3000
    
    CMD ["npm", "run", "dev:docker"]
    ```
    
- Docker-compose file
    
    ```python
    version: '3.8'
    services:
      postgres:
        image: postgres
        ports:
          - 5432:5432
        environment:
          - POSTGRES_PASSWORD=mysecretpassword
    
      user_app:
        build:
          network: host
          context: ./ 
          dockerfile: Dockerfile
    
        environment:
          - DATABASE_URL=postgresql://postgres:mysecretpassword@postgres:5432/postgres
    
        ports:
          - 3000:3000
        depends_on:
          - postgres
    ```

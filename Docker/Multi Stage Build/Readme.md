## 🚀 Multistage Build
A multistage build in Docker is a technique that allows you to use multiple FROM statements in a single Dockerfile, separating the build environment from the runtime environment.

This helps:
* ✅ optimize image size
* ✅ improve security
* ✅ reduce complexity


## 🏗 Lab: Build & Run Node.js App in Docker (with Intro to Multistage Builds)

### 📁 Step 1: Create Project Files
Make a folder:
```bash
mkdir docker-app
```
```bash
cd docker-app
```

📄 Create package.json
```bash
vi package.json
```
Add the given content, by pressing `INSERT`
```json
{
  "name": "docker-app",
  "version": "1.0.0",
  "description": "A simple Node.js application",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```
save the file using `ESCAPE + :wq!`

📄 Create index.js
```bash
vi index.js
```
Add the given content, by pressing `INSERT`
```js
const port = process.env.PORT || 8080;
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  const currentDate = new Date();
  const dateTime = currentDate.toLocaleString();

  res.send(`
    <div style="font-family: Arial, sans-serif; text-align: center; padding: 50px;">
      <h1 style="color: dodgerblue; font-size: 50px;">
        Hello-World-777!
      </h1>
      <p style="color: gold; font-size: 20px;">
        Current Date and Time: ${dateTime}
      </p>
    </div>
  `);
});

app.get('/readiness', (req, res) => res.send('Ready !!'));

app.get('/liveness', (req, res) => res.send('Live !!'));

app.listen(port, () => console.log(`Example app listening on port ${port}!`));

```
save the file using `ESCAPE + :wq!`

📄 Create Dockerfile
```bash
vi Dockerfile
```
Add the given content, by pressing `INSERT`
```Dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app

# Copy package.json and package-lock.json first to leverage Docker cache
COPY package.json ./

# Install dependencies
RUN npm install --only=production

# Copy the entire source code
COPY . .

# Stage 2: Run (Alpine)
FROM alpine:3
WORKDIR /app

# Install Node.js in Alpine
RUN apk add --no-cache nodejs

# Copy built files from the builder stage
COPY --from=builder /app .

# Expose the application port
EXPOSE 8080

# Run the Node.js application
CMD ["node", "index.js"]

```
save the file using `ESCAPE + :wq!`

### 🏃‍♂️ Step 2: Build the Docker Image
```bash
docker build -t test:v1 .
```
Verify:
```bash
docker images
```

### 🏃‍♂️ Step 3: Run the Docker Container
```bash
docker run -d -p 8080:8080 --name myapp test:v1
```
Confirm:
```bash
docker ps
```
### 🏃‍♂️ Step 4: Test Locally
```bash
curl http://localhost:8080
```
✅ You should get the Hello-World! page.

### 🏃‍♂️ Step 5: Open Firewall for External Access
* Go to your cloud console (AWS/GCP/Azure)
* Check security group / firewall rules
* Add inbound TCP rule on port 8080
* Allow from your IP or 0.0.0.0/0 (for testing)

### 🏃‍♂️ Step 6: Test from Browser
```bash
http://<YOUR_PUBLIC_IP>:8080
```
Example:
http://3.99.223.139:8080
✅ You should see the webpage.

### 🛠 Optional: Check Logs
```bash
docker logs myapp
```



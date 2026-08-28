# BASE IMAGE
FROM node:20-alpine

#Set the working directory inside the container
WORKDIR /app

#Copy package files first to leverage Docker cache layers
COPY package*.json ./

#Install dependencies
RUN npm install --omit=dev

#Copy the remaining application source code
COPY . . 

#Expose the port your app
EXPOSE 5000

#run container as node(non-root user) instead of root
USER node

#Define the command to run your app
CMD ["node", "src/index.js"]

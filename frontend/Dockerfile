# Stage 1 : Build the application
#base image
FROM node:20-alpine AS builder

#working directory in container
WORKDIR /app

#copy dependencies to container
COPY package*.json .

#install dependencies
RUN npm install

#copy remaining code 
COPY . .

#build the remaining things
RUN npm run build

# Stage 2 : run application with nginx
#base image 
FROM nginx:1.27-alpine

#copy build thing from stage 1 to comtainer
COPY --from=builder /app/dist /usr/share/nginx/html

COPY nginx.conf /etc/nginx/conf.d/default.conf


#expose the port 
EXPOSE 80

#start the application
CMD ["nginx", "-g", "daemon off;"]



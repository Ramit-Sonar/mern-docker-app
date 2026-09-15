# Use Node.js 22.22.2 as the base image
FROM node:22.22.2

# Set /app as the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json first
# This allows Docker to cache the dependency installation layer
COPY package*.json ./

# Install all project dependencies
RUN npm install

# Copy the remaining backend source code into the container
COPY . .

# Document that the application runs on port 5000
EXPOSE 5000

# Enable polling so file changes are detected properly
# when developing with Docker, especially on Windows
ENV CHOKIDAR_USEPOLLING=true

# Enable polling for file watchers used by some development tools
ENV WATCHPACK_POLLING=true

# Start the application using the "dev" script from package.json
CMD ["npm", "run", "dev"]
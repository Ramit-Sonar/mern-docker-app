# Use Node.js 20 with Alpine Linux as the base image
FROM node:22.22.2

# Set /app as the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json first
# This allows Docker to cache the dependency installation layer
COPY package*.json ./

# Install frontend dependencies
RUN npm install

# Copy the remaining frontend source code into the container
COPY . .

# Document that Vite development server runs on port 5173
EXPOSE 5173

# Enable polling so file changes are detected properly
# when using Docker, especially on Windows
ENV CHOKIDAR_USEPOLLING=true

# Enable polling for file watchers
ENV WATCHPACK_POLLING=true

# Enable React Fast Refresh during development
ENV FAST_REFRESH=true

# Start Vite development server
# "--host" makes Vite accessible outside the container
CMD ["npm", "run", "dev", "--", "--host"]
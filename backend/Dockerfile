FROM node:18-alpine

WORKDIR /app

# Copy package files from root
COPY package*.json ./

# Install ALL dependencies (frontend + backend)
RUN npm install

# Copy entire project
COPY . .

# Change to backend directory for execution
WORKDIR /app

# Expose port
EXPOSE 5000

# Start the backend server
CMD ["npm", "run", "server"]
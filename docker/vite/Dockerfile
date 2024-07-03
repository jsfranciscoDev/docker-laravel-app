# Use the latest version of the official Node.js image
FROM node:latest

# Set working directory
WORKDIR /var/www

# Copy package.json and package-lock.json to the container
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application
COPY . .

# Ensure www-data user owns the application files
RUN chown -R www-data:www-data /var/www

# Set permissions for the public directory
RUN chmod -R 775 /var/www/public

# Set permissions for vite.config.js
RUN chmod 775 /var/www/vite.config.js

# Change to the www-data user
USER www-data

# Expose port for Vite
EXPOSE 5173

# Run Vite development server
CMD ["npm", "run", "dev"]

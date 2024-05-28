# Base image
FROM node:carbon

# Set working directory
WORKDIR /usr/src/app

# Copy server.js to the working directory
COPY server.js .

# Expose port 8080
EXPOSE 8080

# Run the application
CMD ["sh", "-c", "node server.js > log.out"]

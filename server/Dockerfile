# Start with Node.js 22 on a lightweight Debian image
FROM mirror.gcr.io/library/node:22-bookworm-slim

# Set the application environment
ENV NODE_ENV=production

# Set the working directory inside the container
WORKDIR /app

# Copy dependency manifests first
COPY package*.json ./

# Install production dependencies from the lockfile
RUN npm ci --omit=dev

# Copy the backend application
COPY --chown=node:node . .

# Prepare persistent-upload directory permissions
RUN mkdir -p uploads && chown node:node uploads

# Run the application without root privileges
USER node

# Document the Express port
EXPOSE 3000

# Start Express
CMD ["npm", "start"]

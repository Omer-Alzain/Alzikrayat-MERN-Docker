# Stage 1: Build the React application

FROM mirror.gcr.io/library/node:22-bookworm-slim AS build

WORKDIR /app

# Copy dependency manifests first
COPY package*.json ./

# Install dependencies using the lockfile
RUN npm ci

# Copy the React application
COPY . .

# Generate the production build in /app/dist
RUN npm run build


# Stage 2: Serve using Nginx

FROM mirror.gcr.io/library/nginx:alpine

# Replace the default Nginx configuration
COPY nginx/default.conf /etc/nginx/conf.d/default.conf

# Copy only the compiled frontend from Stage 1
COPY --from=build /app/dist /usr/share/nginx/html

# Nginx listens on port 80
EXPOSE 80

# Keep Nginx running in the foreground
CMD ["nginx", "-g", "daemon off;"]

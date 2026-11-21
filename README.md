# Alzikrayat — MERN Stack with Docker

Alzikrayat is a photo-sharing application rebuilt from the original PHP project using MongoDB, Express, React, and Node.js.

For **Course Project 4 — Track B: Docker Containerization**, the application has been containerized using Docker. Docker Compose orchestrates the frontend, backend, and database, while persistent volumes protect database records and uploaded photographs across container restarts and recreation.

## Features

- Register and log in with bcrypt password hashing.
- Session-based login state and logout.
- Home page with recent memories.
- Photo gallery with 3-column, 4-column, list, and slider layouts.
- Photo upload with JPG, PNG, and WEBP validation (maximum 2MB).
- Photo detail page with metadata and comments.
- Logged-in users can comment on any photo.
- Only the photo owner can delete that photo.
- Responsive layout for desktop, tablet, and mobile.

## Technology Stack

| Component | Technology |
|---|---|
| Frontend | React 19, Vite |
| Web server / reverse proxy | Nginx |
| Backend | Node.js 22, Express 5 |
| Database | MongoDB 8 |
| Containerization | Docker |
| Container orchestration | Docker Compose |

## Docker Architecture

The application consists of three services:

1. **Client:** Nginx serves the production React build and forwards API and uploaded-image requests to the backend.
2. **Server:** Express handles authentication, photo operations, comments, and sessions.
3. **MongoDB:** Stores application data and session information.

All services communicate through the Docker Compose network. Only the frontend is exposed to the host machine.

```text
Browser
   |
   | http://localhost:8080
   |
   v
Client (Nginx)
   |
   | /api and /uploads
   v
Server (Express :3000)
   |
   v
MongoDB (:27017)
```

### Docker files

```text
Alzikrayat-MERN-Docker/
├── client/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── nginx/
│   │   └── default.conf
│   └── src/
├── server/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── config/
│   ├── controllers/
│   ├── middleware/
│   ├── models/
│   ├── routes/
│   └── uploads/
├── docker-compose.yml
├── .env
├── .env.example
└── README.md
```

## Docker Implementation

### 1. Backend Dockerfile

The backend uses the official Node.js 22 Bookworm Slim image.

- Sets `/app` as the working directory.
- Installs dependencies using `npm ci --omit=dev`.
- Copies the backend application.
- Creates the uploads directory.
- Runs the application using the non-root `node` user.
- Exposes port 3000 internally.

### 2. Frontend multi-stage Dockerfile

The frontend uses a multi-stage build:

**Build stage:**
- Uses Node.js 22.
- Installs dependencies using `npm ci`.
- Runs `npm run build` to generate the production React files.

**Production stage:**
- Uses Nginx Alpine.
- Copies the generated `dist` directory from the build stage.
- Loads the custom Nginx configuration.
- Serves the frontend on port 80.

This approach keeps the final frontend image smaller by excluding the Node.js build environment and development dependencies.

### 3. Docker Compose

Docker Compose manages the three services:

- `client`
- `server`
- `mongodb`

It also configures service dependencies, health checks, port mapping, environment variables, and persistent storage.

### 4. Persistent storage

Two named Docker volumes are configured:

| Volume | Container path | Purpose |
|---|---|---|
| `mongo_data` | `/data/db` | MongoDB database files |
| `uploads_data` | `/app/uploads` | Uploaded photographs |

These volumes preserve data when containers are stopped, removed, or recreated.

### 5. Networking

Docker Compose creates an internal network for communication between services.

- Nginx communicates with Express using `server:3000`.
- Express communicates with MongoDB using `mongodb:27017`.
- The frontend is accessible from the host through port 8080.
- MongoDB and Express ports are not published directly to the host.

## Requirements

For Docker deployment:

- Docker Engine
- Docker Compose plugin

For running the application locally without Docker:

- Node.js and npm
- MongoDB running locally or a MongoDB Atlas database

## Docker Setup and Installation

### 1. Clone the repository

```bash
git clone https://github.com/Omer-Alzain/Alzikrayat-MERN-Docker.git
cd Alzikrayat-MERN-Docker
```

### 2. Configure environment variables

Create the root `.env` file:

```bash
cp .env.example .env
```

Generate a secure session secret:

```bash
openssl rand -hex 32
```

Replace the placeholder in `.env` with the generated secret.

Never commit the real `.env` file.

### 3. Build and start the application

```bash
docker compose up -d --build
```

This builds the frontend and backend images, pulls MongoDB, and starts all services.

### 4. Check service status

```bash
docker compose ps
```

Check application logs:

```bash
docker compose logs -f
```

### 5. Access the application

Open:

```text
http://localhost:8080
```

The API health endpoint can also be accessed through Nginx:

```text
http://localhost:8080/api/health
```

## Managing Docker Containers

Start the services:

```bash
docker compose up -d
```

Stop the services:

```bash
docker compose stop
```

Start stopped services:

```bash
docker compose start
```

Rebuild and recreate the services:

```bash
docker compose up -d --build
```

View logs:

```bash
docker compose logs -f
```

Stop and remove containers and the Compose network:

```bash
docker compose down
```

**Important:** `docker compose down` preserves named volumes.

To remove the database and uploaded-photo volumes as well:

```bash
docker compose down -v
```

Use the last command carefully because it permanently deletes the data stored in those Docker volumes.

## Local Development Without Docker

The original local development workflow is still available.

Install dependencies:

```bash
npm install
npm install --prefix server
npm install --prefix client
```

Configure `server/.env` using `server/.env.example`.

Start both applications:

```bash
npm run dev
```

The frontend normally runs at `http://localhost:5173`, and the backend runs on port 3000.

To start only one side:

```bash
npm run dev:server
npm run dev:client
```

Create a production frontend build:

```bash
npm run build
```

## Environment Variables

### Docker Compose environment

| Variable | Description |
|---|---|
| `SESSION_SECRET` | Secret used to sign session cookies. Set in the root `.env` file. |

The Compose file also configures the following backend variables:

| Variable | Docker value | Description |
|---|---|---|
| `PORT` | `3000` | Express server port |
| `NODE_ENV` | `development` | Local HTTP cookie configuration |
| `MONGODB_URI` | `mongodb://mongodb:27017/alzikrayat` | MongoDB connection string |
| `CLIENT_URL` | `http://localhost:8080` | Allowed frontend origin |

The local Docker deployment uses HTTP and development cookie settings. A public production deployment should use HTTPS and appropriately configured secure cookies.

### Local backend environment

| Variable | Description |
|---|---|
| `PORT` | Express port (default: `3000`) |
| `NODE_ENV` | Application environment |
| `MONGODB_URI` | MongoDB connection string |
| `SESSION_SECRET` | Session signing secret |
| `CLIENT_URL` | Allowed frontend origin |

Never commit real environment files or secrets.

## REST API

| Method | Endpoint | Purpose |
|---|---|---|
| `POST` | `/api/auth/register` | Create an account |
| `POST` | `/api/auth/login` | Log in |
| `GET` | `/api/auth/me` | Get the current session user |
| `POST` | `/api/auth/logout` | End the current session |
| `GET` | `/api/photos` | Get all photos |
| `GET` | `/api/photos/:id` | Get a photo and its comments |
| `POST` | `/api/photos` | Upload a photo (login required) |
| `DELETE` | `/api/photos/:id` | Delete an owned photo (login required) |
| `POST` | `/api/photos/:id/comments` | Add a comment (login required) |
| `GET` | `/api/health` | Check API status |

The application uses HTTP-only session cookies. Passwords are hashed using bcrypt and are never returned by the API. JWT authentication is not required by the Project 3 requirements.

## Notes

- Uploaded images are stored in `server/uploads/` during local development and `/app/uploads` inside the backend container.
- MongoDB stores photo and comment metadata, as well as session data.
- Deleting a photo also removes its comments and local image file.
- The Docker deployment uses named volumes to persist database data and uploaded photographs.
- For public deployment, configure HTTPS, secure cookies, a strong session secret, an appropriate allowed frontend origin, and persistent storage.

## Course Project 4 — Track B

**Selected track:** Docker Containerization.

The implementation covers:

- Backend Dockerfile.
- Frontend multi-stage Dockerfile.
- Nginx production configuration.
- Docker Compose orchestration.
- MongoDB service and persistent database volume.
- Persistent uploaded-photo volume.
- Environment variable configuration.
- README setup and execution instructions.


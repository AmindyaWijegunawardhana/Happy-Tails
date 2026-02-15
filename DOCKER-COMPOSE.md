# Docker Compose Setup for Happy Tails

This setup orchestrates the entire Happy Tails application using Docker Compose with three main services:
- **Backend**: Express.js API server running on port 5000
- **Frontend**: React app served by Nginx on port 80
- **MySQL**: Database service on port 3306

## Prerequisites

- Docker (version 20.10+)
- Docker Compose (version 1.29+)

## Quick Start

### 1. Clone and Setup

```bash
# Navigate to project root
cd d:\project\happy-tails

# Copy and configure environment variables
copy .env.example .env
# Edit .env with your configuration
```

### 2. Build and Start Services

```bash
# Build and start all services
docker-compose up -d

# View logs
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f mysql

# Check service status
docker-compose ps
```

### 3. Access the Application

- **Frontend**: http://localhost
- **Backend API**: http://localhost:5000
- **Health Check**: http://localhost:5000/api/health

## Service Details

### Backend Service
- **Image**: Node.js 18 Alpine
- **Port**: 5000
- **Health Check**: `/api/health` endpoint
- **Environment Variables**: Database credentials and Node environment

### Frontend Service
- **Image**: Nginx Alpine
- **Port**: 80
- **Health Check**: `/index.html` endpoint
- **Dependencies**: Depends on backend service

### MySQL Service
- **Image**: MySQL 8.0
- **Port**: 3306
- **Volume**: Persisted at `mysql-data`
- **Initialization**: Runs `schema.sql` on startup

## Environment Variables

Create a `.env` file in the project root:

```env
DB_USER=appuser
DB_PASSWORD=your_secure_password
DB_NAME=happy_tails
NODE_ENV=production
```

## Common Commands

```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# Stop and remove all data
docker-compose down -v

# Rebuild images
docker-compose build --no-cache

# View logs for specific service
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f mysql

# Execute commands in container
docker-compose exec backend npm run test
docker-compose exec mysql mysql -uroot -p<password> happy_tails

# Restart a service
docker-compose restart backend
```

## Networking

All services are connected through an internal bridge network `app-network`, allowing services to communicate using their service names:
- Backend can access MySQL at `mysql:3306`
- Frontend connects to backend at `backend:5000`

## Troubleshooting

### Services won't start
```bash
# Check logs
docker-compose logs

# Verify ports aren't in use
netstat -ano | findstr :80
netstat -ano | findstr :5000
netstat -ano | findstr :3306
```

### Database connection issues
```bash
# Test MySQL connectivity
docker-compose exec backend node -e "
  const mysql = require('mysql2/promise');
  const conn = await mysql.createConnection({
    host: 'mysql',
    user: 'appuser',
    password: process.env.DB_PASSWORD,
    database: 'happy_tails'
  });
  console.log('Connected!');
"
```

### Health checks failing
```bash
# Check service health
docker-compose ps

# View detailed logs
docker-compose logs backend
docker-compose logs frontend
```

## Production Considerations

For production deployments:
1. Use strong, unique database passwords
2. Store secrets in a secrets manager (not .env file)
3. Use health checks with appropriate timeouts
4. Consider adding resource limits
5. Enable logging and monitoring
6. Use persistent volumes for database data
7. Set up automated backups

## Architecture Diagram

```
┌─────────────────────────────────────────┐
│         Docker Compose Network          │
├─────────────────────────────────────────┤
│                                         │
│  ┌──────────────┐   ┌──────────────┐   │
│  │   Frontend   │   │   Backend    │   │
│  │   (Nginx)    │──▶│  (Express)   │   │
│  │   Port: 80   │   │  Port: 5000  │   │
│  └──────────────┘   └──────────────┘   │
│                           │             │
│                           ▼             │
│                    ┌──────────────┐     │
│                    │    MySQL     │     │
│                    │  Port: 3306  │     │
│                    └──────────────┘     │
└─────────────────────────────────────────┘
```

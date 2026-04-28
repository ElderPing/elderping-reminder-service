# ElderPing Reminder Service

## Overview

The Reminder Service is a Node.js-based microservice responsible for managing medication and appointment reminders for elderly users in the ElderPing platform. It handles reminder creation, scheduling, delivery, and tracking.

## Technology Stack

- **Runtime**: Node.js 18
- **Framework**: Express.js
- **Database**: PostgreSQL
- **Container**: Docker (multi-stage build with Alpine Linux)
- **Security**: Non-root user execution (USER node)

## Features

- Medication reminders with customizable schedules
- Appointment reminders
- Reminder delivery via multiple channels (in-app, push notifications)
- Reminder adherence tracking
- Reminder history and analytics
- Health check endpoint for monitoring

## API Endpoints

### Reminders
- `POST /api/reminder/create` - Create new reminder
- `GET /api/reminder/:userId` - Get reminders for user
- `GET /api/reminder/:id` - Get specific reminder
- `PUT /api/reminder/:id` - Update reminder
- `DELETE /api/reminder/:id` - Delete reminder
- `POST /api/reminder/:id/complete` - Mark reminder as completed

### Reminder Schedules
- `POST /api/reminder/schedule` - Create reminder schedule
- `GET /api/reminder/schedule/:reminderId` - Get schedule for reminder
- `PUT /api/reminder/schedule/:id` - Update schedule

### Health
- `GET /health` - Health check endpoint

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `DB_HOST` | PostgreSQL database host | Yes |
| `DB_PORT` | PostgreSQL database port | Yes |
| `DB_USER` | PostgreSQL username | Yes |
| `DB_PASSWORD` | PostgreSQL password | Yes |
| `DB_NAME` | Database name (reminder_db) | Yes |
| `PORT` | Service port (default: 3000) | No |

## Database Schema

### Reminders Table
```sql
CREATE TABLE reminders (
  id          SERIAL PRIMARY KEY,
  user_id     INT NOT NULL,
  title       VARCHAR(200) NOT NULL,
  description TEXT,
  type        VARCHAR(50) NOT NULL, -- medication, appointment, etc.
  due_date    TIMESTAMP NOT NULL,
  status      VARCHAR(20) DEFAULT 'pending',
  created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Reminder Schedules Table
```sql
CREATE TABLE reminder_schedules (
  id          SERIAL PRIMARY KEY,
  reminder_id INT NOT NULL REFERENCES reminders(id) ON DELETE CASCADE,
  frequency   VARCHAR(50) NOT NULL, -- daily, weekly, monthly
  days_of_week VARCHAR(20),
  time        TIME NOT NULL,
  end_date    DATE,
  created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Reminder History Table
```sql
CREATE TABLE reminder_history (
  id          SERIAL PRIMARY KEY,
  reminder_id INT NOT NULL REFERENCES reminders(id) ON DELETE CASCADE,
  completed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  status      VARCHAR(20) NOT NULL
);
```

## Docker Image

- **Repository**: `arunnsimon/elderpinq-reminder-service`
- **Tags**: 
  - `dev-latest` - Development builds from develop branch
  - `prod-latest` - Production builds from main branch
  - `<version>` - Release tags

## CI/CD Pipeline

The service uses GitHub Actions for continuous integration and deployment:

1. **Security Scanning**
   - SAST (Static Application Security Testing)
   - SCA (Software Composition Analysis)
   - Trivy vulnerability scanning

2. **Docker Build & Publish**
   - Multi-stage Docker build
   - Push to Docker Hub
   - Tagged based on branch (dev-latest/prod-latest)

3. **GitOps Deployment**
   - Updates Helm chart image tag in elderping-k8s-charts
   - ArgoCD automatically syncs changes

## Kubernetes Deployment

### Helm Chart
Located in `elderping-k8s-charts/microservices/reminder-service/`

**Resources:**
- Deployment with 2 replicas
- Service (ClusterIP on port 3000)
- HorizontalPodAutoscaler (2-5 replicas, 80% CPU target)

**Configuration:**
- Namespace: elderping-dev (dev) / elderping-prod (prod)
- Resource requests: 100m CPU, 128Mi memory
- Resource limits: 500m CPU, 256Mi memory
- Liveness/Readiness probes on /health endpoint

## Security Features

- **Non-root container**: Runs as `node` user (not root)
- **Environment variables**: Sensitive data via Kubernetes Secrets
- **Network policies**: Restricts ingress/egress traffic (when enabled)

## Development

### Local Setup
```bash
# Install dependencies
npm install

# Set environment variables
cp .env.example .env
# Edit .env with your values

# Run development server
npm run dev

# Run tests
npm test

# Build for production
npm run build
```

### Docker Build
```bash
# Build image
docker build -t elderping-reminder-service .

# Run container
docker run -p 3000:3000 --env-file .env elderping-reminder-service
```

## Monitoring

- **Health Check**: `GET /health` returns service status
- **Metrics**: Exposed for Prometheus scraping
- **Logs**: Collected by Loki
- **Dashboards**: Grafana dashboards for monitoring

## Troubleshooting

### Common Issues

**Database Connection Failed**
- Verify DB_HOST, DB_PORT, DB_USER, DB_PASSWORD, DB_NAME
- Check PostgreSQL is accessible from the pod
- Verify network policies allow database access

**Container Not Starting**
- Check pod logs: `kubectl logs <pod-name> -n elderping-dev`
- Verify resource limits are sufficient
- Check liveness probe configuration

## Contributing

1. Create feature branch from develop
2. Make changes and test locally
3. Commit with descriptive message
4. Push to feature branch
5. Create pull request to develop

## License

Proprietary - ElderPing Platform

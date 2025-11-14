# Next.js Application with Docker Deployment

This is a [Next.js](https://nextjs.org) project configured for production deployment using Docker and nginx with automated SSL certificates.

## 🚀 Features

- **Next.js 16** with App Router
- **Docker** containerization
- **Nginx** reverse proxy with SSL support
- **GitHub Actions** CI/CD pipeline
- **Automated SSL certificates** with Let's Encrypt
- **Automated deployment** to Ubuntu servers

## 📋 Prerequisites

- Node.js 20+ (for local development)
- Docker & Docker Compose (for containerized deployment)
- Ubuntu server (for production deployment)
- Domain name with DNS configured

## 🛠️ Local Development

```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

## 🐳 Docker Development

### Initial Configuration

Before running Docker Compose, update the `docker-compose.yml` file:

1. **Set your Docker image name** (line 3):
   ```yaml
   image: "ghcr.io/your-username/your-app:latest"
   ```
   Replace with your GitHub Container Registry image or Docker Hub image.

2. **Customize environment variables** (optional):
   ```yaml
   environment:
     - NODE_ENV=production  # or development/staging
     - PORT=3000           # change if using different port
   ```

3. **Adjust ports if needed** (optional):
   ```yaml
   ports:
     - "3000:3000"  # host:container
   ```

### Running with Docker

```bash
# Build Docker image
docker build -t nextjs-app .

# Run with Docker Compose
docker-compose up -d

# View logs
docker-compose logs -f

# Stop containers
docker-compose down
```

## 🚢 Production Deployment

### One-Time Server Setup

1. **SSH into your server:**
   ```bash
   ssh user@your-server-ip
   ```

2. **Install Docker and Docker Compose:**
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   sudo usermod -aG docker $USER
   sudo apt update
   ```

3. **Verify installation:**
   ```bash
   docker --version
   docker compose version
   ```

4. **Create deployment directory:**
   ```bash
   sudo mkdir -p /opt/your-project
   sudo chown $USER:$USER /opt/your-project
   ```

### GitHub Secrets Setup

Go to: **GitHub Repo → Settings → Secrets and Variables → Actions**

**Add these SECRETS:**
- `CLOUD_HOST`: your-server-ip
- `CLOUD_USER`: your-ssh-username
- `CLOUD_SECRET_KEY`: your-private-ssh-key
- `DEPLOY_PATH`: /opt/your-project

**Add these VARIABLES:**
- `CLOUD_DOMAIN`: your-domain.com
- `CLOUD_EMAIL`: your@email.com

### DNS Setup

Configure A records:
- `your-domain.com` → your-server-ip
- `www.your-domain.com` → your-server-ip

Wait for DNS propagation (check with: `dig your-domain.com`)

### Deploy

1. **Push to main branch:**
   ```bash
   git add .
   git commit -m "deploy"
   git push origin main
   ```

2. **GitHub Actions will automatically:**
   - Build Docker image
   - Deploy to server
   - Generate SSL certificates (first time only)
   - Start services with HTTPS

3. **Check deployment:**
   Visit: https://your-domain.com

### Manual Commands (if needed)

SSH to server:
```bash
cd /opt/your-project
docker compose ps              # Check status
docker compose logs -f         # View logs
docker compose restart         # Restart
docker compose down            # Stop
docker compose up -d           # Start
```

**SSL certificate renewal (automatic via certbot container):**
```bash
docker compose exec certbot certbot renew
docker compose exec nginx nginx -s reload
```

**Manual SSL certificate generation (if needed):**
```bash
sudo docker run --rm \
  -p 80:80 -p 443:443 \
  -v $(pwd)/certbot/conf:/etc/letsencrypt \
  -v $(pwd)/certbot/www:/var/www/certbot \
  certbot/certbot certonly --standalone \
  -d your-domain.com -d www.your-domain.com \
  --agree-tos -m your@email.com --non-interactive
```

## 📁 Project Structure

```
.
├── app/                  # Next.js app directory
│   ├── api/             # API routes
│   │   └── health/      # Health check endpoint
│   ├── layout.tsx       # Root layout
│   └── page.tsx         # Home page
├── nginx/               # Nginx configuration
│   ├── nginx.conf       # Main nginx config
│   └── conf.d/          # Site configurations
├── .github/
│   └── workflows/       # GitHub Actions workflows
├── Dockerfile           # Production Dockerfile
├── docker-compose.yml   # Docker Compose configuration
└── next.config.ts       # Next.js configuration
```

## 🏗️ Architecture

```
Internet → Nginx (80/443) → Next.js (3000)
```

- **Nginx**: Handles SSL, reverse proxy, rate limiting
- **Next.js**: Runs in standalone mode for optimal performance
- **Docker**: Provides isolation and consistency
- **Certbot**: Automatic SSL certificate management

## 🔍 Monitoring

### Health Checks

- Application: `https://your-domain.com/api/health`

### View Logs

```bash
# Docker logs for debugging
docker compose logs -f nextjs
docker compose logs -f nginx

# Nginx access logs
tail -f nginx/logs/access.log
```

## 📚 Learn More

- [Next.js Documentation](https://nextjs.org/docs)
- [Docker Documentation](https://docs.docker.com/)
- [Nginx Documentation](https://nginx.org/en/docs/)

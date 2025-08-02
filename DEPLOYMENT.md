# Deployment Guide - Yukbelajar Clone

This guide provides step-by-step instructions for deploying the Yukbelajar clone to various platforms.

## 🚀 Quick Start (Local Development)

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Open browser
open http://localhost:8000
```

## 🌐 Production Deployment

### Option 1: Vercel (Recommended)

Vercel is the easiest way to deploy Next.js applications.

#### Prerequisites
- GitHub account
- Vercel account (free)

#### Steps
1. **Push to GitHub:**
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/yourusername/yukbelajar-clone.git
git push -u origin main
```

2. **Deploy to Vercel:**
   - Go to [vercel.com](https://vercel.com)
   - Click "New Project"
   - Import your GitHub repository
   - Configure settings:
     - Framework Preset: Next.js
     - Root Directory: `./`
     - Build Command: `npm run build`
     - Output Directory: `.next`
   - Click "Deploy"

3. **Custom Domain (Optional):**
   - Go to Project Settings → Domains
   - Add your custom domain
   - Configure DNS records as instructed

#### Environment Variables
```env
NEXT_PUBLIC_SITE_URL=https://your-domain.vercel.app
```

### Option 2: Netlify

#### Prerequisites
- Netlify account (free)

#### Steps
1. **Build the project:**
```bash
npm run build
npm run export  # If using static export
```

2. **Deploy via Netlify CLI:**
```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login to Netlify
netlify login

# Deploy
netlify deploy --prod --dir=out
```

3. **Or deploy via Netlify Dashboard:**
   - Drag and drop the `out` folder to Netlify
   - Configure build settings:
     - Build command: `npm run build && npm run export`
     - Publish directory: `out`

### Option 3: Self-Hosted (VPS/Dedicated Server)

#### Prerequisites
- Ubuntu/CentOS server
- Node.js 18+
- PM2 (process manager)
- Nginx (reverse proxy)

#### Steps

1. **Server Setup:**
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PM2
sudo npm install -g pm2

# Install Nginx
sudo apt install nginx -y
```

2. **Deploy Application:**
```bash
# Clone repository
git clone https://github.com/yourusername/yukbelajar-clone.git
cd yukbelajar-clone

# Install dependencies
npm install

# Build application
npm run build

# Start with PM2
pm2 start npm --name "yukbelajar" -- start
pm2 save
pm2 startup
```

3. **Configure Nginx:**
```nginx
# /etc/nginx/sites-available/yukbelajar
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/yukbelajar /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

4. **SSL Certificate (Let's Encrypt):**
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
```

### Option 4: Docker Deployment

#### Dockerfile
```dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package.json package-lock.json* ./
RUN npm ci

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

RUN npm run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# Set the correct permission for prerender cache
RUN mkdir .next
RUN chown nextjs:nodejs .next

COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

#### Docker Compose
```yaml
version: '3.8'
services:
  yukbelajar:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    restart: unless-stopped
```

#### Deploy with Docker
```bash
# Build and run
docker-compose up -d

# Or with Docker only
docker build -t yukbelajar-clone .
docker run -p 3000:3000 yukbelajar-clone
```

## 🔧 Environment Configuration

### Production Environment Variables

Create `.env.production`:
```env
# Site Configuration
NEXT_PUBLIC_SITE_URL=https://your-domain.com
NEXT_PUBLIC_API_URL=https://your-domain.com/api

# Security (generate secure values)
JWT_SECRET=your-super-secure-jwt-secret-here
NEXTAUTH_SECRET=your-nextauth-secret-here

# Database (if using real database)
DATABASE_URL=postgresql://user:password@localhost:5432/yukbelajar

# Email (if implementing real email)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password

# Analytics (optional)
GOOGLE_ANALYTICS_ID=GA_MEASUREMENT_ID
```

## 📊 Performance Optimization

### Build Optimization
```bash
# Analyze bundle size
npm run build
npm run analyze

# Enable compression
npm install compression
```

### Next.js Configuration
```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone', // For Docker
  compress: true,
  images: {
    domains: ['placehold.co'],
    formats: ['image/webp', 'image/avif'],
  },
  experimental: {
    optimizeCss: true,
  },
}

module.exports = nextConfig
```

## 🔍 Monitoring & Maintenance

### Health Checks
```bash
# Check application status
curl -f http://localhost:3000/api/health || exit 1

# PM2 monitoring
pm2 monit

# Check logs
pm2 logs yukbelajar
```

### Backup Strategy
```bash
# Backup user data (if using database)
pg_dump yukbelajar > backup_$(date +%Y%m%d).sql

# Backup application files
tar -czf app_backup_$(date +%Y%m%d).tar.gz /path/to/yukbelajar-clone
```

## 🚨 Troubleshooting

### Common Issues

1. **Port Already in Use:**
```bash
# Find process using port 3000
lsof -i :3000
# Kill process
kill -9 <PID>
```

2. **Build Failures:**
```bash
# Clear Next.js cache
rm -rf .next
npm run build
```

3. **Memory Issues:**
```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run build
```

4. **Permission Issues:**
```bash
# Fix file permissions
sudo chown -R $USER:$USER .
chmod -R 755 .
```

### Logs and Debugging

```bash
# Application logs
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# PM2 logs
pm2 logs yukbelajar --lines 100

# System logs
journalctl -u nginx -f
```

## 📈 Scaling Considerations

### Load Balancing
```nginx
upstream yukbelajar_backend {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}

server {
    location / {
        proxy_pass http://yukbelajar_backend;
    }
}
```

### Database Scaling
- Consider PostgreSQL for production
- Implement connection pooling
- Add Redis for caching
- Use CDN for static assets

## 🔐 Security Checklist

- [ ] Enable HTTPS/SSL
- [ ] Configure CORS properly
- [ ] Implement rate limiting
- [ ] Add security headers
- [ ] Regular security updates
- [ ] Environment variable protection
- [ ] Input validation and sanitization

## 📞 Support

If you encounter issues during deployment:

1. Check the logs first
2. Verify all environment variables
3. Ensure all dependencies are installed
4. Test API endpoints individually
5. Check firewall and port configurations

For additional help, create an issue in the repository with:
- Deployment platform
- Error messages
- System specifications
- Steps to reproduce

---

**Note**: This deployment guide covers multiple scenarios. Choose the option that best fits your infrastructure and requirements.

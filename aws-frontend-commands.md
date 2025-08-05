# Frontend EC2 Deployment - 9 Steps

## 🚀 **9 Steps to Deploy Frontend (smartspidy.smartbrew.in)**

### **Step 1: Launch EC2 Instance**
```bash
# Instance Type: t3.micro (Free Tier)
# AMI: Ubuntu 22.04 LTS
# Storage: 8GB GP3
# Security Group: HTTP(80), HTTPS(443), SSH(22)
```

### **Step 2: Connect to EC2**
```bash
ssh -i your-key.pem ubuntu@your-ec2-ip
sudo apt update && sudo apt upgrade -y
```

### **Step 3: Install Node.js**
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
node --version
npm --version
```

### **Step 4: Install nginx**
```bash
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

### **Step 5: Clone Repository**
```bash
git clone https://github.com/your-username/Smart-Spidy.git
cd Smart-Spidy
npm install
```

### **Step 6: Build Frontend**
```bash
npm run build
ls -la dist/
```

### **Step 7: Configure Environment**
```bash
# Create .env file for production
nano .env
```

**Add to .env:**
```bash
VITE_API_URL=https://api.smartbrew.in/api
NODE_ENV=production
```

### **Step 8: Configure nginx**
```bash
sudo nano /etc/nginx/sites-available/smartspidy-frontend
```

**Add this configuration:**
```nginx
server {
    listen 80;
    server_name smartspidy.smartbrew.in;
    root /home/ubuntu/Smart-Spidy/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### **Step 9: Enable Site & SSL**
```bash
sudo ln -s /etc/nginx/sites-available/smartspidy-frontend /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx

# Install SSL Certificate
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d smartspidy.smartbrew.in
```

---

## 📋 **Additional Commands & Updates**

### **DNS Configuration**
```bash
# Add A record in your DNS provider:
# Name: smartspidy
# Type: A
# Value: your-ec2-public-ip
# TTL: 300
```

### **Environment Variables**
```bash
# Create .env file in frontend
nano .env
```

**Add to .env:**
```bash
VITE_API_URL=https://api.smartbrew.in/api
NODE_ENV=production
```

### **Deployment Script**
```bash
# Create deployment script
nano deploy-frontend.sh
```

**Add deployment script:**
```bash
#!/bin/bash
echo "🚀 Deploying Smart-Spidy Frontend..."

# Pull latest changes
git pull origin main

# Install dependencies
npm install

# Build the project
npm run build

# Copy build to nginx directory
sudo cp -r dist/* /var/www/html/

# Set permissions
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/

# Reload nginx
sudo systemctl reload nginx

echo "✅ Frontend deployed successfully!"
```

### **Make Script Executable**
```bash
chmod +x deploy-frontend.sh
```

### **PM2 Process Manager (Optional)**
```bash
# Install PM2
sudo npm install -g pm2

# Create ecosystem file
nano ecosystem.config.js
```

**Add PM2 configuration:**
```javascript
module.exports = {
  apps: [{
    name: 'smartspidy-frontend',
    script: 'npm',
    args: 'run build',
    cwd: '/home/ubuntu/Smart-Spidy',
    instances: 1,
    autorestart: true,
    watch: false,
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'production'
    }
  }]
};
```

### **GitHub Actions Auto-Deployment**
```bash
# Create GitHub Actions workflow
mkdir -p .github/workflows
nano .github/workflows/deploy-frontend.yml
```

**Add GitHub Actions workflow:**
```yaml
name: Deploy Frontend to EC2

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Deploy to EC2
        uses: appleboy/ssh-action@v0.1.5
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            cd Smart-Spidy
            git pull origin main
            npm install
            npm run build
            sudo cp -r dist/* /var/www/html/
            sudo chown -R www-data:www-data /var/www/html/
            sudo systemctl reload nginx
```

### **Monitoring Commands**
```bash
# Check nginx status
sudo systemctl status nginx

# Check logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# Check disk usage
df -h

# Check memory usage
free -h

# Check CPU usage
htop
```

### **Performance Monitoring**
```bash
# Install monitoring tools
sudo apt install htop iotop -y

# Monitor real-time
htop
iotop
```

### **Troubleshooting Commands**

#### **1. nginx 502 Bad Gateway**
```bash
# Check if backend is running
curl http://localhost:3000/health

# Check nginx error logs
sudo tail -f /var/log/nginx/error.log
```

#### **2. Permission Denied**
```bash
# Fix permissions
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```

#### **3. SSL Certificate Issues**
```bash
# Renew certificate
sudo certbot renew

# Check certificate status
sudo certbot certificates
```

### **Security Headers (Optional)**
```bash
# Add security headers to nginx config
sudo nano /etc/nginx/sites-available/smartspidy-frontend
```

**Add to nginx config:**
```nginx
# Security headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;
add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
```

### **Gzip Compression (Optional)**
```bash
# Add gzip compression to nginx config
sudo nano /etc/nginx/sites-available/smartspidy-frontend
```

**Add to nginx config:**
```nginx
# Gzip compression
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;
```

### **Quick Deploy Commands**
```bash
# One-command deployment
cd Smart-Spidy && git pull && npm install && npm run build && sudo cp -r dist/* /var/www/html/ && sudo systemctl reload nginx

# Quick SSL renewal
sudo certbot renew --quiet

# Quick nginx reload
sudo systemctl reload nginx
```

### **Backup Commands**
```bash
# Backup nginx config
sudo cp /etc/nginx/sites-available/smartspidy-frontend /home/ubuntu/backup/

# Backup SSL certificates
sudo cp -r /etc/letsencrypt/ /home/ubuntu/backup/
```

---

## 🎯 **Final Checklist**

- [ ] EC2 instance launched (t3.micro)
- [ ] Node.js installed
- [ ] nginx configured
- [ ] Frontend built and deployed
- [ ] SSL certificate installed
- [ ] Domain configured (smartspidy.smartbrew.in)
- [ ] Environment variables set
- [ ] Deployment script created
- [ ] Monitoring setup

## 🚀 **Your site will be live at: https://smartspidy.smartbrew.in** 
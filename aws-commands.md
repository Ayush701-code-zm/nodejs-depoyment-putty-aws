# 🚀 Smart-Spidy Backend Deployment Commands

## 📋 Complete Backend Deployment Checklist

### **Step 1: System Update**
```bash
cd ~/Downloads
chmod 400 gmail-automation.pem
ssh -i "gmail-automation.pem" ubuntu@ec2-13-203-203-65.ap-south-1.compute.amazonaws.com
sudo apt update && sudo apt upgrade -y
```

### **Step 2: Install Node.js 18**
```bas
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### **Step 3: Verify Node.js**
```bash
node --version
npm --version
```

### **Step 4: Install PM2**
```bash
sudo npm install -g pm2
```

### **Step 5: Install nginx**
```bash
sudo apt install nginx -y
```

### **Step 6: Clone Repository**
```bash
git clone https://github.com/SMARTBREW/Smart-Spidy.git
cd Smart-Spidy/backend
```

### **Step 7: Install Dependencies**
```bash
npm install
```

### **Step 8: Create .env File**
```bash
nano .env
```

**Add these environment variables to .env:**
```env
# Database
SUPABASE_URL=your_supabase_url
SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key

# JWT
JWT_SECRET=your_jwt_secret
JWT_ACCESS_EXPIRATION_MINUTES=480
JWT_REFRESH_EXPIRATION_DAYS=30

# Server
PORT=3000
NODE_ENV=production

# AI Services (Optional)
OPENAI_API_KEY=your_openai_key
INSTAGRAM_ACCESS_TOKEN=your_instagram_token
INSTAGRAM_BUSINESS_ACCOUNT_ID=your_instagram_account_id
```


<!-- Step 9(a little difficult): Configure nginx (MISSING STEP!)
sudo nano /etc/nginx/sites-available/default

server {
    listen 80;
    server_name _;

    # Frontend
    location / {
        root /var/www/html;
        try_files $uri $uri/ /index.html;
    }

    # Backend API proxy
    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
} -->

### **Step 9: Configure nginx (Choose One Option)**

#### **Option A: Basic Setup (HTTP Only)**
```bash
sudo tee /etc/nginx/sites-available/default > /dev/null << 'EOF'
server {
    listen 80;
    server_name _;

    # Frontend
    location / {
        root /var/www/html;
        try_files $uri $uri/ /index.html;
    }

    # Health endpoint
    location /health {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Backend API proxy
    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
EOF
```

#### **Option B: Production Setup (HTTPS with Domain)**
```bash
# 1. Add DNS record in Route 53 (AWS Console)
# Record type: A
# Record name: api
# Value: your-ec2-ip
# TTL: 300

# 2. Wait for DNS propagation (5-10 minutes)
nslookup api.yourdomain.com

# 3. Install certbot
sudo apt install certbot python3-certbot-nginx -y

# 4. Get SSL certificate
sudo certbot --nginx -d api.yourdomain.com

# 5. If certbot fails to install certificate, update nginx config manually:
sudo tee /etc/nginx/sites-available/default > /dev/null << 'EOF'
server {
    listen 80;
    server_name api.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name api.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/api.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.yourdomain.com/privkey.pem;

    # Health endpoint
    location /health {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Backend API proxy
    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
EOF

# 6. Test and reload nginx
sudo nginx -t && sudo systemctl reload nginx

# 7. Install certificate manually (if needed)
sudo certbot install --cert-name api.yourdomain.com

# 8. Test SSL setup
curl https://api.yourdomain.com/health
```
```bash
# 1. Add DNS record in Route 53 (AWS Console)
# Record type: A
# Record name: api
# Value: your-ec2-ip
# TTL: 300

# 2. Wait for DNS propagation (5-10 minutes)
nslookup api.yourdomain.com

# 3. Install certbot
sudo apt install certbot python3-certbot-nginx -y

# 4. Get SSL certificate
sudo certbot --nginx -d api.yourdomain.com

# 5. If certbot fails to install certificate, update nginx config manually:
sudo tee /etc/nginx/sites-available/default > /dev/null << 'EOF'
server {
    listen 80;
    server_name api.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name api.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/api.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.yourdomain.com/privkey.pem;

    # Health endpoint
    location /health {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Backend API proxy
    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
EOF

# 6. Test and reload nginx
sudo nginx -t && sudo systemctl reload nginx

# 7. Install certificate manually (if needed)
sudo certbot install --cert-name api.yourdomain.com

# 8. Test SSL setup
curl https://api.yourdomain.com/health
```

### **Step 10: Test nginx Configuration**
```bash
sudo nginx -t
```

### **Step 11: Start nginx**
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl restart nginx
```

### **Step 12: Start Backend with PM2**
```bash
pm2 start index.js --name "smart-spidy-backend"
pm2 save
pm2 startup
```

### **Step 13: Complete PM2 Startup Setup**
```bash
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu
```

### **Step 14: Verify Everything**
```bash
# Check PM2 status
pm2 status

# Check nginx status
sudo systemctl status nginx

# Test direct backend access
curl http://localhost:3000/health

# Test nginx proxy
curl http://localhost/health
curl https://api.smartbrew.in/health

# Test API endpoint
curl http://localhost/api/notifications/stats
```

## 🔄 Update Deployment Commands

### **Pull Latest Code and Update Backend**
```bash
# Go to project directory
cd ~/Smart-Spidy

# Pull latest changes
git pull origin main

# Update backend
cd backend
npm install
pm2 restart smart-spidy-backend
```

### **Update Environment Variables**
```bash
# Edit .env file
nano ~/Smart-Spidy/backend/.env

# Restart backend
pm2 restart smart-spidy-backend
```

### **Check Logs**
```bash
# Check PM2 logs
pm2 logs smart-spidy-backend

# Check nginx logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

## 🧪 Testing Commands

### **Test Health Endpoint**
```bash
# Direct backend
curl http://localhost:3000/health

# Through nginx
curl http://localhost/health

# Public DNS
curl http://ec2-3-110-216-163.ap-south-1.compute.amazonaws.com/health
```

### **Test Notification System**
```bash
# Generate notifications
npm run generate-notifications

# Check notification stats
curl http://localhost/api/notifications/stats
```

### **Test API Endpoints**
```bash
# Test authentication required
curl http://localhost/api/notifications/stats

# Should return: {"success":false,"message":"Please authenticate"}
```

## 🚨 Troubleshooting Commands

### **Restart Services**
```bash
# Restart PM2
pm2 restart smart-spidy-backend

# Restart nginx
sudo systemctl restart nginx

# Restart both
pm2 restart smart-spidy-backend && sudo systemctl restart nginx
```

### **Check Service Status**
```bash
# Check PM2 status
pm2 status

# Check nginx status
sudo systemctl status nginx

# Check if ports are listening
sudo netstat -tlnp | grep :80
sudo netstat -tlnp | grep :3000
```

### **View Logs**
```bash
# PM2 logs
pm2 logs smart-spidy-backend

# nginx logs
sudo tail -f /var/log/nginx/error.log

# System logs
sudo journalctl -u nginx -f
```

## 📊 Monitoring Commands

### **Check Resource Usage**
```bash
# Check memory usage
free -h

# Check disk usage
df -h

# Check CPU usage
top

# Check PM2 resource usage
pm2 monit
```

### **Check Network**
```bash
# Check if nginx is listening
sudo ss -tlnp | grep :80

# Check if Node.js is listening
sudo ss -tlnp | grep :3000
```

## 🎯 Quick Verification Script

Create a verification script:
```bash
nano ~/verify-deployment.sh
```

**Add this content:**
```bash
#!/bin/bash
echo "🔍 Verifying Smart-Spidy Backend Deployment..."

echo "📊 PM2 Status:"
pm2 status

echo "🌐 nginx Status:"
sudo systemctl status nginx --no-pager

echo "🏥 Health Check:"
curl -s http://localhost/health

echo "🔐 API Test:"
curl -s http://localhost/api/notifications/stats

echo "✅ Verification Complete!"
```

**Make it executable:**
```bash
chmod +x ~/verify-deployment.sh
```

**Run verification:**
```bash
~/verify-deployment.sh
```

---

## 📝 Notes

- **Backend runs on:** Port 3000
- **nginx runs on:** Port 80
- **Health endpoint:** `/health`
- **API endpoints:** `/api/*`
- **PM2 process name:** `smart-spidy-backend`
- **Auto-startup:** Configured with systemd

## 🎉 Success Indicators

✅ **PM2 status shows:** `smart-spidy-backend` online  
✅ **nginx status shows:** `active (running)`  
✅ **Health endpoint returns:** `{"status":"OK",...}`  
✅ **API endpoint returns:** `{"success":false,"message":"Please authenticate"}`  

**Your backend is successfully deployed!** 🚀 

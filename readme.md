
StripeFlow Analytics - Help & Support Guide

📋 Quick Start Guide

Option 1: One-Click Install with Docker (Recommended)

1. Install Docker (if you don't have it):
   ```bash
   # For Windows/Mac: Download from https://www.docker.com/products/docker-desktop
   # For Linux:
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   ```
2. Download StripeFlow:
   ```bash
   # Download the ZIP file from your purchase
   # Extract it to a folder
   unzip stripeflow.zip
   cd stripeflow
   ```
3. Start StripeFlow:
   ```bash
   # Start the application
   docker-compose up -d
   
   # Check if it's running
   docker-compose ps
   ```
4. Access Your Dashboard:
   · Open browser and go to: http://localhost:3000
   · Backend API: http://localhost:3001
   · Health check: http://localhost:3001/api/health

Option 2: Manual Installation

1. Install Node.js 18+:
   ```bash
   # Check if Node.js is installed
   node --version
   
   # If not, install from: https://nodejs.org
   ```
2. Set up Backend:
   ```bash
   cd backend
   npm install
   npm start
   ```
3. Set up Frontend:
   ```bash
   cd frontend
   # No installation needed - just serve the files
   # Use any static file server, for example:
   npx serve .
   # or
   python3 -m http.server 3000
   ```
4. Access: Open browser to http://localhost:3000

---

🔑 Getting Your Stripe API Key

Step-by-Step Guide:

1. Login to Stripe Dashboard:
   · Go to: https://dashboard.stripe.com
   · Sign in to your account
2. Navigate to API Keys:
   · Click on "Developers" in the sidebar
   · Click on "API Keys"
3. Get Your Secret Key:
   · Look for "Secret key" (starts with sk_)
   · Click "Reveal test key" for testing
   · Click "Reveal live key" for production
4. Permission Check:
   Your key needs these permissions:
   · ✅ balance:read
   · ✅ charges:read
   · ✅ customers:read
   · ✅ subscriptions:read
   · ✅ balance_transactions:read

Test Mode vs Live Mode:

· Test Key (sk_test_...): Use for testing, no real money
· Live Key (sk_live_...): Use for real business data

Tip: Start with test mode to verify everything works!

---

🚨 Troubleshooting Guide

Common Issues & Solutions:

Issue 1: "Invalid API Key" Error

```
Error: Invalid Stripe API key or insufficient permissions
```

Solutions:

1. Check if key starts with sk_
2. Ensure you copied the entire key (no spaces before/after)
3. Try generating a new key in Stripe Dashboard
4. Test with a test key first: sk_test_...

Issue 2: Docker Container Won't Start

```
Error: Port 3000 is already in use
```

Solutions:

1. Change ports in docker-compose.yml:
   ```yaml
   ports:
     - "4000:3000"  # Change 4000 to any free port
     - "4001:3001"
   ```
2. Stop existing services on those ports:
   ```bash
   # Find what's using port 3000
   sudo lsof -i :3000
   
   # Kill the process
   sudo kill -9 [PID]
   ```

Issue 3: "Connection Failed" Error

```
Failed to connect to backend server
```

Solutions:

1. Check if backend is running:
   ```bash
   curl http://localhost:3001/api/health
   # Should return: {"status":"healthy"}
   ```
2. Check Docker logs:
   ```bash
   docker-compose logs backend
   ```
3. Check firewall/antivirus isn't blocking connections

Issue 4: No Data Showing

Dashboard loads but shows $0 for everything.

Solutions:

1. Test mode check: Are you using test key but expecting live data?
2. Date range: Have you processed any payments in the last 30 days?
3. Permission check: Does your API key have read permissions?
4. Business type: Do you have active subscriptions (for MRR)?

Issue 5: High Memory Usage

Solutions:

1. Add memory limits in Docker:
   ```yaml
   services:
     backend:
       deploy:
         resources:
           limits:
             memory: 512M
   ```
2. Clear browser cache
3. Restart containers:
   ```bash
   docker-compose restart
   ```

Issue 6: Chart Not Loading

Solutions:

1. Check browser console for JavaScript errors (F12 → Console)
2. Ensure Chart.js is loading (check Network tab)
3. Try different browser (Chrome/Firefox recommended)
4. Disable browser extensions that might interfere

---

🔧 Configuration Options

Environment Variables (Backend)

Create .env file in backend/ folder:

```env
PORT=3001
NODE_ENV=production
CORS_ORIGIN=http://localhost:3000
RATE_LIMIT_WINDOW=15
RATE_LIMIT_MAX=100
```

Customizing the Dashboard

Change Branding:

1. Edit frontend/index.html:
   ```html
   <!-- Change logo text -->
   <span>YourBrand Analytics</span>
   ```
2. Edit frontend/styles.css:
   ```css
   :root {
     --primary: #YOUR_COLOR;  /* Change primary color */
   }
   ```

Change Ports:

1. Edit docker-compose.yml:
   ```yaml
   ports:
     - "8080:3000"  # Frontend port
     - "8081:3001"  # Backend port
   ```
2. Update frontend/dashboard.js:
   ```javascript
   this.apiBaseUrl = 'http://localhost:8081/api';
   ```

---

📊 Data Refresh & Caching

Manual Refresh:

· Click "Refresh" button in top-right
· Or press F5 in browser

Automatic Refresh:

Add to frontend/dashboard.js:

```javascript
// Auto-refresh every 5 minutes
setInterval(() => {
    this.loadDashboardData();
}, 5 * 60 * 1000);
```

Clear Cache:

```bash
# Docker cache
docker-compose down
docker system prune -f
docker-compose up -d

# Browser cache: Ctrl+Shift+Delete
```

---

🔒 Security Best Practices

For Production Use:

1. Use HTTPS:
   ```bash
   # With Nginx reverse proxy
   # Add SSL certificate (Let's Encrypt)
   ```
2. Restrict Access:
   ```nginx
   # Nginx basic auth
   location / {
       auth_basic "Restricted";
       auth_basic_user_file /etc/nginx/.htpasswd;
   }
   ```
3. Firewall Rules:
   ```bash
   # Only allow specific IPs
   ufw allow from YOUR_IP to any port 3000
   ```
4. Regular Updates:
   ```bash
   # Update Docker images
   docker-compose pull
   docker-compose up -d
   ```

---

📈 Performance Optimization

For Large Accounts (1000+ customers):

1. Increase API limits:
   ```javascript
   // In backend/server.js, increase limits:
   stripe.customers.list({ limit: 1000 });
   ```
2. Add pagination:
   ```javascript
   // Fetch data in batches
   async function getAllCustomers(stripe) {
     let allCustomers = [];
     let hasMore = true;
     let startingAfter = null;
   
     while (hasMore) {
       const customers = await stripe.customers.list({
         limit: 100,
         starting_after: startingAfter
       });
       
       allCustomers = allCustomers.concat(customers.data);
       hasMore = customers.has_more;
       startingAfter = customers.data[customers.data.length - 1]?.id;
     }
   
     return allCustomers;
   }
   ```
3. Enable compression:
   ```javascript
   // In backend/server.js
   const compression = require('compression');
   app.use(compression());
   ```

---

🐛 Debugging Tips

Enable Debug Mode:

```javascript
// In frontend/dashboard.js constructor
this.debugMode = true;

// Then add debug logging
if (this.debugMode) {
    console.log('API Response:', data);
}
```

Check Logs:

Docker Logs:

```bash
# See all logs
docker-compose logs

# Follow logs in real-time
docker-compose logs -f

# Check specific service
docker-compose logs backend
```

Browser Developer Tools:

1. Press F12
2. Check "Console" tab for errors
3. Check "Network" tab for failed requests
4. Check "Application" tab for localStorage

Stripe Logs:

1. Go to Stripe Dashboard → Developers → Logs
2. Check API request history
3. Look for permission errors

---

🤝 Getting Help

Before Contacting Support:

1. Check this guide for your issue
2. Verify Stripe API key is correct
3. Check Docker is running: docker ps
4. Try test mode with sk_test_... key
5. Check browser console for errors

Contact Support:

When contacting support, include:

1. Error message (exact text)
2. Stripe mode: Test or Live
3. Installation method: Docker or Manual
4. Operating System: Windows/Mac/Linux
5. Browser: Chrome/Firefox/etc.

Support Email: support@stripeflow.com
Response Time: 24-48 hours (Business days)

---

🔄 Updates & Maintenance

Update to New Version:

```bash
# 1. Backup your data
cp -r stripeflow stripeflow-backup

# 2. Download new version
# 3. Compare and merge customizations
# 4. Deploy new version
docker-compose down
docker-compose up -d --build
```

Backup Your Configuration:

```bash
# Important files to backup:
- backend/.env
- docker-compose.yml
- Any custom CSS/HTML changes
- Your Stripe API key (store securely)
```

---

❓ Frequently Asked Questions

Q: Is my Stripe data safe?

A: Yes! Your API key never leaves your server. All data processing happens locally.

Q: Can I use this with multiple Stripe accounts?

A: Pro and Agency licenses support multiple accounts. Basic license is for one account.

Q: What happens when my update period expires?

A: The software continues working. You just won't get new feature updates (security updates still included).

Q: Can I customize the dashboard?

A: Yes! All licenses allow CSS/HTML customization. Pro and Agency include white-label options.

Q: Does this work with Stripe Connect?

A: Yes, if your API key has access to the connected account data.

Q: Can I run this on a VPS?

A: Yes! Any server with Docker or Node.js support.

Q: What about rate limits?

A: The app respects Stripe's rate limits (100 requests/second). For large accounts, consider implementing caching.

---

🚀 Advanced: Deployment Options

Deploy to VPS (DigitalOcean, Linode, AWS):

```bash
# 1. Connect to your VPS
ssh user@your-server-ip

# 2. Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 3. Install Docker Compose
sudo apt-get install docker-compose

# 4. Transfer files
scp -r stripeflow user@your-server-ip:/home/user/

# 5. Start
cd stripeflow
docker-compose up -d
```

Deploy with PM2 (Manual Install):

```bash
# Install PM2
npm install -g pm2

# Start backend with PM2
cd backend
pm2 start server.js --name stripeflow-backend

# Save PM2 configuration
pm2 save
pm2 startup
```

Set up Domain with SSL:

```nginx
# Nginx configuration
server {
    listen 80;
    server_name analytics.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name analytics.yourdomain.com;
    
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location /api/ {
        proxy_pass http://localhost:3001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

📞 Emergency Recovery

If Dashboard Won't Load:

1. Check basic connectivity:
   ```bash
   # Can you access the health endpoint?
   curl http://localhost:3001/api/health
   
   # Check Docker
   docker ps
   docker-compose logs
   ```
2. Restart everything:
   ```bash
   docker-compose down
   docker system prune -f
   docker-compose up -d
   ```
3. Clear browser data:
   · Press Ctrl+Shift+Delete
   · Clear cache and cookies
   · Restart browser

If Data Seems Wrong:

1. Check Stripe Dashboard directly
2. Verify API key permissions
3. Try test mode vs live mode
4. Check date filters

---

Remember: Always test with a Stripe test key (sk_test_...) before using your live key!

---

Last Updated: [Date]
Version: 1.0.0
Need more help? Email: support@stripeflow.com
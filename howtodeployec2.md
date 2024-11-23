# How to Deploy Medusa.js API to Amazon EC2

## Prerequisites
- An AWS account with EC2 access
- EC2 instance running Amazon Linux 2023
- Security group configured to allow inbound traffic on port 80 (for nginx)
- PEM key file for SSH access

## Deployment Steps

### 1. Transfer Files to EC2
```bash
# Transfer the project zip file to EC2
scp -i medusa-pem.pem solace-medusa-starter-api.zip ec2-user@54.167.220.210:/home/ec2-user/
```

### 2. SSH into EC2
```bash
ssh -i medusa-pem.pem ec2-user@54.167.220.210
```

### 3. Setup Node.js
```bash
# Install n (Node version manager)
sudo npm install -g n

# Install Node.js 20 using n
sudo n 20

# Refresh shell PATH
hash -r
```

### 4. Install and Configure Redis
```bash
# Install Redis
sudo dnf install -y amazon-linux-extras
sudo amazon-linux-extras enable redis6
sudo dnf install -y redis6

# Start and enable Redis service
sudo systemctl start redis6
sudo systemctl enable redis6
```

### 5. Install nginx
```bash
# Install nginx
sudo dnf install -y nginx

# Create nginx configuration
sudo nano /etc/nginx/conf.d/medusa.conf
```

Add this configuration to the file:
```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Start and enable nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 6. Setup Project
```bash
# Unzip the project files
unzip solace-medusa-starter-api.zip

# Install Yarn globally
npm install -g yarn

# Install project dependencies using Yarn
cd solace-medusa-starter-api
yarn install
```

### 7. Start the Application
```bash
# Start the Medusa application
yarn start
```

## Troubleshooting

### If you encounter "Bus error (core dumped)"
This might be related to SWC compatibility issues. Try removing and reinstalling the SWC module:
```bash
rm -rf node_modules/@swc
npm install @swc/core@latest --platform=linux --arch=x64
```

### If you encounter PostgreSQL related errors
Make sure to use Yarn for dependency installation as it handles the dependencies better with the RC versions of Medusa packages:
```bash
rm -rf node_modules package-lock.json
yarn install
```

## Notes
- The application runs on port 8080 and is proxied through nginx on port 80
- Redis is used for the event bus instead of the fake bus used in development
- Make sure all environment variables in .env are properly configured for production

## Maintenance
To update the application:
1. Make changes locally
2. Zip the updated files
3. Transfer to EC2 using scp
4. Unzip and restart the application

## Security Considerations
- Keep your PEM key secure and never commit it to version control
- Regularly update system packages using `sudo dnf update`
- Monitor logs for any issues: `journalctl -u nginx` for nginx logs



# Install PM2
npm install -g pm2

# Start the application
pm2 start "npm run start" --name medusa-api

# Make it start on system boot
pm2 startup
pm2 save



PM2 will:

Keep the process running after terminal closes
Automatically restart on crashes
Start automatically after server reboots
Provide process monitoring and logs
You can check if it's running with:

pm2 status
pm2 logs medusa-api
# Step by Step process

1. Initial VPS Setup:
```bash
# SSH into your VPS
ssh root@your_vps_ip

# Update system
sudo apt update && sudo apt upgrade -y

# Create non-root user
adduser yourusername
usermod -aG sudo yourusername

# Add user to docker group (will create after Docker installation)
sudo usermod -aG docker yourusername
```

2. Install Docker & Docker Compose:
```bash
# Install Docker
curl -fsSL https://get.docker.com | sh

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify installations
docker --version
docker-compose --version
```

3. Setup Project Structure:
```bash
# Create project directory
mkdir -p /var/www/yourdomain
cd /var/www/yourdomain

# Create necessary files
touch docker-compose.yml
touch .env

# Setup environment variables
cat << EOF > .env
NODE_ENV=production
NEXT_PUBLIC_API_URL=https://api.yourdomain.com
DB_PASSWORD=your_secure_password
EOF
```

4. Configure Nginx (Reverse Proxy):
```bash
# Install Nginx
sudo apt install nginx

# Create Nginx configuration
sudo nano /etc/nginx/sites-available/yourdomain.com

# Add configuration
server {
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# Enable the site
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

5. Deploy Your Application:
```bash
# Clone your repository
git clone your_repo_url .

# Build and start containers
docker-compose up -d --build

# Check status
docker-compose ps
docker-compose logs -f
```

6. Setup SSL with Certbot:
```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Get SSL certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

7. Basic Monitoring:
```bash
# Create a monitoring script
cat << EOF > monitor.sh
#!/bin/bash
echo "Docker Container Status:"
docker ps
echo "\nContainer Logs (last 50 lines):"
docker-compose logs --tail=50
echo "\nSystem Resources:"
df -h
free -m
top -bn1 | head -20
EOF

chmod +x monitor.sh
```

8. Setup Auto-restart:
```bash
# Create a restart script
cat << EOF > restart.sh
#!/bin/bash
cd /var/www/yourdomain
docker-compose down
docker-compose up -d
EOF

chmod +x restart.sh

# Add to crontab for automatic weekly restart
(crontab -l 2>/dev/null; echo "0 4 * * 0 /var/www/yourdomain/restart.sh") | crontab -
```

9. Backup Strategy:
```bash
# Create backup script
cat << EOF > backup.sh
#!/bin/bash
BACKUP_DIR="/backup/$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR

# Backup docker-compose and env files
cp docker-compose.yml $BACKUP_DIR/
cp .env $BACKUP_DIR/

# Backup nginx config
cp /etc/nginx/sites-available/yourdomain.com $BACKUP_DIR/

# Backup database (if using database container)
docker-compose exec -T db pg_dump -U postgres database > $BACKUP_DIR/database.sql

# Compress backup
cd /backup
tar -czf "backup_$(date +%Y%m%d).tar.gz" "$(date +%Y%m%d)"
rm -rf "$(date +%Y%m%d)"
EOF

chmod +x backup.sh
```

10. Firewall Setup:
```bash
# Configure UFW
sudo ufw allow OpenSSH
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

11. Common Commands for Management:
```bash
# View logs
docker-compose logs -f

# Restart services
docker-compose restart

# Update containers
git pull
docker-compose down
docker-compose up -d --build

# Check container resources
docker stats

# Clean up unused resources
docker system prune
```

11. Setup GitHub Actions for CI/CD:
```yaml
# .github/workflows/deploy.yml
name: Deploy Application

on:
  push:
    branches: [main, 'feature/*']

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set environment based on branch
        run: |
          if [[ $GITHUB_REF == refs/heads/main ]]; then
            echo "DEPLOY_PATH=/var/www/yourdomain" >> $GITHUB_ENV
            echo "DOMAIN=yourdomain.com" >> $GITHUB_ENV
          else
            FEATURE_NAME=$(echo $GITHUB_REF | sed 's/refs\/heads\/feature\///')
            echo "DEPLOY_PATH=/var/www/$FEATURE_NAME" >> $GITHUB_ENV
            echo "DOMAIN=$FEATURE_NAME.yourdomain.com" >> $GITHUB_ENV
          fi

      - name: Deploy to VPS
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USERNAME }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            # Create directory if not exists
            mkdir -p ${{ env.DEPLOY_PATH }}
            cd ${{ env.DEPLOY_PATH }}
            
            # Pull latest changes
            if [ -d .git ]; then
              git pull
            else
              git clone ${{ github.repository }} .
            fi
            
            # Setup environment
            if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
              cp /etc/secrets/.env.prod .env
            else
              cp /etc/secrets/.env.staging .env
            fi
            
            # Deploy
            docker-compose down
            docker-compose up -d --build
            
            # Setup Nginx for feature branch (if not main)
            if [[ "${{ github.ref }}" != "refs/heads/main" ]]; then
              sudo bash -c "cat > /etc/nginx/sites-available/${{ env.DOMAIN }} << 'EOL'
              server {
                  server_name ${{ env.DOMAIN }};
                  location / {
                      proxy_pass http://localhost:3000;
                      proxy_http_version 1.1;
                      proxy_set_header Upgrade \$http_upgrade;
                      proxy_set_header Connection 'upgrade';
                      proxy_set_header Host \$host;
                  }
              }
              EOL"
              
              sudo ln -sf /etc/nginx/sites-available/${{ env.DOMAIN }} /etc/nginx/sites-enabled/
              sudo nginx -t && sudo systemctl reload nginx
              
              # Setup SSL for feature branch
              sudo certbot --nginx -d ${{ env.DOMAIN }} --non-interactive --agree-tos -m your@email.com
            fi
```

12. Setup VPS for GitHub Actions:
```bash
# 1. Generate SSH key for GitHub Actions
ssh-keygen -t rsa -b 4096 -C "github-actions"

# 2. Add public key to authorized_keys
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# 3. Copy private key (add to GitHub Secrets)
cat ~/.ssh/id_rsa
```

13. Setup GitHub Secrets:
```bash
# Add these secrets in GitHub repository settings
VPS_HOST=your_vps_ip
VPS_USERNAME=your_username
VPS_SSH_KEY=your_private_key
```

14. Setup DNS for Feature Branches:
```bash
# Add wildcard DNS record
# *.yourdomain.com -> your_vps_ip
```

15. Directory Structure for Secrets:
```bash
# Create secrets directory
sudo mkdir /etc/secrets
sudo nano /etc/secrets/.env.prod
sudo nano /etc/secrets/.env.staging

# Secure the directory
sudo chmod 600 /etc/secrets/.env.*
```

16. Cleanup Script for Feature Branches:
```bash
# cleanup.sh
#!/bin/bash
# Cleanup old feature deployments
for domain in $(ls /etc/nginx/sites-available/); do
  if [[ $domain =~ ^feature- ]]; then
    # Check if GitHub branch exists
    BRANCH="feature/${domain#feature-}"
    if ! git ls-remote --heads origin $BRANCH | grep -q $BRANCH; then
      echo "Removing deployment for $domain"
      sudo rm /etc/nginx/sites-available/$domain
      sudo rm /etc/nginx/sites-enabled/$domain
      sudo rm -rf /var/www/$domain
      docker-compose -f /var/www/$domain/docker-compose.yml down
    fi
  fi
done
```


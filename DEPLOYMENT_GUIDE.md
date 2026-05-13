# Connect to EC2
ssh -i your-key.pem ubuntu@your-ec2-public-ip

# Update packages
sudo apt update -y

# Install nginx and git
sudo apt install nginx git -y

# Start and enable nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Go to home directory
cd ~

# Clone your GitHub repo (creates CC-Static folder)
git clone https://github.com/adityabhalgat/CC-Static.git

# Remove default nginx files
sudo rm -rf /var/www/html/*

# Copy your project files to nginx root
sudo cp -r ~/CC-Static/* /var/www/html/

# Set permissions
sudo chmod -R 755 /var/www/html

# Restart nginx
sudo systemctl restart nginx

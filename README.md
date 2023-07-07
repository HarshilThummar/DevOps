#!/bin/bash

# Function to check if a command is available
command_exists() {
  command -v "$1" >/dev/null 2>&1
}

# Check if Docker is installed
#if ! command_exists docker; then
#  echo "Docker is not installed. Installing Docker..."
  # Install Docker
  # Add your installation steps here
#fi

# Check if Docker Compose is installed
#if ! command_exists docker-compose; then
#  echo "Docker Compose is not installed. Installing Docker Compose..."
  # Install Docker Compose
  # Add your installation steps here
#fi

# Function to create a WordPress site
create_wordpress_site() {
  # Get the site name from the command-line argument
  command-line="$1"

  # Check if the site name is provided
  if [ -z "command-line" ]; then
    echo "Please provide a site name as a command-line argument."
    exit 1
  fi

  # Create a docker-compose.yml file for the LEMP stack
  cat <<EOF >docker-compose.yml
version: '3'
services:
  nginx:
    image: nginx
    ports:
      - 80:80
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/html:/usr/share/nginx/html
  php:
    image: php:fpm
    volumes:
      - ./php:/var/www/html
  mysql:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
EOF

  # Create Nginx configuration file
  mkdir -p nginx/conf.d
  cat <<EOF >nginx/conf.d/default.conf
server {
    listen 80;
    server_name $site_name;
    root /usr/share/nginx/html;
    index index.php;

    location / {
        try_files \$uri \$uri/ /index.php?\$args;
    }

    location ~ \.php$ {
        fastcgi_pass php:9000;
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
        include fastcgi_params;
    }
}
EOF

  # Create index.php file
  mkdir -p php
  echo "<?php phpinfo();" > php/index.php

  # Start the containers
  docker-compose up -d

  # Add /etc/hosts entry
  echo "127.0.0.1 example.com" | sudo tee -a /etc/hosts

  # Open the site in a browser
  echo "Site created successfully. Opening example.com in a browser..."
  xdg-open "http://example.com" >/dev/null 2>&1

  # Prompt the user to open the site in a browser
  read -p "Press Enter to open the site in a browser"
  xdg-open "http://example.com" >/dev/null 2>&1
}

# Function to enable/disable the site
toggle_site() {
  # Get the site name from the command-line argument
  command-line="$1"

  # Check if the site name is provided
  if [ -z "command-line" ]; then
    echo "Please provide a site name as a command-line argument."
    exit 1
  fi

  # Check if the site is currently enabled or disabled
  if docker-compose ps | grep -q "$site_name"; then
    # Site is enabled, stop the containers
    docker-compose stop
    echo "Site disabled."
  else
    # Site is disabled, start the containers
    docker-compose start
    echo "Site enabled
    }

# Call the toggle_site function with the provided command-line argument
toggle_site "$1"

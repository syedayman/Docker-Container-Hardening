# Webserver Dockerfile

# Use almalinux 9.2 minimal as the base image for a lightweight setup
FROM almalinux:9.2-minimal

# Install necessary packages:
# - microdnf is used to install yum package manager
# - Install epel repository for additional packages
# - Update the system for security and bug fixes
# - Install nginx, php and related modules for web server functionality
RUN microdnf install yum -y && \  
    yum install epel-release -y && \
    yum update -y && \
    yum install -y \
    nginx \
    php \
    php-mysqlnd \
    php-fpm 

# Copy the website files and various configuration files from the host to the container:
COPY webfiles/ /usr/share/nginx/html
COPY configfiles/nginx.conf     /etc/nginx/nginx.conf
COPY configfiles/php.ini        /etc/php.ini
COPY configfiles/www.conf       /etc/php-fpm.d/www.conf
COPY configfiles/php-fpm.conf   /etc/nginx/conf.d/php-fpm.conf
COPY configfiles/docker-entrypoint.sh /

# Remove the default nginx HTML files and set executable permissions on the entrypoint script
# Change ownership of php files to the 'apache' user for proper permissions
RUN rm -f /usr/share/nginx/html/index.html /usr/share/nginx/html/nginx-logo.png /usr/share/nginx/html/poweredby.png
RUN chmod +x /docker-entrypoint.sh && \
    chown apache:apache /usr/share/nginx/html/*.php

# Expose port 80 to enable HTTP traffic to the web server
EXPOSE 80

# Set the entrypoint to the script, to start the required services
ENTRYPOINT ["/docker-entrypoint.sh"]

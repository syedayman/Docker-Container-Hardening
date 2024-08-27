# Database Dockerfile

# Use the MariaDB image version 11.3.2 as the base image
FROM mariadb:11.3.2

# Create a system user 'mysql' for running the MariaDB process. 
# The '-r' flag indicates it's a system account. 
# The '|| true' ensures this command doesn't fail even if the user already exists.
RUN useradd -r mysql || true 

# Switch to the 'mysql' user for subsequent commands and for running the container.
# improves security by not running the database as the root user.
USER mysql

# Copy the custom MySQL configuration file from the host to the container.
# This configuration file is placed in the appropriate directory for the database to read.
COPY mysqld.cnf /mysql/mysql.conf.d/mysqld.cnf

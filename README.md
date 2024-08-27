# Docker-Container-Hardening-Project
This project hardens container images for a basic web application, consisting of a web server container and a database server container.

![image](https://github.com/user-attachments/assets/de5c2958-a9ab-4d92-a386-babe18248d15)

## Objective
- Refine poorly configured Dockerfiles and associated config files so that hardened images may be created to securely run the application.

## Files 
The original database Dockerfile is shown below.
![image](https://github.com/user-attachments/assets/f0e0f604-5e1f-4da5-9bab-3e3b3a8cbffd)

The original webserver Dockerfile is shown below.
![image](https://github.com/user-attachments/assets/f1ae6486-9ef5-41c2-a832-75816cd8256d)
![image](https://github.com/user-attachments/assets/edbf881d-a5ba-4708-931a-e9d24d6742ec)


## Steps
### Container Hardening
The refined Dockerfile of the database image is provided below.

![image](https://github.com/user-attachments/assets/435c3646-495c-43fd-aaba-f433c5e60194)

The refined Dockerfile firstly uses an updated and specific version of the base image. It has been changed from mariadb:10 to mariadb:11.3.2, as using the most up to date base image, while maintaining a specific version, strikes a balance between staying current with security patches and ensuring consistent, reliable behaviour of the database server.

The refined Dockerfile creates a new system user called mysql with the -r flag to run it as a system account which is just used for running services (as opposed to a user created for interactive use) to make sure the container is run as an unprivileged user to adhere to the principle of least privilege. The USER mysql command ensures all subsequent processes running in the container are run under this user to prevent an attacker from having root level access if the container was compromised. 

The refined Dockerfile removes the EXPOSE 3306 command to reduce the attack surface by preventing port access from inside the containers, especially since it is not necessary for the operation of the database. 

The refined Dockerfile of the webserver image is provided below.

![image](https://github.com/user-attachments/assets/4ebf9a4c-81c5-4f07-8725-73bf4dacdd18)

The refined Dockerfile replaces the discontinued centos:7 base image with a minimal, community-driven, and robust base image called almalinux:9.2-minimal. The minimal installation ensures a reduced attack surface as it includes fewer installed packages meaning fewer vulnerabilities as well as increased performance. However, the yum package manager had to be explicitly installed prior to other commands as it is not included in almalinux-minimal by default.

The refined Dockerfile removes the commands installing an SSH server and generating SSH keys as a means to further decrease the attack surface. Furthermore, enabling password authentication for SSH access is a less secure option as opposed to key based authentication. Using docker exec is the recommended approach to execute commands in a container eliminating the need to remotely log into them via SSH. It is best practice to implement one process per container, thus running additional services here like an SSH server is discouraged and exposes the application to more potential vulnerabilities. As a result, the /usr/sbin/sshd -D command was removed from docker-entrypoint.sh file as well.

The refined Dockerfile removes the created users as they could be a potential entry point for unauthorized access. Running the web container as an unprivileged user could not be implemented as a functionality and this is a limitation. Hence, removing unused user accounts having weak passwords in plaintext means limiting the potential vectors for access to the containers.

The refined Dockerfile removes the commands to expose ports 8004, 2375, and 22 as it reduces the attack surface by limiting port access from inside the containers, especially since exposing these ports are not necessary for the operation of the web server. 

To achieve greater efficiency and improved functionality, Docker Compose was utilized instead of Makefiles to allow for a more streamlined and cohesive management of the multi-container application.

The one-time configuration needed for setting up the environment is achieved with the command docker-compose build. This command builds the images as specified in the Dockerfile, downloading necessary packages, and preparing the environment as defined. Once this initial setup is complete, every subsequent start of the application is done using docker-compose up. This command brings up all the required services and containers outlined in the compose file, deftly handling both the initial launch and routine startups. To stop the application, docker-compose down is employed to gracefully halt all services and perform necessary cleanup of the environment.The docker-compose.yml file used to build and run the containers is illustrated below followed by a detailed description of solely its security features.

![image](https://github.com/user-attachments/assets/74b1b4a9-51a8-4c13-abcf-6e9e4481524c)
![image](https://github.com/user-attachments/assets/d0de5b65-ed54-4774-9639-00db99a301bb)

The name of the dbserver container is defined and the user of the container is set to mysql which was the user added in the Dockerfile of the database image. This is done to ensure that the container is not run as root as this can pose security risks due to the provision of escalated privileges.

The env_file (environment files) directive is used to specify a file containing the environment variables required to run the database server. In this case the environment variables include sensitive information such as database and user credentials including details of MYSQL_ROOT_PASSWORD, MYSQL_DATABASE, MYSQL_USER, and MYSQL_PASSWORD. To refrain from storing passwords in plaintext in the compose file, a separate environment file called vars.env was created and referenced to store the credentials required to initialize the database server. The ownership of the file was changed to the root user and file permissions set so that only the root user can read and write the file (demonstrated in video submission). 

The volumes directive executes two commands, first it mounts a file csvs23db.sql from the host into the container at /docker-entrypoint-initdb.d so that the SQL script is automatically executed when the container is started. The mount is set as read-only (:ro) so that the original file on the host cannot be changed from within the container. Then it creates a volume bind mount data and mounts it at the specified location in the container at /var/lib/mysql. In this way the data stored in the database remains intact across container restarts and updates, thus implementing data persistence. The mounted volume is managed by docker and is separate from the host file system hence providing a layer of abstraction and security.

The compose file keeps the network declaration and network subnet definitions (203.0.113.0/24) from the Makefiles but removes the static IP addresses allocated to the database and webserver containers. This is done so that Docker's dynamic IP allocation can be used instead to reduce predictability as static IPs can make containers more susceptible targets for network-based attacks. Automatic DNS resolution using service names, strengthens network isolation and encapsulation practices to create a more flexible and secure network environment.

The security_opt directive implements two security options for the containers. The first one is the addition of a Seccomp profile which is a Linux kernel feature that restricts the system calls that can be made by a process reducing the risk of kernel exploits or other vulnerabilities by limiting the potential actions and behaviours of the container. Both the containers use the same general default seccomp profile provided by Docker. It is designed to act as a secure default modular component that can be implemented without compromising usability. The second option sets the no-new-privileges flag to true to instruct the kernel to disallow the main container process, and its children, running in the container from gaining any additional privileges. It prevents privilege escalation attacks within the container as an attacker cannot gain additional privileges beyond those the container was initially granted even if they manage to access the container.

The cap_drop and cap_add directives are used to implement fine-grained control over what Linux capabilities are assigned to a container. Both containers use cap_drop: - ALL to first remove all default privileges typically given to a container, and then add only the capabilities necessary for the container to function. This significantly reduces the potential actions that the container's processes can perform hence limiting the risk of privilege escalation or abuse if the container is compromised. The dbserver capabilities include:
- CAP_DAC_OVERRIDE: Allowing the process to bypass file read, write, and execute permission checks.
- CAP_SETGID: Allowing the process to manipulate process GIDs and supplementary GID list.
- CAP_SETUID: Allows the process to manipulate process UIDs.

The webserver capabilities include the ones defined above as well as:
- CAP_NET_RAW: Allowing the container to use raw and packet sockets, useful for implementing custom network protocols.
- CAP_NET_BIND_SERVICE: Allowing the container to bind to network ports numbered below 1024 as only the root user can bind to these ports. 
- CAP_CHOWN: Allowing to change the owner of files and directories.

Cgroups is a Linux kernel feature that allows for allocation and management of system resources like CPU, memory, and I/O bandwidth for a group of processes. Docker uses cgroups to enforce limits and constraints on the resources a container can use. The cpus directive limits the container to using only 50% of a single CPU core. It helps in preventing any single container from overusing the CPU resources. mem_limit restricts the dbserver container to a maximum of 500 MB of memory and webserver to 256 MB as they are more lightweight compared to database servers. pids_limit sets a cap on the number of processes that can be created inside the container. This limit is a defence against fork bombs and similar denial-of-service attacks where a process continually spawns new processes until system resources are fully depleted. By limiting resources per container, you prevent any single container from monopolizing system resources, ensuring other containers and the host system remain stable and responsive. Resource limits protect against attacks such as fork bombs, memory exhaustion, or cryptojacking where a malicious process tries to consume all available system resources. It also provides container isolation such that if a container is compromised, these limits can help contain the impact and prevent the compromised container from affecting other containers or the host system. 


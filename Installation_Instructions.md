# Installation and Configuration Guide: Docker, OHDSI Broadsea, and PostgreSQL on Docker

[![Docker](https://img.shields.io/badge/Docker-20.10%2B-2496ED)](https://www.docker.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15%2B-336791)](https://www.postgresql.org/)
[![WSL2](https://img.shields.io/badge/WSL2-Required-4d4d4d)](https://learn.microsoft.com/en-us/windows/wsl/install)

A comprehensive guide to deploying the OHDSI ecosystem using Docker. Includes PostgreSQL configuration, data migration workflows, and WebAPI setup.

---

## Table of Contents
1. [Prerequisites](#Prerequisites)
2. [Docker Installation](#Docker Installation)
3. [OHDSI Broadsea Setup](#ohdsi-broadsea-setup)
4. [PostgreSQL Management](#postgresql-management)
5. [Data Migration](#data-migration)
6. [WebAPI Configuration](#webapi-configuration)
7. [Troubleshooting](#troubleshooting)
8. [License](#license)

---


## Prerequisites <a name="prerequisites"></a>

### Windows Users
1. Install WSL2: [Official Installation Guide](https://learn.microsoft.com/en-us/windows/wsl/install)
2. Install Ubuntu 22.04 from Microsoft Store as the preferred WSL distribution.

### Ubuntu Users
If you are using Ubuntu natively (not via WSL), ensure you have an updated system before proceeding with the installation steps below.

> [!IMPORTANT] 
> Ensure that you have administrative privileges on your system before proceeding with the installation.

---

## 1. Docker Installation <a name="docker-installation"></a>
Follow the guide below to install Docker on your system. For additional details, refer to the official [Docker Installation Guide](https://docs.docker.com/get-docker/).

### Step 1:  Update Package Index

```bash
sudo apt-get update
```

### Step 2: Install Required Packages
```bash
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

```

### Step 3: Add Docker’s Official GPG Key

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### Step 4: Set Up the Stable Repository


```bash

echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```

### Step 5: Update Package Index Again

```bash
sudo apt-get update
```

### Step 6: Install Docker Engine

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io
```


### Step 7:  Add User to Docker Group

```bash
sudo usermod -aG docker $USER
```
Note: You may need to log out and log back in for the group changes to take effect


### Step 8:  Start Docker Service

```bash
sudo service docker start
```

### Step 9:  Verify Installation

```bash
sudo service docker status
```
> [!TIP] 
> If you encounter issues with Docker installation, refer to the official [Docker Troubleshooting Guide](https://docs.docker.com/get-docker/) for support.

> [!NOTE] 
> It is recommended to periodically update your Docker images and containers to ensure security and performance improvements.


## 2. Installing OHDSI Broadsea

OHDSI Broadsea is a Docker-based solution for deploying the OHDSI tool stack, including ATLAS and WebAPI, in a containerized environment. It simplifies the deployment and management of OHDSI components, making it easier to integrate with various database systems and analytical tools.

For the latest updates and releases, visit the official GitHub repository: [OHDSI Broadsea GitHub](https://github.com/OHDSI/Broadsea).

### Step 1: Clone OHDSI Broadsea Repository

```bash
git clone https://github.com/OHDSI/Broadsea.git

```

### Step 2: Start OHDSI Broadsea Containers

Navigate to the directory where this README.md file is located. In a command-line terminal, execute the following command to start the Broadsea Docker containers. If you are using Linux, you may need to prepend sudo to the command. Wait up to one minute for all containers to fully start.
```bash
cd Broadsea
docker compose --profile default up -d
```
In the web browser, open the URL "https://127.0.0.1"

#OHDSI containers; ATLAS, HADES & ARES should open



### Step 3: Verify Running Containers

```bash
docker ps

```

## 3. Installing PostgreSQL on Docker

The PostgreSQL client allows you to interact with the database from your docker machine or a remote server


### Step 1: Install PostgreSQL Client
The install function varies depending on what is already installed in your WSL #To install PSQL;

```bash
sudo apt install postgresql-client
sudo -u postgres psql
```

### Step 2: Check Network and Assign Credentials

Ensure that the PostgreSQL container is accessible over the correct network and connect it to other containers as needed.

```bash
docker network ls

```

### Step 2: To run pgAdmin (a web-based database management tool) and connect it to PostgreSQL, use:


```bash
docker run -p 8765:80 -e PGADMIN_DEFAULT_EMAIL=<youremailaddress> -e PGADMIN_DEFAULT_PASSWORD=<yourpassword> --network broadsea_default -d dpage/pgadmin4

```
### Step 3: Finding the IP Address of the Atlas Database Container

After starting the necessary containers, retrieve the IP address of the ATLAS Database container (not pgAdmin4) by running:


```bash

docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' XXXXXXXX(Insert ATLAS DB containerID-atlas db ID not pgadmin4) --inspect to find atlas db ip address

```


## 4. Database Backup and Restoration in Docker

### Step 1: Dump Local Database

```bash
set PGPASSWORD=password
pg_dump -U postgres -d mydatabase -F c -f "C:\Users\Amadi\OneDrive\Desktop\HP\data\backup\mydatabase.backup"
```

### Step 2: Transfer Backup File to Docker Container

```bash
docker cp "C:\Users\Amadi\OneDrive\Desktop\HP\data\backup\mydatabase.backup" postgres:/tmp/
```

### Step 3: Restore Database in Docker

```bash
docker exec -it postgres psql -U postgres -d mydatabase -f /tmp/mydatabase.backup
```

### Step 4: Verify Data Import

```bash
docker exec -it postgres psql -U postgres -d mydatabase -c "\dt"
```

## 5. Moving Data from Windows to Docker and Uploading to an Existing Database

### Step 1: Dump Schema Only

```bash
pg_dump -U postgres -d mydatabase -F p -f "path\mydatabase_schema.sql" -n my_schema
```

### Step 2: Transfer Schema File to Docker

```bash
docker cp "path\mydatabase_schema.sql" postgres:/tmp/
```

### Step 3: Apply Schema in Docker Database

```bash
docker exec -it postgres psql -U postgres -d mydatabase -f /tmp/mydatabase_schema.sql
```

## 6. Configuring Broadsea WebAPI

### Step 1: Add New CDM Source

- Update the `source` table to register the CDM database.

### Step 2: Update Source Daimon Table

```bash
INSERT INTO source_daimon (source_id, daimon_type, table_qualifier) VALUES (1, 1, 'cdm_database');
```

### Step 3: Restart OHDSI Containers

```bash
docker-compose restart
```

## 7. Best Practices and Troubleshooting

- **Check Logs:** `docker logs <container_name>`
- **Verify Network:** `docker network inspect broadsea_default`
- **Check Running Containers:** `docker ps`
- **Database Connectivity:** Ensure correct database host (use `docker inspect` to find the IP of the container instead of `localhost`).

---

This guide ensures a clear step-by-step installation and configuration process for Docker, OHDSI Broadsea, PostgreSQL on Docker, and database migrations. Let me know if you'd like additional clarifications or improvements!


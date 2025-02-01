# Installation and Configuration Guide: Docker, OHDSI Broadsea, and PostgreSQL on Docker

## 1. Installing Docker on WSL2

### Step 1: Install Docker

Follow the official Docker installation guide for your system: [Docker Installation Guide](https://docs.docker.com/get-docker/)

### Step 2: Update Package List and Install Required Packages

```bash
sudo apt-get update
sudo apt-get install     apt-transport-https     ca-certificates     curl     gnupg     lsb-release
```

### Step 3: Add Docker’s Official GPG Key and Repository

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo   "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Step 4: Install Docker Engine

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### Step 5: Configure Docker Permissions

```bash
sudo usermod -aG docker $USER
```

Log out and log back in for changes to take effect.

### Step 6: Start and Verify Docker Service

```bash
sudo service docker start
sudo service docker status
```

## 2. Installing PostgreSQL on Docker

### Step 1: Install PostgreSQL Client

```bash
sudo apt install postgresql-client
```

### Step 2: Run PostgreSQL in Docker

```bash
docker run --name postgres -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 -d postgres
```

### Step 3: Access PostgreSQL Container

```bash
docker exec -it postgres psql -U postgres
```

### Step 4: Check Network and Assign Credentials

```bash
docker network ls
docker run -p 8765:80 -e PGADMIN_DEFAULT_EMAIL=<youremailaddress> -e PGADMIN_DEFAULT_PASSWORD=<yourpassword> --network broadsea_default -d dpage/pgadmin4
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <containerID>
```

## 3. Installing OHDSI Broadsea

### Step 1: Clone OHDSI Broadsea Repository

```bash
git clone https://github.com/OHDSI/Broadsea.git
cd Broadsea
```

### Step 2: Set Up Environment Variables

Edit `.env` file to update configuration parameters for WebAPI, ATLAS, and database connections.

### Step 3: Start OHDSI Broadsea Containers

```bash
docker-compose up -d
```

### Step 4: Verify Running Containers

```bash
docker ps
```

### Step 5: Configure WebAPI

- Navigate to `http://127.0.0.1/WebAPI/ddl/results?dialect=postgresql&schema=your_schema&vocabSchema=your_vocab_schema&tempSchema=tmp&initConceptHierarchy=true`
- Update `source_daimon` table to register the new data sources.

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


# Project Overview
This is a containerized three-tier ticketing system for a help-desk. Users open a browser and enter the application, and from there, can submit and view support tickets through an Apache-served web page. 

Apache passes API and health requests back to a Flask application, and Flask reads and writes ticket records to a MariaDB database. The stack is defined entirely in Docker Compose, so the whole application can be built with a single command.

# Prerequisites
The following must be installed on the VM before bringing the stack up:
 
- **Docker Engine** (29.4.0 or later)
- **Docker Compose**
- A shell with `curl` and `python3` available. These are used for the verification script.

You can follow the "Install using the `apt` repository" instructions [here](https://docs.docker.com/engine/install/ubuntu/).

After installing, confirm your versions with:
 
```bash
docker --version
docker compose version
```

You can also enter the following commands to run Docker commands without `sudo`:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

# Getting Started
Follow these steps for getting and setting up a fresh clone of the repository:
 
1. **Clone the repository and enter the project directory.**
 
   ```bash
   git clone https://github.com/anthonypham004/inet4031-testlab12.git
   cd inet4031-testlab12
   ```
 
2. **Create your `.env` file** by copying the provided `.env.example` template, then fill in your own credentials (see [Configuration](#configuration) below).
 
   ```bash
   cp .env.example .env
   ```

3. **Build and start** all three containers.
 
   ```bash
   docker compose up --build
   ```
 
   Docker will build the Flask app and Apache web images, pull the MariaDB image, initialize the database with data from `init.sql`, and start all services. The `app` container waits for `db` to pass its health check before starting, and `web` waits for `app` to pass its health check before starting. The full stack may take around 30 seconds before it's fully ready.

   If you get an error along the lines of
   ```bash
   failed to bind host port 0.0.0.0:80/tcp: address already in use
   ```
   you must've had Apache2 using port 80 already, perhaps from a previous lab. To fix this, disable Apache2 using
   ```bash
   sudo systemctl stop apache2
   ```
   and try building again.
 
4. **Check container status** from a separate terminal to confirm all three are running and healthy. The `db` and `app` containers should come up as `healthy`, and the `web` container should come up as `running`. If this isn't the case after a few minutes, something may have went wrong and will need troubleshooting.
 
   ```bash
   docker compose ps
   ```
 
5. **Open the application** with a browser at `http://<your-IP>:80`. You should see a green "API healthy" indicator and a ticket board populated with sample tickets. You can check the health statuses through the shell:

   ```bash
   curl http://localhost:80/
   ```
   Confirm you get an HTML response.

   ```bash
   curl http://localhost:80/health
   ```
   `{"database": "connected","status": "healthy"}` indicates Flask is healthy.

6. **Verify the system works** by adding your own tickets from the browser or through the shell:
   ```shell
   curl -X POST http://localhost:80/api/tickets \
   -H "Content-Type: application/json" \
   -d '{"title": "My first ticket", "description": "Testing the API"}'
   ```

   To stop and remove the containers while preserving the database volume, run:
   
   ```bash
   docker compose down
   ```
   
   To *also* delete the database volume and start completely fresh, run:
   
   ```bash
   docker compose down -v
   ```

# Configuration

Docker Compose reads credentials from a `.env` file in the project root. This file is listed in `.gitignore` and must *never* be committed to the repository since it contains passwords.
 
Copy the template to get started:
 
```bash
cp .env.example .env
```
 
The `.env` file contains the following variables:
 
- `DB_ROOT_PASSWORD`: The MariaDB root account password.
- `DB_NAME`: Name of the application database (default: `ticketdb`).
- `DB_USER`: MariaDB user account that Flask connects with (default: `flaskuser`).
- `DB_PASSWORD`: Password for `DB_USER`.
 
The template ships with placeholder values. Replace `DB_ROOT_PASSWORD` and `DB_PASSWORD` with your own passwords before running `docker compose up`.

# Verification
To verify the functionality of the application, use the included `check-lab.sh` script from the project root after the stack is up. First make sure that the script is executable, then run it:
 
```bash
chmod +x check-lab.sh
./check-lab.sh
```
 
The script runs nine checks, printing `[PASS]` or `[FAIL]` for each:
 
1. **Container health** — confirms `db` and `app` are `healthy` and `web` is running (3 checks total).
2. **Apache on port 80** — confirms Apache responds to an HTTP request on port 80.
3. **Flask health endpoint** — confirms `GET /health` returns `{"status": "healthy"}`, proving Flask can reach MariaDB.
4. **API functionality** — confirms `GET /api/tickets` returns a JSON array and `POST /api/tickets` successfully creates a new ticket (2 checks total).
5. **Named volume** — confirms the named volume exists for MariaDB.
6. **Named network** — confirms the `app-network` bridge network exists.
 
A fully passing run ends with:
 
```
=============================================
  Results: 9 passed, 0 failed
=============================================
All checks passed. Take your screenshots and submit.
```
 
If any check fails, the script prints a hint and the relevant `docker compose logs <service>` command to help you troubleshoot.


# 🏦 Bank Services — Docker Swarm + Jenkins CI/CD Pipeline

A production-style deployment of 4 banking microservices using **Docker Swarm**, **Jenkins CI/CD**, and **Nginx Reverse Proxy** — all automated from a single GitHub push.

---

## 🏗️ Architecture

```
GitHub Repo
    ↓  (Jenkins pulls code)
Jenkins Pipeline
    ↓  (docker build + push)
Docker Hub
    ↓  (docker stack deploy)
Docker Swarm Cluster (3 Nodes)
    ↓
Nginx Reverse Proxy (Port 80)
    ├── /internet/  → Internet Banking Service (3 replicas)
    ├── /mobile/    → Mobile Banking Service   (3 replicas)
    ├── /insurance/ → Insurance Service        (3 replicas)
    └── /loan/      → Loan Service             (3 replicas)
```

---

## 🚀 Tech Stack

| Tool | Purpose |
|---|---|
| **Docker** | Containerization |
| **Docker Swarm** | Container orchestration across 3 nodes |
| **Docker Stack** | Deploy multi-service apps on Swarm |
| **Jenkins** | CI/CD Pipeline automation |
| **Nginx** | Reverse proxy — single port for all services |
| **Apache2** | Web server inside each container |
| **Docker Hub** | Image registry |

---

## 📁 Project Structure

```
dockerproject/
├── Dockerfile           # Builds the internet banking image
├── index.html           # Internet banking frontend
├── docker-compose.yml   # Defines all 5 services (nginx + 4 banking)
├── JenkinsPipeline      # Jenkins declarative pipeline script
└── nginx.conf           # Nginx reverse proxy configuration
```

---

## ⚙️ Jenkins Pipeline Stages

```
checkout → build → tag → login & push → deploy
```

| Stage | What Happens |
|---|---|
| **checkout** | Clones this repo into Jenkins workspace |
| **build** | Builds Docker image from Dockerfile |
| **tag** | Tags image for Docker Hub |
| **login & push** | Authenticates and pushes image to Docker Hub |
| **deploy** | Deploys full stack to Docker Swarm cluster |

---

## 🐳 Services in the Stack

| Service | Image | Replicas | Path |
|---|---|---|---|
| nginx | nginx:latest | 1 | Port 80 (entry point) |
| internetbanking | shefatfaisal/ib-name | 3 | /internet/ |
| mobilebanking | shefatfaisal/mobilebanking-name | 3 | /mobile/ |
| insurance | shefatfaisal/insurance-name | 3 | /insurance/ |
| loan | shefatfaisal/loans-name | 3 | /loan/ |

> Total: **13 containers** spread across 3 hosts automatically by Docker Swarm

---

## 🛠️ Setup & Deployment

### 1. Create 3 EC2 Servers and Set Up Docker Swarm

```bash
# On all 3 servers
yum install docker -y
systemctl start docker

# On manager node only
docker swarm init

# On worker nodes (use token from manager)
docker swarm join --token <token> <manager-ip>:2377
```

### 2. Install Jenkins on Manager Node

```bash
yum install git java-1.8.0-openjdk maven -y
amazon-linux-extras install java-openjdk11 -y
yum install jenkins -y
systemctl start jenkins.service
```

### 3. Create Nginx Config in Docker Swarm

```bash
docker config create nginx_config ./nginx.conf
```

### 4. Give Docker Permissions to Jenkins

```bash
chmod 777 /var/run/docker.sock
```

### 5. Run Jenkins Pipeline

Add the contents of `JenkinsPipeline` as a Jenkins job and configure these environment variables:
- `image` — local image name
- `repo` — Docker Hub repo name
- `password` — Docker Hub password

### 6. Access the Services

```
http://<your-server-ip>/internet/   → Internet Banking
http://<your-server-ip>/mobile/     → Mobile Banking
http://<your-server-ip>/insurance/  → Insurance
http://<your-server-ip>/loan/       → Loan
```

---

## 🔄 How Docker Swarm Routing Works

Thanks to Docker Swarm's **Routing Mesh**, hitting **any** of the 3 node IPs on port 80 will serve the app — even if the container is on a different node.

```
User → Any Node IP:80 → Docker Routing Mesh → Correct Container
```

---

## 📦 Key Docker Commands

```bash
# Deploy the stack
docker stack deploy -c docker-compose.yml bank

# List all stacks
docker stack ls

# List services in stack
docker stack services bank

# List containers in stack
docker stack ps bank

# Remove the stack
docker stack rm bank
```

---

## 👤 Author

**Shefat Faisal**
- GitHub: [@shefatfaisal](https://github.com/shefatfaisal)
- Docker Hub: [shefatfaisal](https://hub.docker.com/u/shefatfaisal)

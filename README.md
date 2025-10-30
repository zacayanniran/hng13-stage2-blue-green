HNG DevOps Stage 2: Blue/Green Failover Infrastructure


<div align="center">
HNG DevOps Stage 2
Blue/Green Deployment · Container Failover · Health Checks

</div>


🚀 Objective

 Configure a Blue/Green deployment using Docker Compose and NGINX such that:
1. Blue and Green app containers are both running
2. NGINX routes traffic to the active pool
3. On failure, traffic automatically moves to the backup
4. /version exposes metadata via HTTP headers
5. /healthz is used for health checks

This repository implements all required capabilities.

🏗️ Architecture
                    ┌────────────────────────┐
                    │        Client           │
                    │ curl / browser / grader │
                    └────────────┬────────────┘
                                 │ :8080
                          ┌──────▼───────┐
                          │   NGINX      │
                          │ Reverse Proxy│
                          └─────┬────────┘
                ┌───────────────┼────────────────┐
                │               │                │
      Active →  │      Blue App │   Green App    │  ← Backup
                │    :8081      │    :8082       │
                └───────────────┴────────────────┘


Failover occurs if the active app:
1. crashes
2. times out
3. returns 5xx

⚙️ Tech Stack
1. Docker & Docker Compose
2. NGINX reverse proxy
3. Health checks + failover logic
4. Google Cloud Compute Engine (VM deployment)
5. Environment-driven configuration


🚢 Setup Instructions
1. Clone repo
git clone https://github.com/zacayanniran/hng13-stage2-blue-green.git
cd hng-devops-stage2

2. Make scripts executable
chmod +x nginx/start.sh
chmod +x test/failover-test.sh

📦 Environment Variables
Provided in .env:

BLUE_IMAGE=zacchaeusayanniran/nginx-blue:v1
GREEN_IMAGE=zacchaeusayanniran/nginx-green:v1
ACTIVE_POOL=blue
RELEASE_ID_BLUE=release-v1.0-blue
RELEASE_ID_GREEN=release-v1.0-green
PORT=3000
NGINX_HOST_PORT=8080
BLUE_HOST_PORT=8081
GREEN_HOST_PORT=8082


Copy example:
    cp .env.example .env

▶️ Run Locally
docker compose up -d


View logs:
    docker compose logs -f


Stop:
    docker compose down -v

🌐 Endpoints
Endpoint	Description
    1. /version	Returns app pool & release ID
    2. /healthz	Up/down health
    3. /	Demo root route

🧪 Verify Blue is Active
curl -i http://localhost:8080/version


Example output:
    X-App-Pool: blue
    X-Release-Id: blue-v1

🔁 Failover Demonstration
1. Stop Blue
    docker stop hng-devops-stage2-app_blue-1

2. Request again
    curl -i http://localhost:8080/version


Expected:
    X-App-Pool: green
    X-Release-Id: green-v1

✅ Traffic automatically moved to Green.


🔄 Switching Traffic Manually
    Edit .env:
    ACTIVE_POOL=green


Recreate proxy:
    docker compose down
    docker compose up -d


🧰 Health Check Test
    curl -i http://localhost:8080/healthz


🧬 Failover CI (GitHub Actions)
    This repository contains:
        1. .github/workflows/stage2.yml

☁️ Cloud Deployment

The entire infrastructure is deployed on Google Cloud Compute Engine.
The setup includes:

Ubuntu VM with Docker & Docker Compose installed

Blue and Green app containers running simultaneously

NGINX reverse proxy managing failover

Environment variables configured via .env

🌐 Live URL:
👉 http://34.58.153.167:8080/


Google Cloud Deployment Steps

Create a VM (e.g. e2-medium, Ubuntu 22.04 LTS)

SSH into VM:

gcloud compute ssh <instance-name> --zone <your-zone>


Install Docker & Docker Compose

Clone this repo (git clone https://github.com/zacayanniran/hng13-stage2-blue-green.git)
and run:
 ./deploy (To deploy using the script)

Allow firewall for port 8080(nginx), 8081(blue-app) & 8082(green-app):

gcloud compute firewall-rules create allow-http-8080 --allow tcp:8080, 8081, 8082


Access via:
http://<EXTERNAL_IP>:8080


👀 Visual Deployment Diagram
graph TD
    A[Deploy New Green Version] --> C{Healthy?}
    C -->|Yes| D[NGINX switches to Green]
    C -->|No| E[Rollback to Blue]
    D --> F[Old Blue pool destroyed]


🧾 Sample JSON Response
{
  "status": "OK",
  "message": "Application version in header"
}



🔍 Example curl Suite
View version repeatedly
for i in {1..6}; do curl -s -I localhost:8080/version | grep X-App-Pool; sleep 1; done


Verify Release IDs
curl -I localhost:8080/version | grep X-Release-Id


Check root
curl localhost:8080

🚑 Troubleshooting
NGINX restarting?


Check config:
    1. docker compose logs nginx

Port already in use?

Stop conflicting process:
lsof -i :8080


📁 Project Structure
.
├── docker-compose.yml
├── nginx/
│   ├── nginx.conf.template
│   └── start.sh
├── test/
│   └── failover-test.sh
├── deploy.sh
├── .env.example
└── README.md


📣 Contribution Notes
    Pull requests welcome.


✅ Completion Criteria (Met)
1. Both pools run simultaneously
2. /version exposes metadata via headers
3. Failover within same request
4. /healthz health endpoint
5. Failover CI test
6. Docker Compose orchestrated


⭐ Final Thoughts

This implementation ensures:
1. Zero downtime deployment strategy
2. Instant failover capabilities
3. Production-grade proxy configuration


🙌 Author

Zacchaeus Ayanniran
DevOps/Cloud Engineer
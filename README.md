# Employee Management System — DevOps CI/CD

This project started as an Employee Management System built by [Shivank](https://github.com/ShivankK26) using the MERN stack. I forked it and used it as the base for building a full DevOps workflow around it - Docker, Jenkins, and Ansible, all running on AWS EC2.

I built this twice. The 1st version ran locally on Windows with a basic Jenkins pipeline. This version is a full rebuild on EC2, done properly this time, with real CD through Ansible instead of manually deploying by hand.

## Tech Stack

**Application:** React, Vite, Node.js, Express.js, MongoDB, Mongoose
**DevOps:** Git, Docker, Docker Compose, Jenkins, Ansible, Nginx

## Architecture

The setup runs across two separate EC2 instances, which mirrors how CI/CD is usually kept separate from the servers actually running the application:

- **App Server** - runs the application itself: MongoDB, backend, and frontend, all through Docker Compose.
- **CI/CD Server** - runs Jenkins and Ansible. Jenkins builds and pushes the Docker images; Ansible connects over SSH to deploy them to the App Server.

```
GitHub push → Jenkins builds & pushes images → Docker Hub → Ansible deploys → App Server → app live
```

## CI/CD Pipeline

Jenkins runs this pipeline on every push to `main`, triggered automatically via Poll SCM:

```
Checkout
   ↓
Build Images
   ↓
Docker Hub Login
   ↓
Push Images
   ↓
Deploy (Ansible)
```

- **Checkout** - pulls the latest code from GitHub.
- **Build Images** - builds the frontend and backend images with `docker compose build`.
- **Docker Hub Login** - logs in using a Docker Hub access token stored in Jenkins Credentials, never written into the Jenkinsfile or committed to Git.
- **Push Images** - tags and pushes both images to Docker Hub using the current Jenkins build number, not just `latest`, so every deployed version can be traced back to the exact build that produced it.
- **Deploy** - runs `ansible/deploy.yml` against the App Server. Ansible fills in the current build's image tag inside `ansible/templates/docker-compose.yml.j2`, writes the finished file to the server, pulls the new images, and restarts the containers  but only if something actually changed.

## Docker Setup

The frontend and backend each have a multi-stage, Alpine-based Dockerfile (`node:20-alpine`, using `npm ci` for consistent installs). The frontend is built with Vite and served through Nginx; the backend runs as a plain Node/Express container.

MongoDB uses a named volume so data survives restarts, and a healthcheck make sure the backend only starts once MongoDB is actually ready,not just once the container has started.

To run the application locally:

```bash
docker compose up --build
```

To stop it:

```bash
docker compose down
```

The frontend is available at `http://localhost:5173`.

## Ports

| Component | Port | Notes |
|---|---|---|
| Frontend | `5173` | Public — the original port from Shivank's setup |
| Backend | `8000` | Internal only, reachable only through Nginx |
| MongoDB | `27017` | Internal only |
| Jenkins UI | `8080` | Runs on the separate CI/CD server, so no conflict with the frontend |

(While rebuilding on EC2 I briefly moved the frontend to port 8080 to avoid a conflict with Jenkins, then moved it back to 5173 once Jenkins was on its own server.)

## Deployment with Ansible

Rather than writing raw shell commands to redeploy, deployment is handled through a proper Ansible playbook:

- `community.docker.docker_image` pulls the exact image and tag Jenkins just pushed.
- A Jinja2 template renders the final `docker-compose.yml` with the correct image tag filled in.
- A handler restarts the stack only if that file actually changed, so an unrelated pipeline run doesn't cause unnecessary downtime.

Jenkins passes its own build number straight through to Ansible, so whatever Jenkins just built is exactly what gets deployed.

**Ansible layout on the CI/CD server:**

- `/etc/ansible/hosts`  the inventory, listing the App Server's address and which SSH key to use.
- `/opt/ansible/ansible-key.pem`  the private SSH key used to connect to the App Server. Owned by the `jenkins` user, permissions locked to `400`. This file never goes into Git.
- Everything else (`deploy.yml`, `ansible.cfg`, `templates/`, `vars/`) lives in this repository, under `ansible/`, and is pulled fresh by Jenkins on every run.

### Rolling back to a previous build

Every image is tagged with its Jenkins build number, so rolling back just means re-running the playbook with an older tag:

```bash
sudo -u jenkins ansible-playbook -i /etc/ansible/hosts ansible/deploy.yml --extra-vars "image_tag=11"
```

## Running This Yourself

If you want to set this up on your own EC2 instances:

1. Launch two Ubuntu EC2 instances — one for the app, one for Jenkins.
2. On the App Server: install Docker and Docker Compose.
3. On the CI/CD Server: install Docker, Java, Jenkins, and Ansible.
4. Generate an SSH key pair, and make sure the CI/CD server can SSH into the App Server with it.
5. On the CI/CD server, set `/etc/ansible/hosts` to point at the App Server's IP and the SSH key's path.
6. In Jenkins: add your Docker Hub credentials, and create a Pipeline job pointing at this repository, with Poll SCM enabled.
7. Push to `main` - Jenkins will build, push, and deploy automatically from there.

## Screenshots

- Jenkins dashboard and build history - `screenshots/jenkins-dashboard.png`
- A full pipeline run, all stages green - `screenshots/pipeline-success.png`
- Docker Hub showing both images with build-numbered tags  `screenshots/dockerhub-tags.png`
- Containers running on the App Server - `screenshots/instance-a-containers.png`
- The app running in the browser - `screenshots/app-running.png`

## Troubleshooting

The real problems I ran into while building this, and how I fixed them, are documented in [`DECISIONS.md`](DECISIONS.md)  covering both the original application bugs and the EC2/Jenkins/Ansible issues from the rebuild.

## Original Application

This project is a fork of [Shivank's Employee Management System](https://github.com/ShivankK26/Employee-Management-System). The application itself  the React frontend, Express backend, and MongoDB models is his work. Everything around containerizing, automating, and deploying it is mine.

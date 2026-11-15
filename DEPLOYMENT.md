# CI/CD Deployment Guide

This project uses Jenkins to build a Docker image, push it to Docker Hub, and deploy it automatically to an Amazon EC2 instance.

## Deployment flow

```text
Merge into main
        ↓
GitHub webhook
        ↓
Jenkins checkout
        ↓
Docker build
        ↓
Push to Docker Hub
        ↓
SSH to deployment EC2
        ↓
Pull and restart the Docker container
```

## Application configuration

- Docker Hub image: `mj2808/nodedemo01:latest`
- Container name: `nodedemo01`
- Application port inside the container: `3000`
- EC2 HTTP port: `80`
- Port mapping: `80:3000`
- Health endpoint: `/health`

The application is started by the `start` script in `package.json`:

```json
"start": "node server.js"
```

## Jenkins configuration

### Required tools

Configure these under **Manage Jenkins → Tools**:

```text
NodeJS installation name: node24
NodeJS version: 24.19.0
```

The Jenkins server also needs Git and Docker installed. The Jenkins user must be able to run Docker:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo -u jenkins -H docker ps
```

### Required credentials

Configure these credentials in Jenkins:

| Credential ID | Kind | Use |
|---|---|---|
| Repository credential | Username with password | Private GitHub repository checkout |
| `docker` | Username with password | Docker Hub login and push |
| `nodemoec2` | SSH Username with private key | SSH deployment to EC2 |

For `nodemoec2`:

```text
Username: ec2-user
Private key: complete OpenSSH private key
```

The private key must include the complete header and footer. Never commit it to GitHub or store it in this repository.

## Deployment EC2 configuration

The deployment EC2 must have Docker installed and running:

```bash
sudo dnf install -y docker
sudo systemctl enable --now docker
sudo usermod -aG docker ec2-user
```

Reconnect after adding `ec2-user` to the Docker group, then verify:

```bash
docker --version
docker ps
```

The deployment EC2 Security Group should allow:

```text
SSH TCP 22: Jenkins EC2 Security Group only
HTTP TCP 80: 0.0.0.0/0
```

Jenkins and the deployment EC2 are in the same VPC, so Jenkins deploys using the deployment EC2 private IP configured in the Jenkinsfile.

## Manual verification on the deployment EC2

The Jenkins deployment stage performs these operations automatically:

```bash
docker pull mj2808/nodedemo01:latest
docker rm -f nodedemo01 || true
docker run -d \
  --name nodedemo01 \
  --restart unless-stopped \
  -p 80:3000 \
  mj2808/nodedemo01:latest
```

Verify the container and health endpoint:

```bash
docker ps
curl http://localhost/health
```

From a browser, use the deployment EC2 public IP:

```text
http://YOUR-EC2-PUBLIC-IP/
http://YOUR-EC2-PUBLIC-IP/health
```

Because the container maps host port `80` to container port `3000`, do not add `:3000` to the browser URL.

## GitHub webhook

Configure the GitHub repository webhook as:

```text
http://YOUR-JENKINS-URL/github-webhook/
```

Enable JSON payloads and Push events. A merge into `main` creates a push event and starts Jenkins.

## Common checks

Check Docker on Jenkins:

```bash
sudo -u jenkins -H docker ps
```

Check SSH connectivity from Jenkins to the deployment EC2:

```bash
nc -vz DEPLOYMENT-EC2-PRIVATE-IP 22
```

Check the deployed container:

```bash
docker ps
docker logs nodedemo01
```

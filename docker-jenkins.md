# Docker Jenkins Management Cheat Sheet

---

## 1. System Prerequisites

Check Docker version:
```bash
docker --version
docker info
```

Check Docker daemon status:
```bash
systemctl status docker
```

Start Docker daemon (if not running):
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Check available disk space:
```bash
df -h
docker system df
```

---

## 2. Docker Network Setup

Create a dedicated network for Jenkins and DinD:
```bash
docker network create jenkins
```

List all Docker networks:
```bash
docker network ls
```

Inspect the Jenkins network:
```bash
docker network inspect jenkins
```

---

## 3. Docker-in-Docker (DinD) Setup

Run the Docker-in-Docker container (required for Jenkins to run Docker commands):
```bash
docker run -d \
  --name jenkins-docker \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  -v jenkins-docker-certs:/certs/client \
  -v jenkins_data:/var/jenkins_home \
  -p 2376:2376 \
  docker:dind \
  --storage-driver overlay2
```

---

## 4. Start Jenkins (Production-Ready)

Run Jenkins with persistent volume, DinD support, resource limits, and auto-restart:
```bash
docker run -d \
  --name jenkins \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_data:/var/jenkins_home \
  -v jenkins-docker-certs:/certs/client:ro \
  --restart=on-failure \
  --memory=2g \
  --cpus=2 \
  jenkins/jenkins:lts
```

Start an existing Jenkins container:
```bash
docker start jenkins
```

> **Restart policy options:**  
> `no` | `on-failure` | `always` | `unless-stopped`  
> Use `--restart=unless-stopped` for production to survive host reboots.

---

## 5. Stop / Restart Jenkins

Stop Jenkins gracefully:
```bash
docker stop jenkins
```

Restart Jenkins:
```bash
docker restart jenkins
```

Stop with a custom timeout (seconds before force kill):
```bash
docker stop --time=30 jenkins
```

---

## 6. Retrieve Jenkins Initial Admin Password

Get the initial admin password from the container:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Or from the volume directly:
```bash
sudo cat /var/lib/docker/volumes/jenkins_data/_data/secrets/initialAdminPassword
```

---

## 7. Inspect & Monitor Jenkins

List running containers:
```bash
docker ps
```

List all containers (including stopped):
```bash
docker ps -a
```

Inspect full container details:
```bash
docker inspect jenkins
```

Check container health status:
```bash
docker inspect --format='{{.State.Health.Status}}' jenkins
```

View last N log lines:
```bash
docker logs --tail=100 jenkins
```

Follow logs live:
```bash
docker logs -f jenkins
```

Monitor real-time CPU, memory, and network usage:
```bash
docker stats jenkins
```

Monitor all containers:
```bash
docker stats
```

Watch Docker events in real time:
```bash
docker events --filter name=jenkins
```

---

## 8. Manage Jenkins Container

Open a shell inside Jenkins:
```bash
docker exec -it jenkins bash
```

Run a one-off command inside Jenkins without shell:
```bash
docker exec jenkins cat /var/jenkins_home/config.xml
```

Check Jenkins process inside the container:
```bash
docker exec jenkins ps aux
```

Copy Jenkins data from container to host:
```bash
docker cp jenkins:/var/jenkins_home ./jenkins_backup
```

Copy files from host into Jenkins container:
```bash
docker cp ./plugins jenkins:/var/jenkins_home/plugins
```

---

## 9. Image Management

Pull the latest Jenkins LTS image:
```bash
docker pull jenkins/jenkins:lts
```

Pull a specific Jenkins version:
```bash
docker pull jenkins/jenkins:2.452.3-lts
```

List all local Docker images:
```bash
docker images
```

Build a custom Jenkins image from a Dockerfile:
```bash
docker build -t my-jenkins:1.0 .
```

Tag an image for a registry:
```bash
docker tag my-jenkins:1.0 myregistry.example.com/my-jenkins:1.0
```

Remove a local image:
```bash
docker rmi jenkins/jenkins:lts
```

Remove all dangling (unused) images:
```bash
docker image prune -f
```

---

## 10. Docker Registry (Push / Pull)

Login to Docker Hub:
```bash
docker login
```

Login to a private registry:
```bash
docker login myregistry.example.com
```

Push a custom image to a registry:
```bash
docker push myregistry.example.com/my-jenkins:1.0
```

Pull from a private registry:
```bash
docker pull myregistry.example.com/my-jenkins:1.0
```

Logout from registry:
```bash
docker logout
```

---

## 11. Docker Volumes for Jenkins

Create a named volume:
```bash
docker volume create jenkins_data
```

Create the DinD certs volume:
```bash
docker volume create jenkins-docker-certs
```

List all volumes:
```bash
docker volume ls
```

Inspect the Jenkins volume:
```bash
docker volume inspect jenkins_data
```

Run Jenkins with volumes mounted:
```bash
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_data:/var/jenkins_home \
  --restart=unless-stopped \
  jenkins/jenkins:lts
```

---

## 12. Backup Jenkins Data

Backup the Jenkins volume to a tar file:
```bash
docker run --rm \
  -v jenkins_data:/data \
  -v $(pwd):/backup \
  alpine tar cvf /backup/jenkins_backup.tar /data
```

Backup with timestamp:
```bash
docker run --rm \
  -v jenkins_data:/data \
  -v $(pwd):/backup \
  alpine tar cvf /backup/jenkins_backup_$(date +%Y%m%d_%H%M%S).tar /data
```

---

## 13. Restore Jenkins Data

Restore the Jenkins volume from a tar file:
```bash
docker run --rm \
  -v jenkins_data:/data \
  -v $(pwd):/backup \
  alpine tar xvf /backup/jenkins_backup.tar -C /
```

> Stop the Jenkins container before restoring to avoid data corruption:
> ```bash
> docker stop jenkins
> # run restore command
> docker start jenkins
> ```

---

## 14. Remove Jenkins

Remove a stopped Jenkins container:
```bash
docker rm jenkins
```

Force remove a running Jenkins container:
```bash
docker rm -f jenkins
```

Remove the Jenkins image:
```bash
docker rmi jenkins/jenkins:lts
```

Remove the Jenkins container and its volumes together:
```bash
docker rm -v jenkins
```

---

## 15. Volume Management

Remove the Jenkins data volume (caution: deletes all data):
```bash
docker volume rm jenkins_data
```

Remove unused volumes:
```bash
docker volume prune -f
```

---

## 16. Update Jenkins to a New Version

Pull the new image:
```bash
docker pull jenkins/jenkins:lts
```

Stop and remove the old container (data is safe in the volume):
```bash
docker stop jenkins
docker rm jenkins
```

Run the new container with the same volume:
```bash
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_data:/var/jenkins_home \
  --restart=unless-stopped \
  jenkins/jenkins:lts
```

---

## 17. Docker Compose for Jenkins Stack

Create a `docker-compose.yml`:
```yaml
version: '3.8'

services:
  jenkins-docker:
    image: docker:dind
    container_name: jenkins-docker
    privileged: true
    networks:
      jenkins:
        aliases:
          - docker
    environment:
      - DOCKER_TLS_CERTDIR=/certs
    volumes:
      - jenkins-docker-certs:/certs/client
      - jenkins_data:/var/jenkins_home
    ports:
      - "2376:2376"
    restart: unless-stopped

  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    networks:
      - jenkins
    environment:
      - DOCKER_HOST=tcp://docker:2376
      - DOCKER_CERT_PATH=/certs/client
      - DOCKER_TLS_VERIFY=1
    volumes:
      - jenkins_data:/var/jenkins_home
      - jenkins-docker-certs:/certs/client:ro
    ports:
      - "8080:8080"
      - "50000:50000"
    restart: unless-stopped
    depends_on:
      - jenkins-docker

networks:
  jenkins:

volumes:
  jenkins_data:
  jenkins-docker-certs:
```

Start the full Jenkins stack:
```bash
docker compose up -d
```

Stop the stack:
```bash
docker compose down
```

Stop and remove volumes:
```bash
docker compose down -v
```

View stack logs:
```bash
docker compose logs -f
```

---

## 18. System Cleanup (Production Housekeeping)

Remove all stopped containers:
```bash
docker container prune -f
```

Remove all unused images:
```bash
docker image prune -a -f
```

Remove all unused volumes:
```bash
docker volume prune -f
```

Remove all unused networks:
```bash
docker network prune -f
```

Full system cleanup (containers, images, volumes, networks — use with caution):
```bash
docker system prune -a --volumes -f
```

Check Docker disk usage:
```bash
docker system df
```

---

## 19. Jenkins CLI (Advanced)

Download Jenkins CLI jar from running instance:
```bash
curl -O http://localhost:8080/jnlpJars/jenkins-cli.jar
```

List all Jenkins jobs:
```bash
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth admin:<password> list-jobs
```

Trigger a Jenkins job:
```bash
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth admin:<password> build <job-name>
```

Restart Jenkins via CLI:
```bash
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth admin:<password> restart
```

Safe restart Jenkins (waits for running jobs):
```bash
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth admin:<password> safe-restart
```

Install a plugin:
```bash
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth admin:<password> install-plugin <plugin-name>
```

---

## 20. Quick Reference Summary

| Task                          | Command                                              |
|-------------------------------|------------------------------------------------------|
| Start Jenkins                 | `docker start jenkins`                               |
| Stop Jenkins                  | `docker stop jenkins`                                |
| Restart Jenkins               | `docker restart jenkins`                             |
| View logs                     | `docker logs -f jenkins`                             |
| Open shell                    | `docker exec -it jenkins bash`                       |
| Get admin password            | `docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword` |
| Check health                  | `docker inspect --format='{{.State.Health.Status}}' jenkins` |
| Monitor resources             | `docker stats jenkins`                               |
| Backup volume                 | `docker run --rm -v jenkins_data:/data -v $(pwd):/backup alpine tar cvf /backup/jenkins_backup.tar /data` |
| Restore volume                | `docker run --rm -v jenkins_data:/data -v $(pwd):/backup alpine tar xvf /backup/jenkins_backup.tar -C /` |
| Update Jenkins                | `docker pull jenkins/jenkins:lts` → `docker rm jenkins` → re-run |
| Full cleanup                  | `docker system prune -a --volumes -f`                |

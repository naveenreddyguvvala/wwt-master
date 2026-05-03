# Docker Setup and Application Deployment Guide

This Readme shows the steps followed to set up the environment, clone the project, build the Docker image, and resolve an issue encountered during the build process.

---

## 1. Installing Docker and Git on the server

First, the system packages were updated and required tools were installed.

```bash
# Update the system
sudo dnf update -y

# Install Docker and Git
sudo dnf install -y docker git
```

After installation, Docker service was started and enabled to run automatically on system boot.

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

---

## 2. Fixing Docker Permissions

By default, Docker requires root privileges. To avoid using `sudo` for every command, the user was added to the Docker group.

```bash
# Add your user to the docker group
sudo usermod -a -G docker ec2-user
```

To apply the changes immediately without logging out:

```bash
newgrp docker
```

---

## 3. Project Setup (Git)

The project repository was cloned and accessed locally.

```bash
# Clone the repository
git clone [https://github.com](https://github.com/naveenreddyguvvala/wwt-master.git)

# Navigate into the project folder
cd wwt-master/wwwt-master
```

---

## 4. Building and Running Docker

The Docker image was built using the following command:

```bash
docker build -t my-app-image .
```

### Issue Faced During Build

During the build process, it failed at the package installation step:

```bash
RUN apt-get install python3 python3-pip
```

Error observed:

```
Do you want to continue? [Y/n] Abort.
```
<img width="1062" height="200" alt="image" src="https://github.com/user-attachments/assets/ce26fb61-53a5-41cf-a82b-2cf8b3df55c9" />

---

## 5. Understanding the Issue

The failure occurred because:

* The `apt-get install` command expects user confirmation (`Y/n`)
* Docker build runs in a non-interactive mode
* Since no input can be provided, the process automatically aborts

---

## 6. Root Cause

* The install command requires confirmation
* The `-y` flag was not used
* Docker cannot handle interactive prompts during build

---

## 7. Fix Applied

The Dockerfile was updated to include the `-y` flag:

```dockerfile
RUN apt-get update
RUN apt-get install -y python3 python3-pip
```

This ensures automatic confirmation during installation.
<img width="712" height="280" alt="image" src="https://github.com/user-attachments/assets/2c82bc34-127a-41a2-ab4d-b33f0617edce" />


---
### Issue : Python Package Installation Error

Error:

```
error: externally-managed-environment
```

#### Root Cause

* Ubuntu latest uses Python 3.12
* System Python blocks global pip installs

#### Fix

```dockerfile
RUN pip install --break-system-packages -r requirements.txt
```
<img width="650" height="250" alt="Added break package" src="https://github.com/user-attachments/assets/2887a874-d37d-4f3c-8ffa-96804f3086a5" />

---

### Issue : Container Failed to Start

Error:

```
exec: "python": executable file not found in $PATH
```

#### Root Cause

* Ubuntu provides `python3`, not `python`

#### Fix

```dockerfile
ENTRYPOINT ["python3", "app.py"]
```


---

## 8. Final Dockerfile

<img width="723" height="300" alt="addedentrypoint 5" src="https://github.com/user-attachments/assets/e1de03c7-ea27-4f75-9508-aa744358ecc8" />
```

<img width="1555" height="318" alt="build runsuccess 3" src="https://github.com/user-attachments/assets/bb9b5433-fdf8-48b0-a9ff-ad581882abaf" />

---

## 9. Optimization (Recommended Practice)

To improve efficiency and reduce image size, the commands were combined:

```dockerfile
RUN apt-get update && apt-get install -y python3 python3-pip
```
## 10. Build Docker Image

```bash
docker build -t my-app-image .
```

---

## 11. Run Container

```bash
docker run -d -p 5000:5000 --name my-web-app -e APP_PORT=5000 my-app-image
```

### Why `-e APP_PORT=5000` is used

* `-e` is used to pass environment variables into the container
* `APP_PORT=5000` tells the application which port to run on
* This avoids hardcoding the port in code

Example inside the application:

```python
import os
port = int(os.getenv("APP_PORT", 5000))
app.run(host="0.0.0.0", port=port)
```
<img width="1153" height="497" alt="buildsucces and appisrunning" src="https://github.com/user-attachments/assets/8198b644-5e57-49f1-93ce-87c86543a670" />

---

## 12. Access Application

```
http://<EC2-Public-IP>:5000
```
<img width="1121" height="287" alt="output" src="https://github.com/user-attachments/assets/c322c899-d277-4493-938e-8726a9b58f2d" />
---


## 14. Debugging Commands

Check running containers:

```bash
docker ps
```

Check logs:

```bash
docker logs my-web-app
```

Test locally:

```bash
curl http://localhost:5000
```


---

## 15. Key Learnings

* Always use `-y` with `apt-get install` in Docker
* Latest Ubuntu restricts global pip installs
* Use `--break-system-packages` or virtual environments
* Use `python3` instead of `python` in Ubuntu
* Container running does not mean all routes are available
* Always verify application endpoints

---

---

## 17. Conclusion

The deployment is successful from Docker and infrastructure perspective.

# 🔐 Enterprise Java Auth Portal & Automated AWS EC2 Deployment Pipeline

[![Java CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?logo=github-actions&logoColor=white)](.github/workflows/java-ci.yml)
[![AWS EC2](https://img.shields.io/badge/Deploy-AWS%20EC2-FF9900?logo=amazon-ec2&logoColor=white)](https://aws.amazon.com/ec2/)
[![Java](https://img.shields.io/badge/Java-17-ED8B00?logo=openjdk&logoColor=white)](https://adoptium.net/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.3.5-6DB33F?logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Spring Security](https://img.shields.io/badge/Spring%20Security-6.x-6DB33F?logo=springsecurity&logoColor=white)](https://spring.io/projects/spring-security)

A production-ready **Java Spring Boot 3 & Spring Security** web application featuring secure form-based authentication, role-based access control (RBAC), and automated zero-touch CI/CD deployment to **AWS EC2** using **GitHub Actions**.

---

## 📸 Project Preview & UI Showcase

| Login Screen | Authenticated User Dashboard |
| :--- | :--- |
| ![Login Screen](docs/screenshots/login-preview.png) | ![Dashboard Screen](docs/screenshots/dashboard-preview.png) |
| *Modern Tailwind-styled sign-in interface* | *Protected role-based user dashboard* |

> *Note: Place your screenshot images inside `docs/screenshots/` and update the image path references above for your repository.*

---

## 📋 Table of Contents

1. [Project Overview](#-project-overview)
2. [Tech Stack & Architecture](#-tech-stack--architecture)
3. [Project Directory Structure](#-project-directory-structure)
4. [Step-by-Step Guide: Going Live on AWS EC2](#-step-by-step-guide-going-live-on-aws-ec2)
   - [Phase 1: AWS EC2 Instance Provisioning](#phase-1-aws-ec2-instance-provisioning)
   - [Phase 2: EC2 Server Initialization & Systemd Setup](#phase-2-ec2-server-initialization--systemd-setup)
   - [Phase 3: GitHub Actions Secrets Setup](#phase-3-github-actions-secrets-setup)
   - [Phase 4: Running the Pipeline & Automated Deployment](#phase-4-running-the-pipeline--automated-deployment)
   - [Phase 5: Verification & Health Check](#phase-5-verification--health-check)
5. [Application Endpoints & Security Matrix](#-application-endpoints--security-matrix)
6. [Demo User Credentials](#-demo-user-credentials)
7. [Automated Testing](#-automated-testing)
8. [Server Management & Troubleshooting](#-server-management--troubleshooting)

---

## 🌟 Project Overview

This repository provides a complete template and production blueprint for building, testing, and shipping Java Spring Boot web applications:

- **BCrypt Encrypted Credentials:** Passwords are encrypted using modern BCrypt hashing.
- **Session-Based Form Security:** Protection against CSRF attacks and unauthenticated route access.
- **Automated Health Probes:** Public `/health` endpoint for continuous health checking and deployment smoke tests.
- **Zero-Downtime System Service:** Managed via Linux `systemd` daemon on Ubuntu EC2.
- **Continuous Integration & Delivery:** Fully automated workflow running JUnit 5 tests, packaging executable `.jar` artifacts, and transferring builds via secure SCP/SSH to AWS EC2.

---

## 🛠 Tech Stack & Architecture

```text
[ Developer Push ] ──► [ GitHub Repository (main) ]
                              │
                              ▼
                   [ GitHub Actions Runner ]
                   ├── 1. Setup JDK 17 & Maven Cache
                   ├── 2. Run JUnit 5 Security Tests
                   ├── 3. Package Spring Boot app.jar
                   └── 4. SCP Build to AWS EC2 via SSH
                              │
                              ▼
                      [ AWS EC2 Instance ]
                      ├── Linux systemd Service Daemon
                      └── Live Web App on Port 8080
```

- **Backend:** Java 17, Spring Boot 3.3.5, Spring Security 6
- **Frontend:** Thymeleaf, HTML5, Tailwind CSS
- **Build & Package:** Apache Maven
- **Testing:** JUnit 5, Spring Boot Test, Spring Security Test, MockMvc
- **Cloud Infrastructure:** AWS EC2 (Ubuntu 22.04 LTS)
- **Deployment Automation:** GitHub Actions (`appleboy/scp-action`, `appleboy/ssh-action`)

---

## 📂 Project Directory Structure

```text
java-login-app/
├── .github/
│   └── workflows/
│       └── java-ci.yml                 # Automated CI/CD pipeline configuration
├── docs/
│   └── screenshots/                    # Project UI & architecture screenshots
├── src/
│   ├── main/
│   │   ├── java/com/devmart/loginapp/
│   │   │   ├── LoginAppApplication.java # Spring Boot entry point
│   │   │   ├── config/
│   │   │   │   └── SecurityConfig.java  # Security filter chain & BCrypt setup
│   │   │   └── controller/
│   │   │       └── AuthController.java  # Request controllers & health endpoint
│   │   └── resources/
│   │       ├── application.properties   # App server port & configuration
│   │       └── templates/
│   │           ├── login.html           # Login screen template
│   │           └── dashboard.html       # Protected user dashboard
│   └── test/
│       └── java/com/devmart/loginapp/
│           └── LoginAppApplicationTests.java # Automated endpoint unit tests
├── pom.xml                              # Maven build dependencies & plugins
└── README.md                            # Complete developer documentation
```

---

## 🚀 Step-by-Step Guide: Going Live on AWS EC2

Follow these 5 sequential phases to take your project from GitHub to a live AWS EC2 instance.

### Phase 1: AWS EC2 Instance Provisioning

1. Log in to your **AWS Management Console** and navigate to **EC2 Dashboard**.
2. Click **Launch Instance** and select:
   - **Name:** `java-app-server`
   - **OS Image (AMI):** Ubuntu Server 22.04 LTS or 24.04 LTS (64-bit x86)
   - **Instance Type:** `t2.micro` (Free Tier) or `t3.small`
   - **Key Pair:** Select an existing key pair or generate a new `.pem` key (e.g., `ec2-dev-key.pem`).
3. Configure **Network Settings (Security Group Inbound Rules)**:
   - **Rule 1 (SSH):** Type `SSH`, Port `22`, Source `0.0.0.0/0` *(allows SSH access & CI/CD runner)*
   - **Rule 2 (Web Port):** Type `Custom TCP`, Port `8080`, Source `0.0.0.0/0` *(allows web visitors)*
4. Click **Launch Instance**. Copy the **Public IPv4 address** (e.g., `13.232.45.120`).

---

### Phase 2: EC2 Server Initialization & Systemd Setup

Connect to your EC2 instance via terminal/WSL and configure the server runtime environment:

```bash
# 1. Connect to EC2 via SSH
ssh -i "path/to/ec2-dev-key.pem" ubuntu@<YOUR-EC2-PUBLIC-IP>

# 2. Update system packages & install OpenJDK 17 Runtime
sudo apt update && sudo apt upgrade -y
sudo apt install -y openjdk-17-jre-headless
java -version

# 3. Create deployment directory with proper permissions
sudo mkdir -p /opt/java-login-app
sudo chown -R ubuntu:ubuntu /opt/java-login-app
```

Now, create a background `systemd` daemon configuration file:

```bash
sudo nano /etc/systemd/system/java-login-app.service
```

Paste the following system unit configuration:

```ini
[Unit]
Description=Java Spring Boot Authentication Application
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/opt/java-login-app
ExecStart=/usr/bin/java -jar /opt/java-login-app/app.jar
SuccessExitStatus=143
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Save and exit (`Ctrl + O`, `Enter`, `Ctrl + X`).

Enable the service and grant passwordless service restart access to user `ubuntu`:

```bash
sudo systemctl daemon-reload
sudo systemctl enable java-login-app
echo "ubuntu ALL=(ALL) NOPASSWD: /bin/systemctl restart java-login-app" | sudo tee /etc/sudoers.d/java-login-app
```

---

### Phase 3: GitHub Actions Secrets Setup

To allow GitHub Actions to copy builds and restart services on your EC2 instance securely, configure your repository secrets:

1. Go to **GitHub Repository** → **Settings** → **Secrets and variables** → **Actions**.
2. Click **New repository secret** and add the following 3 environment secrets:

| Secret Name | Value Description | Example Value |
| :--- | :--- | :--- |
| `EC2_HOST` | Public IPv4 address of your running EC2 instance | `13.232.45.120` |
| `EC2_USER` | Default SSH username for Ubuntu instances | `ubuntu` |
| `EC2_SSH_KEY` | Full raw text content of your `.pem` private key file | `-----BEGIN RSA PRIVATE KEY----- ...` |

---

### Phase 4: Running the Pipeline & Automated Deployment

Every push to the `main` branch automatically triggers the workflow `.github/workflows/java-ci.yml`.

To deploy manually or trigger a build:

1. Commit and push any code change to `main`:
   ```bash
   git add .
   git commit -m "feat: updated user dashboard design"
   git push origin main
   ```
2. Navigate to the **Actions** tab in GitHub to watch live build logs:
   - **`build-and-test` Job:** Compiles Java code, runs JUnit unit tests, and outputs `target/app.jar`.
   - **`deploy-to-ec2` Job:** Transports `app.jar` via SCP to `/opt/java-login-app/` on EC2, restarts the `systemd` background service, and runs an HTTP health check (`curl -f http://localhost:8080/health`).

---

### Phase 5: Verification & Health Check

Once GitHub Actions shows a green tick (`✅`), verify your live application:

1. Open your web browser and navigate to:
   ```text
   http://<YOUR-EC2-PUBLIC-IP>:8080
   ```
2. Verify the application health endpoint:
   ```text
   http://<YOUR-EC2-PUBLIC-IP>:8080/health
   ```

   *Expected Response:*
   ```json
   {
     "status": "UP",
     "service": "java-login-app"
   }
   ```

---

## 🔑 Demo User Credentials

Test application authentication and role authorization using these default accounts:

| User Role | Username | Password | Access Rights |
| :--- | :--- | :--- | :--- |
| **Administrator** | `admin` | `admin123` | Full dashboard access (`ROLE_ADMIN`) |
| **Standard User** | `engineer` | `devops2026` | User dashboard access (`ROLE_USER`) |

---

## 🌐 Application Endpoints & Security Matrix

| HTTP Method | Route Endpoint | Authentication Level | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/login` | Public | Renders sign-in form page |
| `POST` | `/login` | Public | Validates user credentials & sets session cookie |
| `GET` | `/dashboard` | Authenticated | Protected landing dashboard for authenticated users |
| `POST` | `/logout` | Authenticated | Clears active session and redirects to `/login?logout=true` |
| `GET` | `/health` | Public | Deployment health check API |

---

## 🧪 Automated Testing

The project includes unit tests using Spring Boot's test suite and `MockMvc`. To execute tests locally:

```bash
mvn test
```

### Included Test Cases:
- `healthEndpointReturnsUp()`: Verifies HTTP 200 OK and status JSON response from `/health`.
- `loginWithValidCredentialsSucceeds()`: Tests valid user login and session redirection to `/dashboard`.
- `loginWithInvalidCredentialsFails()`: Verifies failed login redirects to `/login?error=true`.

---

## 🔧 Server Management & Troubleshooting

### Useful EC2 Management Commands
If you need to inspect or manage the application directly on your EC2 instance:

```bash
# Check service status
sudo systemctl status java-login-app

# Restart the application manually
sudo systemctl restart java-login-app

# Stop/Start the application
sudo systemctl stop java-login-app
sudo systemctl start java-login-app

# View live application logs
sudo journalctl -u java-login-app -f
```

### Troubleshooting Common Issues

| Issue | Cause | Solution |
| :--- | :--- | :--- |
| **Browser times out** | EC2 Security Group blocking port 8080 | Add an Inbound Rule allowing TCP port `8080` from `0.0.0.0/0`. |
| **GitHub Action SSH error** | Incorrect `EC2_SSH_KEY` or `EC2_HOST` | Ensure the entire contents of your `.pem` key (including header/footer) are copied into GitHub Secrets. |
| **`systemctl restart` permission denied** | Sudoers configuration missing | Re-run `echo "ubuntu ALL=(ALL) NOPASSWD: /bin/systemctl restart java-login-app" \| sudo tee /etc/sudoers.d/java-login-app`. |
| **App fails on startup** | Port 8080 already in use | Identify process using port 8080 via `sudo lsof -i :8080` and terminate it (`sudo kill -9 <PID>`). |

---

## 📜 License

This project is licensed under the [MIT License](LICENSE).
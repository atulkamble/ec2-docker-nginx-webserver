## 📦 Project: **Deploy a Static Website on Nginx with Persistent Logs using Docker on EC2 Amazon Linux**

---

## 📌 Overview

We’ll:

* Launch an Amazon Linux EC2 instance.
* Install required packages (Docker, Git, etc.).
* Clone a GitHub repo with a simple website.
* Run the website using Nginx in Docker.
* Use Docker volume to persist Nginx logs.
* Expose it via EC2’s public IP.
* Configure security group for HTTP access.

---

## 📜 Requirements:

* AWS account
* Amazon Linux 2 AMI EC2 instance (t2.micro is fine for free tier)
* Security group with port 80 open

---

## 📂 Project Structure (on EC2)

```
/home/ec2-user/docker-nginx-project/
├── Dockerfile
├── docker-compose.yml
├── nginx/
│   └── default.conf
├── html/
│   └── index.html
```

---

## 🚀 Step-by-Step Execution:

### ✅ 1️⃣ Launch EC2 Instance

* AMI: **Amazon Linux 2**
* Instance type: **t2.micro**
* Security Group: Allow **HTTP (80)** and **SSH (22)**

---

### ✅ 2️⃣ Connect to EC2

```bash
ssh -i "your-key.pem" ec2-user@<ec2-public-ip>
```

---

### ✅ 3️⃣ Install Docker & Git

```bash
sudo yum update -y
sudo yum install docker git -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user
```

Then reboot:

```bash
sudo reboot
```

Reconnect via SSH.

---

### ✅ 4️⃣ Create Project Directory

```bash
mkdir docker-nginx-project
cd docker-nginx-project
```

---

### ✅ 5️⃣ Create HTML Page

```bash
mkdir html
echo "<h1>Hello from EC2 + Nginx + Docker!</h1>" > html/index.html
```

---

### ✅ 6️⃣ Create Nginx Config

```bash
mkdir nginx
cat <<EOT > nginx/default.conf
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
}
EOT
```

---

### ✅ 7️⃣ Create Dockerfile

```bash
cat <<EOT > Dockerfile
FROM nginx:latest
COPY html /usr/share/nginx/html
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
EOT
```

---

### ✅ 8️⃣ Create Docker Volume for Logs

```bash
docker volume create nginx_logs
```

---

### ✅ 9️⃣ Create Docker Compose File

```bash
cat <<EOT > docker-compose.yml
version: '3'
services:
  nginx:
    build: .
    ports:
      - "80:80"
    volumes:
      - nginx_logs:/var/log/nginx

volumes:
  nginx_logs:
EOT
```

---

### ✅ 🔟 Build and Run

```bash
docker-compose up --build -d
```

---

### ✅ 1️⃣1️⃣ Test in Browser

Visit:

```
http://<ec2-public-ip>
```

You should see:
**Hello from EC2 + Nginx + Docker!**

---

### ✅ 1️⃣2️⃣ Check Logs

```bash
docker volume ls
docker volume inspect docker-nginx-project_nginx_logs
docker exec -it <nginx_container_id> cat /var/log/nginx/access.log
```

---

## 📦 Final Directory Snapshot

```bash
tree
.
├── Dockerfile
├── docker-compose.yml
├── html
│   └── index.html
└── nginx
    └── default.conf
```

---

## 📑 Clean Up

To stop and remove everything:

```bash
docker-compose down --volumes
```

---

## 📖 Bonus: Package into a GitHub Repo

You can push this whole `docker-nginx-project` folder into your GitHub for reuse or portfolio.

---

## ✅ Recap

✔️ EC2 with Amazon Linux
✔️ Docker + Docker Compose
✔️ Nginx container serving HTML
✔️ Docker volume for logs
✔️ Exposed via EC2 public IP

**EC2 User Data script** for automated setup on instance launch, **and** a complete **CloudFormation template** to provision the EC2 instance, security group, and automate Docker + Nginx website deployment via User Data.

---

## 📜 1️⃣ EC2 User Data Script (bash)

Use this in the **Advanced Details → User data** section when launching an EC2 instance (Amazon Linux 2):

```bash
#!/bin/bash
# Update system and install Docker
yum update -y
yum install -y docker git

# Start and enable Docker
systemctl start docker
systemctl enable docker
usermod -aG docker ec2-user

# Install Docker Compose
curl -L "https://github.com/docker/compose/releases/download/v2.26.1/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# Create project directory
mkdir /home/ec2-user/docker-nginx-project
cd /home/ec2-user/docker-nginx-project

# Create index.html
mkdir html
echo "<h1>Hello from EC2 + Nginx + Docker (Automated)!</h1>" > html/index.html

# Create nginx config
mkdir nginx
cat <<EOT > nginx/default.conf
server {
    listen 80;
    server_name localhost;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
}
EOT

# Create Dockerfile
cat <<EOT > Dockerfile
FROM nginx:latest
COPY html /usr/share/nginx/html
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
EOT

# Create Docker Compose file
cat <<EOT > docker-compose.yml
version: '3'
services:
  nginx:
    build: .
    ports:
      - "80:80"
    volumes:
      - nginx_logs:/var/log/nginx

volumes:
  nginx_logs:
EOT

# Create Docker volume
docker volume create nginx_logs

# Build and run container
docker-compose up --build -d
```

✅ When the instance boots, it’ll install Docker, set up the Nginx container, and serve your HTML content.

---

## 📜 2️⃣ AWS CloudFormation Template (YAML)

This template will:

* Create a security group (port 22, 80)
* Launch an EC2 instance (Amazon Linux 2)
* Run the same User Data script at startup

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: EC2 with Dockerized Nginx Webserver (Persistent Logs)

Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow SSH and HTTP
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0c768662cc797cd75  # Amazon Linux 2 AMI (Mumbai region, update per your region)
      KeyName: your-key-pair-name  # Replace with your actual key pair name
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum update -y
          yum install -y docker git
          systemctl start docker
          systemctl enable docker
          usermod -aG docker ec2-user
          curl -L "https://github.com/docker/compose/releases/download/v2.26.1/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose
          chmod +x /usr/local/bin/docker-compose
          mkdir /home/ec2-user/docker-nginx-project
          cd /home/ec2-user/docker-nginx-project
          mkdir html
          echo "<h1>Hello from EC2 + Nginx + Docker (CF Automated)!</h1>" > html/index.html
          mkdir nginx
          cat <<EOT > nginx/default.conf
          server {
              listen 80;
              server_name localhost;
              location / {
                  root /usr/share/nginx/html;
                  index index.html;
              }
              access_log /var/log/nginx/access.log;
              error_log /var/log/nginx/error.log;
          }
          EOT
          cat <<EOT > Dockerfile
          FROM nginx:latest
          COPY html /usr/share/nginx/html
          COPY nginx/default.conf /etc/nginx/conf.d/default.conf
          EOT
          cat <<EOT > docker-compose.yml
          version: '3'
          services:
            nginx:
              build: .
              ports:
                - "80:80"
              volumes:
                - nginx_logs:/var/log/nginx
          volumes:
            nginx_logs:
          EOT
          docker volume create nginx_logs
          docker-compose up --build -d

Outputs:
  PublicIP:
    Description: Public IP of EC2 Instance
    Value: !GetAtt WebServerInstance.PublicIp
```

> ⚠️ Replace `ami-0c768662cc797cd75` with the latest Amazon Linux 2 AMI ID in your region (you can fetch it via AWS Console or CLI).

---

## ✅ Recap:

* **EC2 User Data Script** → for quick one-off manual EC2 launches
* **CloudFormation Template** → for full IaC-managed infrastructure

## 📦 Project: **EC2 with Dockerized Nginx Webserver (with Docker Compose & Persistent Logs)**

---

## 📂 Terraform Project Structure:

```
terraform-ec2-nginx/
├── main.tf
├── variables.tf
├── outputs.tf
├── user-data.sh
```

---

## 📜 1️⃣ `user-data.sh` (Same Bash script from earlier)

Create a file `user-data.sh`:

```bash
#!/bin/bash
yum update -y
yum install -y docker git
systemctl start docker
systemctl enable docker
usermod -aG docker ec2-user

curl -L "https://github.com/docker/compose/releases/download/v2.26.1/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

mkdir /home/ec2-user/docker-nginx-project
cd /home/ec2-user/docker-nginx-project

mkdir html
echo "<h1>Hello from EC2 + Nginx + Docker (Terraform Automated)!</h1>" > html/index.html

mkdir nginx
cat <<EOT > nginx/default.conf
server {
    listen 80;
    server_name localhost;
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
}
EOT

cat <<EOT > Dockerfile
FROM nginx:latest
COPY html /usr/share/nginx/html
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
EOT

cat <<EOT > docker-compose.yml
version: '3'
services:
  nginx:
    build: .
    ports:
      - "80:80"
    volumes:
      - nginx_logs:/var/log/nginx
volumes:
  nginx_logs:
EOT

docker volume create nginx_logs
docker-compose up --build -d
```

---

## 📜 2️⃣ `main.tf`

```hcl
provider "aws" {
  region = var.aws_region
}

resource "aws_key_pair" "deployer" {
  key_name   = var.key_name
  public_key = file(var.public_key_path)
}

resource "aws_security_group" "web_sg" {
  name        = "web-server-sg"
  description = "Allow SSH and HTTP access"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_instance" "web_server" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = var.instance_type
  key_name               = aws_key_pair.deployer.key_name
  vpc_security_group_ids = [aws_security_group.web_sg.id]
  user_data              = file("user-data.sh")

  tags = {
    Name = "Terraform-Nginx-Docker-EC2"
  }
}

output "instance_public_ip" {
  description = "Public IP of the EC2 instance"
  value       = aws_instance.web_server.public_ip
}
```

---

## 📜 3️⃣ `variables.tf`

```hcl
variable "aws_region" {
  description = "AWS region to deploy resources"
  type        = string
  default     = "ap-south-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "key_name" {
  description = "Name for the AWS Key Pair"
  type        = string
}

variable "public_key_path" {
  description = "Path to your local public SSH key"
  type        = string
}
```

---

## 📜 4️⃣ `outputs.tf`

```hcl
output "ec2_public_ip" {
  value       = aws_instance.web_server.public_ip
  description = "Public IP Address of the deployed EC2 instance"
}
```

---

## ✅ 5️⃣ Usage Instructions:

### 📌 Initialize Terraform:

```bash
terraform init
```

### 📌 Validate the code:

```bash
terraform validate
```

### 📌 Plan to see what will happen:

```bash
terraform plan -var="key_name=your-key-name" -var="public_key_path=/path/to/public_key.pub"
```

### 📌 Apply and deploy:

```bash
terraform apply -var="key_name=your-key-name" -var="public_key_path=/path/to/public_key.pub"
```

Once deployed, Terraform will output the public IP — visit it in your browser.

---

## ✅ Recap:

✔️ Amazon EC2 via Terraform
✔️ Custom Security Group
✔️ Docker + Docker Compose setup via User Data
✔️ Dockerized Nginx website deployment
✔️ Clean modular Terraform files



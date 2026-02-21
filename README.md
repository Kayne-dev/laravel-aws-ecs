# 🚀 Laravel on AWS ECS - Complete Deployment Guide

A production-ready Laravel application deployed on AWS ECS (Elastic Container Service) with Docker containerization, demonstrating modern cloud-native deployment practices.

![AWS](https://img.shields.io/badge/AWS-ECS-orange)
![Docker](https://img.shields.io/badge/Docker-Containerized-blue)
![Laravel](https://img.shields.io/badge/Laravel-11.x-red)
![PHP](https://img.shields.io/badge/PHP-8.2-purple)
![License](https://img.shields.io/badge/License-MIT-green)

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Local Development Setup](#local-development-setup)
- [Docker Configuration](#docker-configuration)
- [AWS Deployment](#aws-deployment)
- [Troubleshooting](#troubleshooting)
- [Lessons Learned](#lessons-learned)
- [Contributing](#contributing)
- [License](#license)

---

## 🎯 Overview

This project demonstrates the complete journey of deploying a Laravel application to AWS using modern DevOps practices. From local Docker development to production deployment on AWS ECS with Application Load Balancer, this repository serves as a comprehensive guide for cloud deployment.

**Live Demo:** `http://laravel-simple-alb-709460486.us-east-1.elb.amazonaws.com/`

**What This Project Covers:**
- ✅ Docker containerization of Laravel applications
- ✅ Multi-stage Docker builds for optimized images
- ✅ AWS ECR (Elastic Container Registry) image management
- ✅ AWS ECS (Fargate) serverless container orchestration
- ✅ Application Load Balancer configuration
- ✅ CloudWatch logging and monitoring
- ✅ Security group and VPC networking
- ✅ Production-ready configurations

---

## 🏗️ Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         INTERNET                             │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │   Application Load Balancer        │
        │   (laravel-simple-alb)            │
        │   Port 80 (HTTP)                  │
        └───────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │      Target Group (IP-based)      │
        │      Health Check: /              │
        └───────────────────────────────────┘
                            │
        ┌───────────────────┴───────────────────┐
        │                                        │
        ▼                                        ▼
┌──────────────┐                        ┌──────────────┐
│  ECS Task 1  │                        │  ECS Task 2  │
│  (Fargate)   │                        │  (Fargate)   │
│              │                        │              │
│ ┌──────────┐ │                        │ ┌──────────┐ │
│ │ Laravel  │ │                        │ │ Laravel  │ │
│ │Container │ │                        │ │Container │ │
│ │          │ │                        │ │          │ │
│ │ • Nginx  │ │                        │ │ • Nginx  │ │
│ │ • PHP-FPM│ │                        │ │ • PHP-FPM│ │
│ │ • App    │ │                        │ │ • App    │ │
│ └──────────┘ │                        │ └──────────┘ │
└──────────────┘                        └──────────────┘
        │                                        │
        └────────────────┬───────────────────────┘
                         │
                         ▼
                ┌─────────────────┐
                │   CloudWatch    │
                │   Logs & Metrics│
                └─────────────────┘
```

### Deployment Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    DEVELOPMENT PHASE                         │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │   1. Local Development            │
        │   • Code Laravel app              │
        │   • Test with docker-compose      │
        └───────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │   2. Docker Containerization      │
        │   • Create Dockerfile             │
        │   • Configure nginx + PHP-FPM     │
        │   • Set up Supervisor             │
        │   • Build image locally           │
        └───────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │   3. Local Testing                │
        │   • docker build                  │
        │   • docker run -p 8000:80         │
        │   • Test on localhost:8000        │
        │   • Verify all services running   │
        └───────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                      AWS SETUP PHASE                         │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │   4. Push to Amazon ECR           │
        │   • aws ecr get-login-password    │
        │   • docker tag                    │
        │   • docker push                   │
        └───────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │   5. Configure AWS Resources      │
        │   • Create VPC & Subnets          │
        │   • Configure Security Groups     │
        │   • Set up IAM Roles              │
        │   • Create CloudWatch Log Groups  │
        └───────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │   6. Create Load Balancer         │
        │   • Application Load Balancer     │
        │   • Target Group (IP-based)       │
        │   • Health Check Configuration    │
        │   • Listener Rules                │
        └───────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │   7. Create ECS Resources         │
        │   • ECS Cluster (Fargate)         │
        │   • Task Definition               │
        │   • Environment Variables         │
        │   • Container Configuration       │
        └───────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │   8. Deploy ECS Service           │
        │   • Link to ALB                   │
        │   • Set desired task count        │
        │   • Configure auto-scaling        │
        │   • Deploy!                       │
        └───────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                   PRODUCTION PHASE                           │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │   9. Monitor & Verify             │
        │   • Check CloudWatch Logs         │
        │   • Verify Health Checks          │
        │   • Test via ALB DNS              │
        │   • Monitor Metrics               │
        └───────────────────────────────────┘
                            │
                            ▼
                ┌───────────────────┐
                │  🎉 LIVE ON AWS!  │
                └───────────────────┘
```

### Network Flow

```
User Request
     │
     ▼
┌─────────────────┐
│  Internet       │
│  Gateway        │
└─────────────────┘
     │
     ▼
┌─────────────────────────────────┐
│  Security Group: laravel-alb-sg │
│  Inbound: 0.0.0.0/0:80          │
└─────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────┐
│  Application Load Balancer      │
│  Public Subnets (Multi-AZ)      │
└─────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────┐
│  Security Group: laravel-ecs-sg │
│  Inbound: laravel-alb-sg:80     │
└─────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────┐
│  ECS Tasks (Fargate)            │
│  Private/Public Subnets         │
│  Container Port: 80             │
└─────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────┐
│  CloudWatch Logs                │
│  /ecs/laravel-simple            │
└─────────────────────────────────┘
```

---

## ✨ Features

### Application Features
- ✅ Laravel 11.x with PHP 8.4
- ✅ Nginx web server
- ✅ PHP-FPM for optimal performance
- ✅ Supervisor for process management
- ✅ Redis support (optional)
- ✅ Queue worker support
- ✅ OPcache enabled for production

### Infrastructure Features
- ✅ **Containerized with Docker**
  - Multi-stage builds for optimization
  - Minimal image size
  - Reproducible builds

- ✅ **AWS ECS (Fargate)**
  - Serverless container orchestration
  - Auto-scaling capability
  - Zero server management

- ✅ **High Availability**
  - Multi-AZ deployment
  - Application Load Balancer
  - Health checks and auto-recovery

- ✅ **Monitoring & Logging**
  - CloudWatch integration
  - Centralized logging
  - Real-time metrics

- ✅ **Security**
  - VPC isolation
  - Security group rules
  - IAM role-based access
  - Environment variable management

---

## 📦 Prerequisites

Before you begin, ensure you have:

- **Development Machine:**
  - Docker Desktop installed
  - Git
  - PHP 8.2+ and Composer (for local development)
  - AWS CLI configured
  - Text editor (VS Code recommended)

- **AWS Account:**
  - Active AWS account
  - IAM user with permissions for:
    - ECR (push/pull images)
    - ECS (create clusters, services, tasks)
    - EC2 (security groups, load balancers)
    - CloudWatch (logs)
    - VPC (if creating new VPC)

- **Knowledge:**
  - Basic Docker concepts
  - Laravel framework basics
  - Basic AWS services understanding
  - Command line proficiency

---

## 💻 Local Development Setup

### 1. Clone the Repository

```bash
git clone https://github.com/Kayne-dev/laravel-aws-ecs.git
cd laravel-aws-ecs
```

### 2. Install Dependencies

```bash
# Install PHP dependencies
composer install

# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate
```

### 3. Configure Environment

Edit `.env`:

```env
APP_NAME=LaravelECS
APP_ENV=local
APP_DEBUG=true
APP_KEY=base64:your-generated-key-here

DB_CONNECTION=sqlite
# Or use MySQL/PostgreSQL for local development

CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_CONNECTION=sync
```

### 4. Run with Docker Compose (Local Development)

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Access application
open http://localhost:8000

# Stop services
docker-compose down
```

### 5. Run Migrations (if using database)

```bash
docker-compose exec app php artisan migrate
```

---

## 🐳 Docker Configuration

### Project Structure

```
.
├── Dockerfile                 # Multi-stage production build
├── docker-compose.yml         # Local development environment
├── .dockerignore             # Files to exclude from build
└── docker/
    ├── nginx.conf            # Nginx web server configuration
    ├── supervisord.conf      # Process manager configuration
    └── php.ini               # PHP production settings
```

### Dockerfile Explanation

Our Dockerfile uses a multi-stage build approach:

**Stage 1: Composer Dependencies**
```dockerfile
FROM composer:latest AS composer
WORKDIR /app
COPY composer.json composer.lock ./
RUN composer install --no-dev --optimize-autoloader
```

**Stage 2: Production Image**
```dockerfile
FROM php:8.2-fpm
# Install system dependencies
# Install PHP extensions
# Copy application and dependencies
# Configure nginx + supervisor
# Set permissions
```

### Build the Image Locally

```bash
# Build
docker build -t laraappimg:latest .

# Test run
docker run -p 8000:80 \
  -e APP_KEY="base64:your-app-key" \
  -e APP_ENV=local \
  -e APP_DEBUG=true \
  -e DB_CONNECTION=sqlite \
  laraappimg:latest

# Access
open http://localhost:8000
```

### Key Files

**docker/nginx.conf**
- Configures Nginx as reverse proxy
- Routes requests to PHP-FPM
- Serves static assets
- Security headers

**docker/supervisord.conf**
- Manages multiple processes in container
- Runs PHP-FPM
- Runs Nginx
- Runs Laravel queue worker (optional)

**docker/php.ini**
- Production PHP settings
- OPcache configuration
- Memory limits
- Upload size limits

---

## ☁️ AWS Deployment

### Phase 1: Prepare AWS Environment

#### 1. Configure AWS CLI

```bash
# Configure credentials
aws configure

# Verify
aws sts get-caller-identity
```

#### 2. Create ECR Repository

```bash
# Create repository
aws ecr create-repository \
  --repository-name laraappimg \
  --region us-east-1

# Note the repository URI
# Output: 450230586664.dkr.ecr.us-east-1.amazonaws.com/laraappimg
```

---

### Phase 2: Push Docker Image to ECR

```bash
# 1. Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  450230586664.dkr.ecr.us-east-1.amazonaws.com

# 2. Tag image
docker tag laraappimg:latest \
  450230586664.dkr.ecr.us-east-1.amazonaws.com/laraappimg:latest

# 3. Push to ECR
docker push \
  450230586664.dkr.ecr.us-east-1.amazonaws.com/laraappimg:latest

# 4. Verify
aws ecr describe-images \
  --repository-name laraappimg \
  --region us-east-1
```

---

### Phase 3: Set Up AWS Infrastructure

#### 1. Create CloudWatch Log Group

```bash
aws logs create-log-group \
  --log-group-name /ecs/laravel-simple \
  --region us-east-1
```

#### 2. Create IAM Roles

**Task Execution Role** (allows ECS to pull images and write logs):

```bash
# Create trust policy
cat > ecs-trust-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "ecs-tasks.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }]
}
EOF

# Create role
aws iam create-role \
  --role-name ecsTaskExecutionRole \
  --assume-role-policy-document file://ecs-trust-policy.json

# Attach policy
aws iam attach-role-policy \
  --role-name ecsTaskExecutionRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
```

#### 3. Create Security Groups

**ALB Security Group:**
```bash
# Get default VPC ID
VPC_ID=$(aws ec2 describe-vpcs \
  --filters "Name=isDefault,Values=true" \
  --query "Vpcs[0].VpcId" \
  --output text)

# Create ALB security group
ALB_SG=$(aws ec2 create-security-group \
  --group-name laravel-alb-sg \
  --description "Security group for Laravel ALB" \
  --vpc-id $VPC_ID \
  --output text)

# Allow HTTP traffic
aws ec2 authorize-security-group-ingress \
  --group-id $ALB_SG \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

**ECS Security Group:**
```bash
# Create ECS security group
ECS_SG=$(aws ec2 create-security-group \
  --group-name laravel-ecs-sg \
  --description "Security group for Laravel ECS tasks" \
  --vpc-id $VPC_ID \
  --output text)

# Allow traffic from ALB
aws ec2 authorize-security-group-ingress \
  --group-id $ECS_SG \
  --protocol tcp \
  --port 80 \
  --source-group $ALB_SG
```

#### 4. Create Application Load Balancer

```bash
# Get subnet IDs
SUBNET_IDS=$(aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=$VPC_ID" \
  --query "Subnets[*].SubnetId" \
  --output text)

# Create ALB
ALB_ARN=$(aws elbv2 create-load-balancer \
  --name laravel-simple-alb \
  --subnets $SUBNET_IDS \
  --security-groups $ALB_SG \
  --scheme internet-facing \
  --type application \
  --query 'LoadBalancers[0].LoadBalancerArn' \
  --output text)

# Create target group
TG_ARN=$(aws elbv2 create-target-group \
  --name laravel-simple-tg \
  --protocol HTTP \
  --port 80 \
  --vpc-id $VPC_ID \
  --target-type ip \
  --health-check-path / \
  --query 'TargetGroups[0].TargetGroupArn' \
  --output text)

# Create listener
aws elbv2 create-listener \
  --load-balancer-arn $ALB_ARN \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=$TG_ARN

# Get ALB DNS name
aws elbv2 describe-load-balancers \
  --load-balancer-arns $ALB_ARN \
  --query 'LoadBalancers[0].DNSName' \
  --output text
```

---

### Phase 4: Deploy to ECS

#### 1. Create ECS Cluster

```bash
aws ecs create-cluster \
  --cluster-name laravel-simple \
  --region us-east-1
```

#### 2. Register Task Definition

Create `task-definition.json`:

```json
{
  "family": "laravel-simple-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::450230586664:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "laravel-app",
      "image": "450230586664.dkr.ecr.us-east-1.amazonaws.com/laraappimg:latest",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {"name": "APP_ENV", "value": "production"},
        {"name": "APP_DEBUG", "value": "false"},
        {"name": "APP_KEY", "value": "base64:your-app-key-here"},
        {"name": "DB_CONNECTION", "value": "sqlite"},
        {"name": "CACHE_DRIVER", "value": "file"},
        {"name": "SESSION_DRIVER", "value": "file"},
        {"name": "QUEUE_CONNECTION", "value": "sync"}
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/laravel-simple",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "laravel"
        }
      }
    }
  ]
}
```

Register it:

```bash
aws ecs register-task-definition \
  --cli-input-json file://task-definition.json
```

#### 3. Create ECS Service

```bash
aws ecs create-service \
  --cluster laravel-simple \
  --service-name laravel-simple-service \
  --task-definition laravel-simple-task \
  --desired-count 1 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[$SUBNET_IDS],securityGroups=[$ECS_SG],assignPublicIp=ENABLED}" \
  --load-balancers targetGroupArn=$TG_ARN,containerName=laravel-app,containerPort=80
```

#### 4. Monitor Deployment

```bash
# Watch service events
aws ecs describe-services \
  --cluster laravel-simple \
  --services laravel-simple-service

# View logs
aws logs tail /ecs/laravel-simple --follow

# Check task status
aws ecs list-tasks \
  --cluster laravel-simple \
  --service-name laravel-simple-service
```

---

## 🔍 Monitoring & Debugging

### CloudWatch Logs

```bash
# Tail logs in real-time
aws logs tail /ecs/laravel-simple --follow

# Filter for errors
aws logs tail /ecs/laravel-simple --follow --filter-pattern "ERROR"

# Get specific log stream
aws logs get-log-events \
  --log-group-name /ecs/laravel-simple \
  --log-stream-name laravel/laravel-app/task-id
```

### ECS Service Health

```bash
# Check service status
aws ecs describe-services \
  --cluster laravel-simple \
  --services laravel-simple-service \
  --query 'services[0].{Status:status,Running:runningCount,Desired:desiredCount}'

# Check task health
aws ecs describe-tasks \
  --cluster laravel-simple \
  --tasks $(aws ecs list-tasks --cluster laravel-simple --service-name laravel-simple-service --query 'taskArns[0]' --output text)
```

### Target Group Health

```bash
# Check target health
aws elbv2 describe-target-health \
  --target-group-arn $TG_ARN
```

---

## 🐛 Troubleshooting

### Common Issues and Solutions

#### Issue 1: Container Fails to Start

**Symptoms:**
- Tasks transition to STOPPED state immediately
- CloudWatch shows initialization errors

**Solutions:**
```bash
# Check logs for specific error
aws logs tail /ecs/laravel-simple --follow

# Common fixes:
# 1. Missing APP_KEY
#    Update task definition with valid APP_KEY

# 2. Permission issues
#    Check Dockerfile sets correct permissions:
#    RUN chown -R www-data:www-data /var/www/html/storage
#    RUN chmod -R 775 /var/www/html/storage

# 3. Missing directories
#    Ensure Dockerfile creates required Laravel directories
```

#### Issue 2: Health Checks Failing

**Symptoms:**
- Tasks start but are marked unhealthy
- Target group shows unhealthy targets

**Solutions:**
```bash
# Verify health check path
aws elbv2 describe-target-groups \
  --target-group-arns $TG_ARN \
  --query 'TargetGroups[0].HealthCheckPath'

# Check if app responds on /
docker run -p 8000:80 laraappimg:latest
curl http://localhost:8000

# Update health check if needed
aws elbv2 modify-target-group \
  --target-group-arn $TG_ARN \
  --health-check-path /
```

#### Issue 3: 502 Bad Gateway

**Symptoms:**
- ALB returns 502 error
- Tasks show as running

**Solutions:**
```bash
# 1. Check security groups
#    Ensure ECS security group allows traffic from ALB security group

# 2. Verify container port
#    Task definition containerPort must match docker EXPOSE port (80)

# 3. Check CloudWatch logs
aws logs tail /ecs/laravel-simple --follow
```

#### Issue 4: Cannot Access via ALB

**Symptoms:**
- ALB DNS doesn't resolve
- Connection timeout

**Solutions:**
```bash
# 1. Check ALB security group allows public access
aws ec2 describe-security-groups --group-ids $ALB_SG

# Should have rule:
# IpPermissions: [{IpProtocol: tcp, FromPort: 80, ToPort: 80, IpRanges: [{CidrIp: 0.0.0.0/0}]}]

# 2. Verify ALB is active
aws elbv2 describe-load-balancers --load-balancer-arns $ALB_ARN
# State should be: "active"

# 3. Check target registration
aws elbv2 describe-target-health --target-group-arn $TG_ARN
```

#### Issue 5: APP_KEY Cipher Error

**Symptoms:**
- Laravel shows "Unsupported cipher" error
- 500 error in browser

**Solutions:**
```bash
# Generate new APP_KEY
php artisan key:generate --show

# Update task definition with new key
# Must be format: base64:xxxxxxxxxxxxx

# Force new deployment
aws ecs update-service \
  --cluster laravel-simple \
  --service laravel-simple-service \
  --force-new-deployment
```

---

## 📚 Lessons Learned

### 1. Docker Containerization

**Challenge:** Nginx and PHP-FPM process management in a single container

**Solution:** Used Supervisor to manage multiple processes (nginx, php-fpm, queue workers)

**Key Takeaway:** 
- Single-process containers are ideal, but for monolithic apps, Supervisor is a pragmatic solution
- Always run containers locally before deploying to cloud
- Test with the exact environment variables you'll use in production

### 2. AWS Networking

**Challenge:** Understanding the flow from ALB → ECS tasks and security group configuration

**Solution:** Drew out the network architecture diagram and configured security groups step-by-step

**Key Takeaway:**
- Security groups are stateful - if you allow inbound, outbound is automatic
- ALB needs to be in public subnets with internet gateway
- ECS tasks can be in public or private subnets (public is simpler for Fargate)

### 3. Environment Variables vs Secrets Manager

**Challenge:** Initial confusion between task definition environment variables and AWS Secrets Manager

**Solution:** Started with environment variables for simplicity, can migrate to Secrets Manager later

**Key Takeaway:**
- Environment variables in task definition are fine for non-sensitive data
- Use AWS Secrets Manager for production secrets (database passwords, API keys)
- Always validate APP_KEY format: must be `base64:xxxxx`

### 4. CloudWatch Logging

**Challenge:** Finding relevant logs among thousands of log entries

**Solution:** Used log stream filters and `aws logs tail` command

**Key Takeaway:**
- Set up structured logging from day one
- Use log prefixes to identify different services
- CloudWatch Insights is powerful for log analysis

### 5. Health Checks

**Challenge:** Tasks passing container health checks but failing ALB health checks

**Solution:** Ensured health check path in target group matches what the app serves

**Key Takeaway:**
- Container health checks != Target group health checks
- Always test health check endpoint locally first
- Use `/` or a dedicated `/health` endpoint
- Set appropriate grace periods for app startup

---

## 🚀 Deployment Checklist

Before deploying to production, ensure:

- [ ] All environment variables are set correctly
- [ ] APP_KEY is properly generated and base64 encoded
- [ ] Docker image builds successfully
- [ ] Container runs locally without errors
- [ ] Health check endpoint returns 200 OK
- [ ] CloudWatch log group exists
- [ ] IAM roles have correct permissions
- [ ] Security groups are properly configured
- [ ] ALB is in internet-facing mode
- [ ] Target group is type "IP addresses"
- [ ] ECS tasks have public IP enabled (or use NAT gateway)
- [ ] Subnets span at least 2 availability zones

---

## 🔧 Maintenance

### Update Application

```bash
# 1. Build new image
docker build -t laraappimg:v2 .

# 2. Tag for ECR
docker tag laraappimg:v2 \
  450230586664.dkr.ecr.us-east-1.amazonaws.com/laraappimg:v2

# 3. Push to ECR
docker push \
  450230586664.dkr.ecr.us-east-1.amazonaws.com/laraappimg:v2

# 4. Update task definition with new image tag
# (Create new revision via console or CLI)

# 5. Update service
aws ecs update-service \
  --cluster laravel-simple \
  --service laravel-simple-service \
  --force-new-deployment
```

### Scale Application

```bash
# Horizontal scaling (more tasks)
aws ecs update-service \
  --cluster laravel-simple \
  --service laravel-simple-service \
  --desired-count 3

# Vertical scaling (bigger tasks)
# Update task definition with more CPU/Memory
# Then update service to use new revision
```

### View Metrics

```bash
# CPU utilization
aws cloudwatch get-metric-statistics \
  --namespace AWS/ECS \
  --metric-name CPUUtilization \
  --dimensions Name=ServiceName,Value=laravel-simple-service Name=ClusterName,Value=laravel-simple \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z \
  --period 3600 \
  --statistics Average
```

---

## 💰 Cost Optimization

### Estimated Monthly Costs

Based on us-east-1 pricing:

- **ECS Fargate (1 task, 0.5 vCPU, 1GB):** ~$15/month
- **Application Load Balancer:** ~$20/month
- **Data Transfer:** ~$5/month (1GB out)
- **CloudWatch Logs:** ~$1/month (1GB)
- **ECR Storage:** ~$1/month (first 500MB free)

**Total:** ~$42/month for 1 task

### Cost Saving Tips

1. **Use Fargate Spot** for dev/test (70% savings)
2. **Enable auto-scaling** to scale down during low traffic
3. **Set CloudWatch log retention** to 7 days for non-production
4. **Use Aurora Serverless** if using RDS (pay per use)
5. **Delete old ECR images** regularly

---

## 🔐 Security Best Practices

### Implemented

- ✅ Use IAM roles (not access keys) for ECS tasks
- ✅ Security groups follow principle of least privilege
- ✅ No secrets in Dockerfile or code
- ✅ Environment variables for configuration
- ✅ VPC isolation

### Recommended Additions

- [ ] Move secrets to AWS Secrets Manager
- [ ] Enable encryption at rest for logs
- [ ] Use AWS WAF with ALB
- [ ] Enable VPC Flow Logs
- [ ] Implement AWS GuardDuty
- [ ] Use ACM for HTTPS certificates
- [ ] Enable Container Insights
- [ ] Scan Docker images for vulnerabilities

---

## 📈 Next Steps

### Immediate Improvements

1. **Add HTTPS**
   - Request ACM certificate
   - Add HTTPS listener to ALB
   - Redirect HTTP to HTTPS

2. **Custom Domain**
   - Register domain in Route 53
   - Create A record pointing to ALB
   - Update app configuration

3. **Database**
   - Set up RDS (MySQL/PostgreSQL)
   - Update task definition with DB credentials
   - Run migrations

4. **CI/CD Pipeline**
   - Set up GitHub Actions or GitLab CI
   - Automate build and deployment
   - Add automated tests

### Advanced Features

5. **Auto Scaling**
   - Configure service auto-scaling
   - Set CPU/Memory target thresholds
   - Test scaling behavior

6. **Caching**
   - Add ElastiCache (Redis)
   - Configure Laravel to use Redis
   - Improve performance

7. **Monitoring**
   - Set up CloudWatch alarms
   - Configure SNS notifications
   - Create custom dashboards

8. **Blue-Green Deployment**
   - Use CodeDeploy
   - Zero-downtime deployments
   - Easy rollback

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

### How to Contribute

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Areas for Contribution

- Documentation improvements
- Additional deployment scripts
- Terraform/CloudFormation templates
- CI/CD pipeline examples
- Performance optimizations
- Security enhancements

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 📧 Contact

**Kenechukwu Nnaka** - [@yourtwitter](https://x.com/CodeX_DeeFii)

**Project Link:** [https://github.com/Kayne-dev/laravel-aws-ecs](https://github.com/yourusername/laravel-aws-ecs)

**LinkedIn:** [Your LinkedIn](https://www.linkedin.com/in/kenechukwun)

---

## 🙏 Acknowledgments

- Laravel Framework Team
- AWS Documentation
- Docker Community
- Stack Overflow Contributors
- Everyone who helped debug those "mysterious" 502 errors

---

## 📚 Resources

### Documentation
- [Laravel Deployment](https://laravel.com/docs/deployment)
- [AWS ECS Documentation](https://docs.aws.amazon.com/ecs/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/userguide/what-is-fargate.html)

### Tutorials
- [Docker for PHP Developers](https://phpdocker.io/)
- [AWS ECS Workshop](https://ecsworkshop.com/)
- [Container Security](https://kubernetes.io/docs/concepts/security/)

### Tools
- [AWS CLI](https://aws.amazon.com/cli/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Visual Studio Code](https://code.visualstudio.com/)
- [Postman](https://www.postman.com/)

---

## 📊 Project Stats

![GitHub stars](https://img.shields.io/github/stars/Kayne-dev/laravel-aws-ecs?style=social)
![GitHub forks](https://img.shields.io/github/forks/Kayne-dev/laravel-aws-ecs?style=social)
![GitHub issues](https://img.shields.io/github/issues/Kayne-dev/laravel-aws-ecs)
![GitHub pull requests](https://img.shields.io/github/issues-pr/Kayne-dev/laravel-aws-ecs)

---

**⭐ If this project helped you, please star it on GitHub! It helps others discover it too.**

---

*Last Updated: February 2026*

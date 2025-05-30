# ECS Service Terraform Module Development

This Terraform module provisions **Amazon ECS services** integrated with best practices, including support for:

- Application Load Balancers (ALB)
- AWS IAM Roles for Tasks
- ECR Image Deployment
- CloudWatch Logging
- Auto Scaling
- RDS
- Secrets Management via AWS Secrets Manager

---

## 🔧 Module Features

✅ **Reusable and Composable**  
Designed to be easily integrated into different environments or projects.

✅ **Best Practices Built-In**  
Implements AWS-recommended security, scalability, and observability features.

✅ **Flexible Config**  
Supports customization for port mappings, environment variables, container definitions, and task roles.

✅ **Environment Agnostic**  
Supports multi-environment deployments (e.g., dev, staging, prod) via input variables.

---

## 🏗️ Architecture Overview

The module creates the following AWS resources per ECS service:

- ECS Task Definition
- ECS Service with optional ALB integration
- Target Group and Listener Rules
- IAM Task Role and Execution Role
- CloudWatch Log Group
- Auto Scaling policies (CPU/Memory-based)
- Optional SecretsManager parameters injection

---

## 📦 Usage Example

```hcl
module "ecs_service" {
  source = "git::https://github.com/your-org/terraform-aws-ecs-service.git"

  name               = "web-app"
  cluster_arn        = aws_ecs_cluster.main.arn
  task_exec_role_arn = aws_iam_role.ecs_execution.arn
  task_role_arn      = aws_iam_role.ecs_task.arn

  container_definition = {
    name             = "web-app"
    image            = "123456789012.dkr.ecr.us-east-1.amazonaws.com/web-app:latest"
    cpu              = 256
    memory           = 512
    port_mappings    = [{ containerPort = 80, hostPort = 80 }]
    environment      = [{ name = "ENV", value = "production" }]
    secrets          = [{ name = "DB_PASSWORD", valueFrom = "arn:aws:secretsmanager:us-east-1:123456789012:secret:db_pass" }]
    log_configuration = {
      logDriver = "awslogs"
      options = {
        awslogs-group         = "/ecs/web-app"
        awslogs-region        = "us-east-1"
        awslogs-stream-prefix = "ecs"
      }
    }
  }

  alb_target_group_arn = aws_lb_target_group.app.arn
  desired_count        = 2
  enable_autoscaling   = true
  min_capacity         = 2
  max_capacity         = 6
  scale_cpu_threshold  = 60
}

# Multi-Tenant AI Agent Platform

> **Production-ready infrastructure for deploying intelligent AI agents at scale on AWS**

A complete infrastructure-as-code solution demonstrating enterprise-grade cloud architecture, security best practices, and AI integration. Built with Terraform, AWS ECS Fargate, PostgreSQL, and Claude AI via AWS Bedrock.

**🎯 Not a product to install** — this is a reference architecture and working implementation you can learn from, fork, and customize.

---

## 🏗️ What This Demonstrates

This project showcases production-ready patterns for:

- **Multi-tenant SaaS architecture** with strict data isolation
- **Serverless AI agent orchestration** using ECS Fargate
- **Zero-trust CI/CD** with GitHub Actions OIDC (no long-lived credentials)
- **Secure database access** via bastion host and private subnets
- **Enterprise AI integration** using AWS Bedrock (Claude 3.5 Sonnet)
- **Infrastructure as Code** with modular Terraform design
- **Cost optimization** for startup/SMB deployments (~$135/month dev environment)

---

## 🤖 Example AI Agents Included

### **Butler (Personal Assistant)**
- Natural language interface powered by Claude AI
- PostgreSQL-backed conversation history
- Multi-tenant isolation via Row-Level Security
- Interactive chat mode for testing

### **Scout (Research Agent)** *(Planned)*
- Web monitoring for business intelligence signals
- Automated prospect research

### **Writer (Content Generator)** *(Planned)*
- Sales collateral generation
- Personalized outreach creation

---

## 🏛️ Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│  GitHub (Version Control + CI/CD)                       │
│  ├─ Infrastructure as Code (Terraform)                  │
│  ├─ Agent Code (Python)                                 │
│  └─ Automated Deployment (GitHub Actions + OIDC)        │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│  AWS VPC (10.0.0.0/16)                                  │
│                                                          │
│  ┌─────────────────┐      ┌─────────────────┐          │
│  │ Public Subnets  │      │ Private Subnets │          │
│  │ (2 AZs)         │      │ (2 AZs)         │          │
│  │                 │      │                 │          │
│  │ • NAT Gateways  │──────│ • ECS Fargate   │          │
│  │ • Bastion Host  │      │ • RDS Postgres  │          │
│  └─────────────────┘      └─────────────────┘          │
│                                                          │
│  Security:                                               │
│  • Database in private subnet (no internet access)      │
│  • Agents isolated per tenant                           │
│  • Secrets in AWS Secrets Manager                       │
│  • VPC endpoints (S3, ECR, Bedrock)                     │
└─────────────────────────────────────────────────────────┘
```

---

## 🔐 Security Highlights

### **Zero-Trust CI/CD**
- GitHub Actions uses OpenID Connect (OIDC) to AWS
- No long-lived AWS credentials stored in GitHub
- Temporary credentials issued per workflow run (1-hour expiration)
- Repository-specific IAM trust policies

### **Network Isolation**
- RDS database in private subnets only
- No direct internet access to data layer
- Bastion host for secure administrative access
- VPC endpoints for AWS service communication

### **Multi-Tenant Data Isolation**
- PostgreSQL Row-Level Security policies
- Tenant-scoped S3 prefixes
- Separate ECS task instances per tenant (optional)
- IAM roles limit cross-tenant access

### **Secrets Management**
- AWS Secrets Manager for credentials
- Automatic rotation support
- Encrypted at rest and in transit
- Least-privilege IAM policies

--

## 🚀 Quick Start

### **Prerequisites**
- AWS Account with admin access
- Terraform >= 1.7.0
- AWS CLI configured
- GitHub account

### **1. Bootstrap AWS**
```bash
# Create S3 bucket for Terraform state
aws s3api create-bucket \
  --bucket your-terraform-state-bucket \
  --region us-east-1

# Create DynamoDB table for state locking
aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

### **2. Configure Terraform**
```bash
# Clone repository
git clone https://github.com/cameronkeithhensley/multi-tenant-ai-agent-platform.git
cd multi-tenant-ai-agent-platform/terraform

# Update backend configuration
nano main.tf
# Change bucket name to your S3 bucket

# Initialize and deploy
terraform init
terraform plan
terraform apply
```

### **3. Test Local Agent**
```bash
# Install dependencies
pip3 install -r agents/butler/requirements.txt

# Configure environment
cp agents/butler/.env.example agents/butler/.env
# Edit .env with your database credentials

# Create SSH tunnel to database (in separate terminal)
ssh -i ~/bastion-key.pem -N -L 5432:your-rds-endpoint:5432 ec2-user@bastion-ip

# Run agent
python3 agents/butler/butler.py --health-check
python3 agents/butler/butler.py  # Interactive mode
```

---

## 📂 Project Structure

```
.
├── terraform/
│   ├── main.tf                 # Root module
│   ├── variables.tf            # Input variables
│   ├── outputs.tf              # Outputs
│   └── modules/
│       ├── networking/         # VPC, subnets, NAT, endpoints
│       ├── compute/            # ECS cluster, task definitions
│       ├── database/           # RDS PostgreSQL
│       ├── storage/            # S3 buckets, lifecycle policies
│       ├── security/           # IAM roles, GitHub OIDC
│       └── bastion/            # Jump host for DB access
│
├── agents/
│   ├── shared/
│   │   ├── database.py         # PostgreSQL connection helper
│   │   └─ claude_client.py    # Bedrock/Claude API wrapper
│   │
│   └── butler/                 # Personal assistant agent
│       ├── butler.py           # Main agent logic
│       ├── requirements.txt    # Python dependencies
│       └── Dockerfile          # Container definition
│
└── .github/workflows/
    ├── terraform.yml           # Infrastructure deployment
    └── deploy.yml              # Agent deployment to ECS
```

---

## 🛠️ Technology Stack

### **Infrastructure**
- **IaC:** Terraform 1.7+
- **Cloud:** AWS (VPC, ECS Fargate, RDS, S3, Secrets Manager)
- **CI/CD:** GitHub Actions with OIDC
- **Networking:** Private subnets, NAT Gateway, VPC Endpoints

### **Compute**
- **Runtime:** ECS Fargate (serverless containers)
- **Container Registry:** Amazon ECR
- **Orchestration:** ECS Services with auto-scaling

### **Data**
- **Database:** PostgreSQL 16 (RDS)
- **Storage:** Amazon S3 with lifecycle policies
- **Caching:** (Planned: Redis/ElastiCache)

### **AI/ML**
- **LLM:** Claude 3.5 Sonnet via AWS Bedrock
- **Agent Framework:** Custom (not LangChain - see [architecture decisions](#architecture-decisions))

### **Languages**
- **Infrastructure:** HCL (Terraform)
- **Agents:** Python 3.11
- **CI/CD:** YAML

---

## 🎓 Architecture Decisions

### **Why Custom Agent Framework vs. LangChain?**

**Chose:** Custom implementation (~300 lines)

**Rationale:**
- ✅ Full control over agent behavior
- ✅ Minimal dependencies (boto3, psycopg2)
- ✅ Easier to debug and modify
- ✅ Better performance (no abstraction overhead)
- ✅ Exactly what we need, nothing more

**Trade-off:** Must build features like tool calling ourselves (acceptable for MVP)

### **Why ECS Fargate vs. Kubernetes?**

**Chose:** ECS Fargate

**Rationale:**
- ✅ Simpler operations (no control plane to manage)
- ✅ Lower cost ($0 control plane vs $73/month for EKS)
- ✅ Faster deployment cycles
- ✅ Sufficient for 100+ customers
- ✅ Native AWS integration

**Trade-off:** Less portable than K8s (acceptable - we're AWS-native)

### **Why RDS vs. Aurora?**

**Chose:** RDS PostgreSQL

**Rationale:**
- ✅ Much cheaper at low scale ($13 vs $50/month minimum)
- ✅ Simpler configuration
- ✅ Same PostgreSQL features we need
- ✅ Easy migration path to Aurora later

**Trade-off:** Manual failover vs automatic (acceptable for MVP)

### **Why Bedrock vs. Direct Anthropic API?**

**Chose:** AWS Bedrock

**Rationale:**
- ✅ Centralized AWS billing
- ✅ VPC endpoints (no internet egress required)
- ✅ IAM-based access control
- ✅ Integrated CloudWatch logging
- ✅ Better for team environments

**Trade-off:** Slight latency overhead (acceptable)

---

## 📈 Scaling Considerations

### **Current Capacity**
- **Customers:** 1-10
- **Concurrent Tasks:** 3-10
- **Database:** db.t4g.micro (1 vCPU, 1GB RAM)
- **NAT:** 2 gateways (high availability)

### **Scale to 100 Customers**
- **Customers:** 100
- **Concurrent Tasks:** 30-100
- **Database:** Upgrade to db.t4g.large (2 vCPU, 8GB RAM) or Aurora
- **ECS:** Auto-scaling based on CloudWatch metrics
- **Cost:** ~$8,000/month, $299/customer = $29,900 revenue (63% margin)

### **Scale to 1,000 Customers**
- **Database:** Aurora PostgreSQL (multi-AZ)
- **ECS:** Multiple clusters per region
- **Caching:** ElastiCache (Redis) for session state
- **Cost:** ~$35,000/month, $299/customer = $299,000 revenue (88% margin)

---

## 🔬 Local Development

### **Prerequisites**
- Python 3.11+
- Docker Desktop
- AWS CLI
- PostgreSQL client (psql)

### **Setup**
```bash
# 1. Install Python dependencies
pip3 install -r agents/butler/requirements.txt

# 2. Configure environment
cp agents/butler/.env.example agents/butler/.env
# Edit with your AWS credentials and database password

# 3. Get database password from AWS
aws secretsmanager get-secret-value \
  --secret-id your-db-credentials \
  --query SecretString \
  --output text

# 4. Create SSH tunnel to database (separate terminal)
ssh -i ~/bastion-key.pem -N -L 5432:your-rds:5432 ec2-user@bastion-ip

# 5. Run agent
python3 agents/butler/butler.py
```

### **Docker Development**
```bash
# Build image
docker build -f agents/butler/Dockerfile -t ai-agent:latest .

# Run locally
docker run --rm -it \
  --add-host=host.docker.internal:host-gateway \
  --env-file .env \
  ai-agent:latest

# Or use docker-compose
docker-compose up
```

---

## 🧪 Testing

### **Health Checks**
```bash
# Test agent connectivity
python3 agents/butler/butler.py --health-check

# Expected output:
# ✅ Database connection: PASS
# ✅ Claude API: PASS
# ✅ All systems operational!
```

### **Integration Tests**
```bash
# Test database queries
python3 -c "from agents.shared.database import Database; \
  db = Database('test-tenant'); \
  db.connect(); \
  print(db.execute('SELECT version()'))"

# Test Claude API
python3 -c "from agents.shared.claude_client import ClaudeClient; \
  client = ClaudeClient(); \
  print(client.chat('Say hello in 5 words'))"
```

---

## 📊 Monitoring

### **CloudWatch Metrics**
- ECS task CPU/memory utilization
- RDS connections, CPU, storage
- NAT Gateway data processed
- Bedrock API call latency and errors

### **CloudWatch Logs**
- Agent conversation logs
- Database query logs (slow queries)
- Infrastructure deployment logs

### **Cost Monitoring**
```bash
# Check current month's costs
aws ce get-cost-and-usage \
  --time-period Start=2026-02-01,End=2026-02-28 \
  --granularity MONTHLY \
  --metrics BlendedCost \
  --group-by Type=DIMENSION,Key=SERVICE
```

---

## 🚢 Deployment

### **Automated (Recommended)**
Push to `main` branch triggers GitHub Actions:
1. Terraform validates and applies infrastructure changes
2. Docker images built and pushed to ECR
3. ECS services updated with new task definitions
4. Health checks verify deployment

### **Manual**
```bash
# Deploy infrastructure
cd terraform
terraform apply

# Build and push Docker image
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin your-ecr-url
docker build -f agents/butler/Dockerfile -t your-ecr-url/agent:latest .
docker push your-ecr-url/agent:latest

# Update ECS service
aws ecs update-service \
  --cluster your-cluster \
  --service butler-service \
  --force-new-deployment
```

---

## 🤝 Contributing

This is a personal portfolio project, but feedback and suggestions are welcome!

**Found a bug?** Open an issue  
**Have an improvement?** Submit a pull request  
**Questions?** Start a discussion  

---

## 📄 License

MIT License - Feel free to use this code as a reference or starting point for your own projects.

---

## 🎯 Use Cases

### **For Learning**
- Study production-ready cloud architecture
- Understand multi-tenant SaaS patterns
- Learn Terraform module design
- Explore AI agent orchestration

### **For Hiring Managers**
This project demonstrates:
- ✅ Cloud architecture (AWS, IaC, networking)
- ✅ Security best practices (zero-trust, encryption, least-privilege)
- ✅ DevOps/CI/CD (GitHub Actions, OIDC, automated deployments)
- ✅ AI integration (Bedrock, Claude API)
- ✅ Database design (PostgreSQL, multi-tenancy)
- ✅ Cost optimization (right-sizing, scaling strategies)
- ✅ Code quality (modular, documented, tested)

### **For Customization**
Fork this repository to:
- Build your own AI agent platform
- Adapt the architecture for different use cases
- Use as a starting template for SaaS products
- Learn infrastructure-as-code patterns

---

## 📞 Contact

**Cameron Hensley**  
- GitHub: [@cameronkeithhensley](https://github.com/cameronkeithhensley)
- LinkedIn: [Cameron Hensley](https://linkedin.com/in/cameronhensley)

---

## 🙏 Acknowledgments

Built with guidance from Claude (Anthropic) during an extended pair-programming session demonstrating AI-assisted software development.

**Infrastructure inspired by:**
- AWS Well-Architected Framework
- Terraform Best Practices
- SaaS Architecture Patterns

---

**⭐ If you find this project helpful, please consider starring the repository!**

---

*Last Updated: February 10, 2026*

# AI Support Ticket Automation Platform

An event-driven customer support automation platform built with Python and AWS.

The system receives customer support requests, automatically classifies and prioritizes them using AI, detects duplicate issues, generates suggested responses, and exposes ticket information through a REST API.

The project is designed around serverless AWS services and Infrastructure as Code, with deployment automated through the AWS CLI and CloudFormation.

---

## Problem

Customer support teams often spend significant time manually processing incoming tickets.

For every new request, an agent may need to:

* Read the customer's message
* Identify the issue category
* Determine its priority
* Search for similar previous tickets
* Decide which team should handle it
* Write an initial response
* Update the ticketing system

This project automates the repetitive parts of that workflow while keeping human agents in control of the final response.

---

## Solution

The platform creates an automated pipeline:

```text
Customer Request
       |
       v
   REST API
       |
       v
   API Gateway
       |
       v
     Lambda
       |
       +----------------+
       |                |
       v                v
    DynamoDB        AI Analysis
                       |
                       v
                  Classification
                       |
              +--------+--------+
              |        |        |
              v        v        v
           Category Priority  Team
              |        |        |
              +--------+--------+
                       |
                       v
                Suggested Reply
                       |
                       v
                 Support Agent
```

The AI does not automatically send a response to the customer. Instead, it produces a suggested response that can be reviewed by a support agent.

---

## Features

### Ticket Management

* Create support tickets
* Retrieve individual tickets
* List tickets
* Filter tickets by status
* Filter tickets by priority
* Update ticket status
* Assign tickets to teams

### AI Automation

The AI processing pipeline can determine:

* Ticket category
* Priority
* Sentiment
* Suggested department
* Short summary
* Suggested response

Example:

```json
{
  "category": "billing",
  "priority": "high",
  "department": "payments",
  "sentiment": "negative",
  "summary": "Customer reports a duplicate charge.",
  "suggested_response": "We are reviewing the duplicate transaction..."
}
```

### Duplicate Detection

Incoming tickets can be compared with existing issues to identify potentially duplicated requests.

Example:

```text
Ticket #1042
"Payment was charged twice."

Ticket #1078
"I see two charges for the same order."

Possible duplicate: 91%
```

This allows support teams to group related requests instead of processing every ticket independently.

---

# Architecture

```text
                         ┌───────────────┐
                         │    Client     │
                         └───────┬───────┘
                                 |
                                 v
                         ┌───────────────┐
                         │ API Gateway   │
                         └───────┬───────┘
                                 |
                                 v
                         ┌───────────────┐
                         │    Lambda     │
                         │    Python     │
                         └───────┬───────┘
                                 |
                    ┌────────────┴────────────┐
                    |                         |
                    v                         v
             ┌─────────────┐          ┌─────────────┐
             │  DynamoDB   │          │  AI Service │
             │   Tickets   │          │  Analysis   │
             └─────────────┘          └──────┬──────┘
                                             |
                                             v
                                      Classification
                                             |
                                             v
                                      Suggested Reply
```

---

# AWS Services

| Service        | Responsibility                |
| -------------- | ----------------------------- |
| API Gateway    | REST API                      |
| AWS Lambda     | Serverless Python application |
| DynamoDB       | Ticket storage                |
| Amazon Bedrock | AI analysis                   |
| CloudWatch     | Logging and monitoring        |
| IAM            | Access control                |
| CloudFormation | Infrastructure as Code        |
| AWS CLI        | Deployment and administration |

---

# REST API

## Create Ticket

```http
POST /tickets
```

Request:

```json
{
  "customer_id": "customer-102",
  "subject": "Charged twice for my order",
  "message": "I placed one order but my card was charged twice."
}
```

Response:

```json
{
  "ticket_id": "TCK-10291",
  "status": "open",
  "priority": "high"
}
```

---

## Get Ticket

```http
GET /tickets/{ticket_id}
```

Example:

```bash
curl https://api.example.com/tickets/TCK-10291
```

---

## List Tickets

```http
GET /tickets
```

Optional filters:

```text
/tickets?status=open
/tickets?priority=high
/tickets?category=billing
```

---

## Analyze Ticket

```http
POST /tickets/{ticket_id}/analyze
```

Example response:

```json
{
  "ticket_id": "TCK-10291",
  "category": "billing",
  "priority": "high",
  "sentiment": "negative",
  "department": "payments",
  "confidence": 0.94
}
```

---

# Project Structure

```text
ai-support-automation/
│
├── src/
│   ├── handlers/
│   │   ├── create_ticket.py
│   │   ├── get_ticket.py
│   │   ├── list_tickets.py
│   │   └── analyze_ticket.py
│   │
│   ├── services/
│   │   ├── ai_service.py
│   │   ├── ticket_service.py
│   │   └── database.py
│   │
│   ├── models/
│   │   └── ticket.py
│   │
│   └── utils/
│       ├── logger.py
│       └── validation.py
│
├── infrastructure/
│   └── cloudformation.yaml
│
├── tests/
│   ├── test_ticket.py
│   ├── test_api.py
│   └── test_ai_service.py
│
├── scripts/
│   ├── deploy.sh
│   └── destroy.sh
│
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
```

---

# Infrastructure as Code

AWS resources are defined using CloudFormation.

Example:

```bash
aws cloudformation deploy \
  --template-file infrastructure/cloudformation.yaml \
  --stack-name ai-support-automation \
  --capabilities CAPABILITY_NAMED_IAM
```

Check the deployment:

```bash
aws cloudformation describe-stacks \
  --stack-name ai-support-automation
```

---

# Local Development

Clone the repository:

```bash
git clone https://github.com/yourusername/ai-support-automation.git

cd ai-support-automation
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate it:

### Linux/macOS

```bash
source .venv/bin/activate
```

### Windows

```bash
.venv\Scripts\activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

---

# Environment Variables

Create a `.env` file:

```text
AWS_REGION=us-east-1
DYNAMODB_TABLE=support-tickets
AI_MODEL_ID=your-model-id
LOG_LEVEL=INFO
```

Never commit credentials or `.env` files to GitHub.

---

# Testing

Run the test suite:

```bash
pytest
```

Run with coverage:

```bash
pytest --cov=src
```

---

# Example Workflow

A customer submits:

```text
"My payment was taken twice for the same order.
Please help me get one of the charges refunded."
```

The system creates a ticket:

```text
Ticket ID: TCK-10291
```

The AI analysis produces:

```text
Category: Billing
Priority: High
Department: Payments
Sentiment: Negative
```

The system then generates a suggested response:

```text
We understand that you may have been charged twice
for the same order. Our payments team will review
the transaction and investigate the duplicate charge.
```

The support agent reviews the suggestion before sending it.

---

# Security

The project follows basic AWS security practices:

* IAM roles instead of hard-coded credentials
* Least-privilege permissions
* Environment variables for configuration
* No AWS credentials committed to Git
* API input validation
* CloudWatch logging
* Separate development and production configuration

---

# Monitoring

CloudWatch can be used to monitor:

* Lambda errors
* Lambda execution duration
* API errors
* Ticket-processing failures
* AI processing failures

Example:

```bash
aws logs tail /aws/lambda/analyze-ticket \
  --follow
```

---

# Future Improvements

Planned improvements include:

* Authentication with Amazon Cognito
* Agent dashboard
* WebSocket notifications
* Knowledge-base retrieval
* RAG-based response generation
* Automatic SLA monitoring
* Ticket analytics
* Human feedback loop for AI classification
* CI/CD deployment
* Infrastructure testing

---

# Learning Objectives

This project demonstrates practical experience with:

* Python
* REST API development
* Serverless architecture
* AWS Lambda
* API Gateway
* DynamoDB
* Amazon Bedrock
* CloudFormation
* AWS CLI
* Event-driven architecture
* Infrastructure as Code
* AI-assisted automation
* Logging and monitoring
* Automated testing

---

# Disclaimer

This repository is an educational/portfolio implementation of an AI-assisted customer support workflow.

AI-generated classifications and responses should be reviewed by a human before being used in customer-facing communication.

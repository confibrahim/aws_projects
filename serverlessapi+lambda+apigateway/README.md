# AWS Serverless REST API (Lambda, API Gateway, DynamoDB)

A serverless REST API backend built on AWS that receives HTTP requests, processes JSON payloads using a Python Lambda function, and writes structured records into a DynamoDB table.

## Architecture Overview

Traffic flows through the following path:
`Client Request (Postman/cURL)` -> `API Gateway (REST)` -> `AWS Lambda (Python)` -> `Amazon DynamoDB`

* **API Gateway:** Exposes HTTPS endpoints and handles request routing to backend services.
* **AWS Lambda:** Python execution environment that parses JSON payloads, generates unique UUID keys, attaches UTC timestamps, and handles errors.
* **Amazon DynamoDB:** NoSQL database storing persistent item records.
* **IAM Roles:** Execution role scoping Lambda permissions strictly to `dynamodb:PutItem` actions on the designated table.

---

## API Specifications

### `POST /prod/lar`

**Sample Request Body:**
```json
{
  "name": "Sample Payload",
  "status": "active"
}

Sample Response (201 Created):
JSON

{
  "message": "Data saved successfully",
  "data": {
    "id": "e9b7441c-7ac2-49f1-a8ae-3479687e1bd7",
    "timestamp": "2026-08-24T16:17:56.943444+00:00",
    "payload": {
      "name": "Sample Payload",
      "status": "active"
    }
  }
}

Troubleshooting & Resolution Log
Issue: 403 Forbidden ({"message": "Missing Authentication Token"})

    Initial Hypothesis: Assumed IAM permissions were preventing API Gateway from triggering the Lambda function or Lambda from writing to DynamoDB.

    Diagnostic Steps:

        Re-verified IAM execution policies.

        Directly tested the Python Lambda function in the AWS Console, confirming it successfully wrote test entries to DynamoDB.

        Ran a verbose request trace using curl -v to inspect HTTP handshake details:
        Bash

    curl -v [https://2r7ajgikcg.execute-api.eu-west-2.amazonaws.com/prod](https://2r7ajgikcg.execute-api.eu-west-2.amazonaws.com/prod)

Root Cause: Requests were hitting the base stage URL (/prod) rather than the target resource path (/prod/lar). AWS API Gateway defaults to returning a 403 Missing Authentication Token error when calling an unmapped path or method on a stage, rather than returning a standard 404 Not Found.

Resolution: Appended the resource path (/lar) to the invocation URL in Postman and cURL requests, yielding successful 201 Created responses.

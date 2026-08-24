# AWS S3 + CloudFront Static Website Deployment

A secure, static website hosting architecture built on AWS and integrated with Cloudflare for custom domain management and SSL termination.

## Architecture Overview

Traffic flows through the following path:
`User Browser` -> `Cloudflare DNS (CNAME)` -> `AWS CloudFront (CDN)` -> `AWS S3 (Private Origin)`

* **Storage:** Amazon S3 bucket hosting static site assets. Direct public web access to the bucket is blocked.
* **CDN:** Amazon CloudFront caches content at edge locations to improve global access latency. Access to S3 is authorized exclusively via **Origin Access Control (OAC)**.
* **Security & Encryption:** HTTPS enforced using custom SSL/TLS certificates provisioned in AWS Certificate Manager (ACM).
* **DNS:** Domain routing and DNS management handled via Cloudflare.

---

## Configuration & Security Policies

### S3 Bucket Policy (OAC Access Only)
The S3 bucket restricts direct access and only accepts traffic signed by the dedicated CloudFront distribution:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipalReadOnly",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::s3-conf-ibrahim/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::167217327485:distribution/E1DP3ZTLI40KKK"
        }
      }
    }
  ]
}

Troubleshooting & Resolution Log
Issue 1: 403 AccessDenied XML Error on S3 Bucket Access

    Root Cause: Initial bucket policy scoped the resource path incorrectly (.../index.html/*), preventing matching on root or sub-resource requests. Direct public access was also blocked while CloudFront OAC was not fully enforcing signed requests.

    Resolution: Re-scoped the S3 policy resource target to arn:aws:s3:::s3-conf-ibrahim/* and removed redundant public read statements. Direct S3 public access remains blocked in favor of OAC.

Issue 2: Custom Domain Routing & SSL Pending Status

    Root Cause: Requests using custom domain names failed to route properly through CloudFront edge nodes, and ACM certificates remained in Pending Validation.

    Resolution: Added Alternate Domain Names (CNAMEs) to the CloudFront distribution, updated Cloudflare DNS records to point to AWS nameservers/edge targets, and added the ACM validation CNAME record into Cloudflare DNS to issue the SSL certificate.

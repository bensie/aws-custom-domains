# Vanity

This project aims to provide repeatable a infrastructure template for a web service to point any user-owned domain or subdomain at your application.

It runs entirely and securely in your own AWS account using only services that are 100% managed by AWS. In most cases it costs just a few pennies each month to run this service.

## Assumptions

* Your backend already provides account-level subdomains on your app's domain (user.mycoolapp.com). Vanity will point at this hostname when creating a new custom domain and requests will re-write the `HOST` header to match the backend subdomain, ensuring minimal web server configuration changes.
* Your backend is served with HTTPS and you want custom domains to only be served over HTTPS. HTTP is not supported.
* You have an AWS account. **I recommend creating a new, separate AWS account just for this service for security reasons and so you don't fill your main account with customer-specific data**. Permissions are tightly scoped where possible, but the IAM role granted to the Lambda functions has broad access to CloudFront distributions, Certificate Manager certificates, and Route 53 domains within your account.

## Dependencies

With a few clicks and the default settings, your copy of this service can be up and running in about 3 minutes. Know that some AWS services will incur usage costs (it's minimal though, pennies per month in most cases).

The following AWS services are used:

* API Gateway
* CloudFront
* CloudFormation
* Certificate Manager
* DynamoDB
* IAM
* Lambda
* Route 53
* SES
* SNS
* Step Functions

## Installation

Click the "Launch Stack" to bootstrap everything you need in the us-east-1 (N. Virginia) region.

[![Launch stack in us-east-1](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=custom-domains&templateURL=https://s3.amazonaws.com/vanity-custom-domains/v1.0.0-beta30/cloudformation/stack.yml)

After creating your CloudFormation stack, the "Outputs" section of the Stack Detail contains the API Gateway base URL (called CustomDomainsAPIURL) for your custom domain service. It looks something like `https://xxxx.execute-api.us-east-1.amazonaws.com/v1/custom_domains`

## Usage

This service provides 3 JSON API endpoints for management of custom domains. You can create a custom domain, fetch the status of a custom domain, and delete a custom domain.

For each action, you must supply the custom domain in the path. A `POST` request to this endpoint creates a new custom domain, a `GET` request fetches info about the existing custom domain, and a `DELETE` request removes the custom domain.

```
https://xxxx.execute-api.us-east-1.amazonaws.com/v1/custom_domains/my.customdomain.com
```

### Create a custom domain

To create a new custom domain, you must supply the `origin_domain_name` as an attribute in a JSON request body:

#### Request

```
curl -X "POST" "https://xxxx.execute-api.us-east-1.amazonaws.com/v1/custom_domains/my.customdomain.com" \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "origin_domain_name": "user.mycoolapp.com"
}'
```

#### Response

```
HTTP/1.1 201 Created
Content-Type: application/json
...

{"domain_name":"my.customdomain.com","origin_domain_name":"usersubdomain.myapp.com","setup_started_at":1513277524474,"setup_verified_at":null,"setup_verification_failed_at":null,"delete_started_at":null,"nameservers":null,"route53_hosted_zone_created_at":null,"route53_hosted_zone_id":null,"nameserver_delegation_verified_at":null,"ses_domain_identity_created_at":null,"ses_domain_identity_verified_at":null,"ses_domain_dkim_verified_at":null,"acm_certificate_arn":null,"acm_certificate_verified_at":null,"cloudfront_distribution_id":null,"cloudfront_distribution_domain_name":null,"cloudfront_distribution_verified_at":null}
```

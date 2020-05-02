# Helm Learning

Project to learn how to get multiple services deployed to AWS EKS on different paths.

## What I Am Trying to Learn

It's easy enough to get two separate external addresses for the APIs exposed in an EKS
cluster. What I really want to do is expose the services on a single endpoint using the
request paths as the determination for which internal service to call. Basically I want
the end result to response to the following URLs (more information on the curls used to
test the public endpoints below):

  1. `http://<AWS_ASSIGNED_IP_ID123>/service1` pointing to **sample-app1-service**
  2. `http://<AWS_ASSIGNED_IP_ID123/service2/test` pointing to **sample-app2-service**

Note the goal to have the same assigned IP but different paths.

## How to Set up AWS EKS

The instructions followed for AWS EKS are from AWS-provided documentation:
[EKS: Getting started with the AWS Management Console](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)

## Testing Commands

### Non-Included File: aws-values.yaml

So as to not expose anything in my configuration, any sensitive or config-related stuff
from AWS should be in a file called **aws-values.yaml**. Right now, the file is expected
to be structured like this:

    EKS:
      SecurityGroups: 'MY_SECURITY_GROUP_1'
      Subnets: 'MY_SUBNET_1, MY_SUBNET_2, MY_SUBNET_3, MY_SUBNET_4'
      CertificateARN: 'MY_CERTIFICATE_ARN'

### Setup/Teardown

I am constantly installing/deleting these to figure all this out, so I run these two commands:

    helm install testing . -f aws-values.yaml
    helm delete testing

### Getting the Public Endpoints

Basic endpoints snags for the services:

    kubectl get services

From AWS you should get an *EXTERNAL-IP* header that looks like:

    00000000000000000000000000000000-000000000.us-east-1.elb.amazonaws.com

| Service                      | Docker Project                                                              | Test Curl                              |
| ---------------------------- | --------------------------------------------------------------------------- | -------------------------------------- |
| sample-app1-service-external | [vad1mo/hello-world-rest](https://hub.docker.com/r/vad1mo/hello-world-rest) | `curl -X GET 'http://<AWS_URL_1>'`     |
| sample-app2-service-external | [chentex/go-rest-api](https://hub.docker.com/r/chentex/go-rest-api)         | `curl -X GET 'http://<AWS_URL_2>/test` |

## Template Files

| Working            | File                              | What It Is *Supposed* To Do                                                                      |
| ------------------ | --------------------------------- | ------------------------------------------------------------------------------------------------ |
| :no_entry_sign:    | ingress.yaml                      | Exposes *sample-app1-service* and *sample-app2-service* via a single IP on different paths       |
| :heavy_check_mark: | sample-app1-deployment.yaml       | Basic deployment for [vad1mo/hello-world-rest](https://hub.docker.com/r/vad1mo/hello-world-rest) |
| :heavy_check_mark: | sample-app1-service-external.yaml | Exposes *sample-app1-deployment.yaml*'s deployment to the world                                  |
| :heavy_check_mark: | sample-app1-service-internal.yaml | Exposes *sample-app1-deployment.yaml*'s deployment to internal services                          |
| :heavy_check_mark: | sample-app2-deployment.yaml       | Basic deployment for [chentex/go-rest-api](https://hub.docker.com/r/chentex/go-rest-api)         |
| :heavy_check_mark: | sample-app2-service-external.yaml | Exposes *sample-app2-deployment.yaml*'s deployment to the world                                  |
| :heavy_check_mark: | sample-app2-service-internal.yaml | Exposes *sample-app2-deployment.yaml*'s deployment to internal services                          |

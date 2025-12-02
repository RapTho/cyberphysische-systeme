# Overview

## Link to documentation

The workshop is published at [https://raptho.github.io/cyberphysische-systeme](https://raptho.github.io/cyberphysische-systeme)

## Course content

This course will enable you to:

- Build your own mosquitto container image
- Deploy the mosquitto broker on IBM Code Engine
- Develop a publisher app in Python
- Develop and deploy your subscriber app in Python
- Persist messages via the subscriber app in an IBM Cloudant database
- Query data from the IBM Cloudant database

## Architecture

![Architecture Diagram](./files/architecture.svg)

## Sequence Diagram

![Sequence-diagram](./files/sequence-diagram.svg)

## Table of Contents

0. [Fundamentals](./0_Theory.md)
1. [Prerequisites](./1_Prerequisites.md)
2. [Building a Container Image](./2_Containerization.md)
3. [Deploying on IBM Cloud](./3_Deploy_on_IBM-Cloud.md)
4. [Publisher App](./4_publisher.md)
5. [Subscriber App](./5_subscriber.md)
6. [API Test Client](./6_API-client.md)
7. [Cloudant Database](./7_Cloudant.md)

## Windows-specific guide

See the [Windows Environment Setup Guide](./8_Windows-Environment-Setup.md) for detailed instructions on environment variables and command-line differences.

# Welcome to CAPI Gateway Documentation

[![CAPI-LB](https://github.com/surisoft-io/capi-lb/actions/workflows/main.yml/badge.svg)](https://github.com/rodrigoserracoelho/capi-lb/actions/workflows/main.yml)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
![Docker Image Version (latest by date)](https://img.shields.io/docker/v/surisoft/capi)

## What is CAPI Gateway

CAPI, like any other API gateway, sits between a client and a collection of services.

CAPI supports features such as: routing, user authentication, throttling, limiting, policies, error handling, load balancing, failover and statistics.

## Key Features
* Light API Gateway powered by Apache Camel dynamics routes.
* Support for [Multiple Oauth2 providers](oauth2.md). 
* Support for Open Policy Agent Rego (`raygo`)
* Websocket support
* Distributed tracing system (Open Telemetry Collector / Zipkin)
* Metrics (Prometheus)
* Load Balancer (Round robin)
* Reverse Proxy
* Failover (With and without Round Robin)
* Multi Tenant support
* Stick Session (Cookies and Headers)
* Certificate Manager (using the CAPI Metrics API)
* Supports Hashicorp Consul for service discovery
* Supports Hashicorp Consul KV Store for distributed runtime key values.

[Let us know if you find any issue with the documentation](https://github.com/surisoft-io/capi-lb/issues/new?assignees=&labels=use+case&template=use_case.md&title=%5BDOCS%5D+){ .md-button }
# Consul Integration

## How to make your service discovered by Consul and CAPI?

CAPI integrates with Hashicorp Consul (see Configuration, to find how to enable).
The following example uses a dummy service implemented with Spring Boot.

There are many strategies of making your service to be discovered by Consul, but for simplicity we will just mention 2:
* Install Hashicorp Consul agent on your Kubernetes cluster: (https://developer.hashicorp.com/consul/docs/k8s)
* Enable Consul auto discovery on your Spring Boot project. 



## Let's use the second for our example

Make sure to include the following dependencies in your project:
```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-consul</artifactId>
  <version>4.0.2</version>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-consul-discovery</artifactId>
  <version>4.0.2</version>
</dependency>
```

Now that Consul is enabled, you will need to tell Consul about your service, using your application file:
```yaml
server
  port: 8080
spring:
  application:
    name: dummy
  cloud:
    consul:
      enabled: true
      port: 8500
      host: http://localhost
      discovery:
        instance-id: dummy-localhost-${server.port}
        instance-group: dev
        scheme: http
        hostname: localhost
        port: ${server.port}
        tags: Owner=Your name, emailid=mail@domain.com
        metadata:
          group: dev
        health-check-url: http://localhost:${server.port}/actuator/health
```
As soon your service is running, will self register on Consul, and it should now be visible by CAPI.

If CAPI is running with the default configuration on your localhost, you should now be able to call your service using CAPI, like this:
```
curl http://localhost:8380/capi/dummy/dev/actuator/health
```

If you then scale up your service by having another instance running on 8081, that instance will also self register on Consul and it will be added as a new node in the "dummy" service. 

CAPI will then detect this change and add the new node to the routing mechanism. 

After you will be able to observe CAPI load balancing between your 2 nodes.

## Important Metadata info for CAPI

CAPI will always check the Consul's metadata `metadata:` to figure out how to deploy a service.

The only mandatory field in the metadata is the group (`group`). 

CAPI will use this field to determine your context.

Example for a service called `test-service`:

```yaml
metadata:
  group: prod
```
If this example your service will be exposed on `/capi/test-service/prod/`.

Here are optional metadata info that your service can send to Consul.

* `schema`: If your service is exposed on HTTPS, declare this field as: *https*. (Default: HTTP)
* `secured`: If you want CAPI to protect your service ( [See oauth2 integration](oauth2.md)), declare this field with *true*) (Default: false)
* `subscription-group`: If CAPI is proctecting your service, you may want to declare which subscribers are allowed to consume your service.
Example:
```yaml
metadata:
  secured: true
  subscription-group: webapp1, webapp2
```
In this example only JWT tokens with one of `webapp1` or `webapp2` in the subscription claim will be able to consume your service.

* `type`: If your service is a *Websocket*, declare this field as *websocket*, and CAPI will make your websocket available.  [See Websocket Support](websocket.md)).

* `keep-group`: By default, CAPI will not forward any headers related with CAPI itself, but if you need your service to know the context where your service is exposed, you can use this metadata property for CAPI to send the information on a header.
Example: Service `/capi/test-service/production`:
```yaml
metadata:
  keep-group: true
```

CAPI will forward the request to your service, with an extra-header: 
`capi-group: /test-service/production`.
 
 * `root-context`: By default, CAPI will forward all the requests to the root context of your service. If your service is listening on a specific context you need to provide this metadata info to CAPI.
 Example: If your service is listening on: `http://some-node:8080/my-context`.
 ```yaml
 metadata:
  root-context: /my-context
 ```
 Every request to CAPI on `/capi/test-service/production/getCustomer?ID=1` will be forwarded to `http://some-node:8080/my-context/getCustomer?ID=1`

 * `allowed-origins`: If you started CAPI with CORS Support (See `Configuration`), you can define which origins are allowed to consume your service.
 Example:
 ```yaml
 metadata:
  allowed-origins: http://localhost, http:domain.com 
 ```
 If you don't define any allowed origins, CAPI will allow the OPTIONS request for any Origin.

 For a complete list of available `metadata` go to [Metadata Integration](meta.md).

 ## Consul Key Value Store

 CAPI can use Consul KV API to store distributed runtime data.
 
 #### Currently supported features:
 
 * CORS Allowed Headers: CAPI instance always starts with a default set of allowed headers (used by browser preflight requests). If you want to be able to add/remove allowed headers at runtime without having to restart your instances you can use the Metrics endpoint to manage the headers, they will be added to Consul KV Store and updated on all CAPI running nodes.

#### Features in the roadmap to be supported:

* Truststore updates
* Sticky sessions alternative to Kafka.

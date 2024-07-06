# How CAPI is configured?
By default CAPI uses Hashicorp Consul to discover services, and all the services returned by Consul's catalog API will be available on CAPI.

## Namespace Configuration
You can give a namespace to CAPI, with 2 possible options:

### Option 1: Permissive
```yaml
capi:
  namespace: demo
  strict: false
```

With a name space in permissive mode, CAPI will create routes for every services discovered on Consul, except those with a different namespace in its metadata [More about Metadata here...](meta.md)


### Option 2: Strict
```yaml
capi:
  namespace: demo
  strict: true
```
With a name space in strict mode, CAPI will create routes for services discovered with the Metadata namespace set to `demo`. 



#### CAPI exposes by default the following ports:
* 8380 - Route Port: This is the main port, where your services will be exposed.
* 8381 - Metric Port: Where all the metrics (actuators) are available, includes by default: Prometheus, Health, Routes, Info (General info about CAPI runtime), Certicate Management (Trustore), Open API explorer.
* 8382 - Websocket Port (if enabled).
* 8383 - gRPC Port (if enabled / experimental).
* 8389 - Error Port, all errors are forwarded to this endpoint.

#### Traces integration are optional, so if you need, make sure to enable it.
Integrations supported:
* Zipkin
* Jaeger
* Open Telemetry Collector (otel)

#### How to enable:

```yaml
capi:
 traces:
   enabled: true
   endpoint: http://localhost:9411/api/v2/spans
```


#### CAPI discovers your Services using Consul. (in this example, CAPI will check for new/changes on Consul every 20 seconds)
#### CAPI can also use Consul Key Store for storing run time info, like CORS headers 
```yaml
capi:
 consul:
  kv:
    enabled: false
    timer:
      interval: 20000
  token:
  hosts: http://localhost:8500
    discovery:
      enabled: true
      timer:
        interval: 20000

```
#### CAPI can have its own Certificate Trust Store and not use the JVM default. If you want, you need to enable it:
```yaml
capi:
  trust:
    store:
      enabled: true
      path: /your/cacerts/path
      password: changeit
```

#### CAPI management endpoint runs on dedicated port (default `8381`):
```yaml
management:
  server:
    port: 8381
  security:
    enabled: false
  endpoint:
    camelroutes:
      enabled: true
  endpoints:
    web:
      base-path: /metrics/
      exposure:
        include: 'health,prometheus,routes,capi'
```

It is important to expose the `routes` and `capi` endpoints, since they provide important information about the run time. 

#### CAPI uses Apache Camel for the routing mechanism. 
### The "C" of CAPI actually stands Camel

By default the route context is exposed on /capi, but you can change it:
```yaml
camel:
  servlet:
    mapping:
      context-path: /capi/*
```

#### OPA Configuration
By default, OPA is disabled, to enable change the following properties:
```yaml
capi:
  opa:
    enabled: true
    endpoint: http://localhost:8181
```
For more information about OPA on CAPI, go [Here...](opa.md)

#### OAUTH2 Configuration
By default, oauth2 is disabled, to enable change the following properties:
```yaml
capi:
  oauth2:
    cookieName:
    provider:
      enabled: false
      keys:
        - http://localhost:8080/realms/realm1/protocol/openid-connect/certs
        - http://localhost:8080/realms/realm2/protocol/openid-connect/certs
```
For more information about oauth2 on CAPI, go [Here...](oauth2.md)
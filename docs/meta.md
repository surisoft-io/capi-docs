# Metadata

Service Metadata is the heart of any service exposed by CAPI.
Since CAPI doesn't provide any API for service deployment, it relies interely on the service metadata registered on Consul. (See [Consul Integration](consul.md))

Its ence the responsability of the service operations to tell CAPI how they want their service to be exposed.
Here you have a full list of all the metadata available:

| Property             | Mandatory        | Default value   | Description                                                              |
| -------------------- | ---------------- | --------------- | ------------------------------------------------------------------------ |
| `root-context`       | :material-close: | null            | If provided, CAPI will forward all requests to the given context         |
| `schema`             | :material-close: | http            | If your service is listening on HTTP, you need to put `schema: https`    |
| `secured`            | :material-close: | false           | Enable global authorization. [Details...](#secured)                      |
| `group`              | :material-check: | -               | Mandatory [Details...](#group)                                                            |
| `ingress`            | :material-close: | null            | If defined CAPI will forward the traffic to the specified ingress [Details...](#ingress)  |
| `type`               | :material-close: | rest            | If null, CAPI assumes you are exposing a rest service. The other option is: `type: websocket` |
| `subscription-group` | :material-close: | null            | See [OAuth2...](oauth2.md)
| `allowed-origins`    | :material-close: | null            |
| `keep-group`         | :material-close: | false           |
| `open-api`           | :material-close: | null            |
| `opa-rego`           | :material-close: | null            |
| `namespace`          | :material-close: | null            |


??? note

    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.


# Secured
If `secured: true` CAPI will enable authorization for all the operations on your service.
If you are using Open API to define the security of your operations, CAPI will ignore them and protected even the operations that you declared as open to anonymous calls.
If you want CAPI to honor your Open API security definition ignore this field or put it as false. (Check [Open-API](#open-api))  

# Group
Group metadata is mandatory because it will define how your Service is exposed on CAPI.
Imagine you have a service running on Spring Boot, called `sample`. 
For the same service you may have multiple environments.

Imagine you are running the sample service on `dev` and `test`:

Dev instance will register on Consul with the following metadata:
```yaml
spring:
  application:
    name: sample
  cloud:
    consul:
      discovery:
        metadata:
          group: dev
```
CAPI will expose your service on: `http://localhost:8380/capi/sample/dev`

# Ingress
CAPI uses Consul Catalog to discover and expose services. If you don't specify an ingress, CAPI will try to forward requests to your service based on consul ServiceAddress and ServicePort.
Imagine your service is exposed on HTTP 192.168.2.2 on port 8080. 
If no ingress provided, CAPI will try to forward all the requests to http://192.168.2.2:8080.
```yaml
spring:
  application:
    name: sample
  cloud:
    consul:
      discovery:
        metadata:
          group: dev
          ingress: http://mysample.dev
```
With the following configuration, CAPI will ignore the IP in the ServiceAddress and ServicePort, and use the endpoint defined in the ingress.

# Open-API
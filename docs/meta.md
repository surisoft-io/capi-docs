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
| `keep-group-first`   | :material-close: | false           |

??? Note

    Below you can find a detailed description about all the metadata fields available.
    If you are not using Spring Boot (the examples used in the documentation), please refer to Hashicorp Consul API, to check how to register your service.
    In the Consul catalog, the metadata needs to apper like the example below:
    ```json
    "ServiceMeta": {
        "group": "dev",
        "ingress": "http://sample.dev",
        
    }
    ```


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
If you want CAPI to only allow valid requests for your service, you need to provide your open api endpoint.

CAPI will read your service definition, and will only allow requests to existing operations.

CAPI will also check for authorization if any of your operations defines a security requirement.
```yaml
spring:
  application:
    name: sample
  cloud:
    consul:
      discovery:
        metadata:
          open-api: http://local/sample/v3/api-docs
```

# Keep Group First
By default, when CAPI is reading your service definition, it will take the service name (sample) and your metadata.group (dev) to create the route for your service, your service will then be exposed in the following fashion:

`http://local:8380/capi/sample/dev`

If you want the route to be created with the group first, you can enable this:

```yaml
spring:
  application:
    name: sample
  cloud:
    consul:
      discovery:
        metadata:
          group: dev
          keep-group-first: true
```
If enable your service will be exposed in the following fashion

`http://local:8380/capi/dev/sample`



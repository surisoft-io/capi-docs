## OPA Authorization (REST and Websocket)
OPA is a Policy-based control for cloud native environments.
For more information about OPA: https://www.openpolicyagent.org/

CAPI uses OPA Policies.

OPA policies are expressed in a high-level declarative language called Rego. Rego (pronounced “ray-go”) is purpose-built for expressing policies over complex hierarchical data structures. For detailed information on Rego see the Policy Language documentation.

You can provide your own OPA instance or use our helm charts.
CAPI only needs to be able to access OPA API's.

To start CAPI with support for OPA, please make sure to provide the following environment properties:

```yaml
opa
  enabled: true
  endpoint: http://localhost:8181
```

Imagine the following scenario:
###  You want CAPI to only allow traffic to your service if the following conditions are met:
* Token signed by a spefic key provided by you.
* Token not expired.
* The authorized party whith a list that you control.

For these requirements lets design the following REGO.
```go
package capi.test.dev

import future.keywords.if
import future.keywords.in

default allow := false

jwks := `{
    "keys": [
        {
        "kty": "RSA",
        "e": "AQAB",
        "use": "sig",
        "kid": "test",
        "alg": "RS256",
        "n": "zYF3UBCfWxTKzkK.........."
        }
    ]
}`

clients := ["my-azp" ]

current_time = time.now_ns() / 1000000000

allow if {
	clients[_] = claims.azp
    current_time < claims.exp
}

claims := payload if {
	io.jwt.verify_rs256(input.token, jwks)
	[_, payload, _] := io.jwt.decode(input.token)
}
```

After creating this REGO you will need to publish on OPA:

```bash
curl --request PUT \
  --url http://localhost:8181/v1/policies/capi/test/dev \
  --data 'package capi.eu_search.dev

import future.keywords.if
import future.keywords.in

default allow := false

jwks := `{
    "keys": [
        {
        "kty": "RSA",
        "e": "AQAB",
        "use": "sig",
        "kid": "eucommission",
        "alg": "RS256",
        "n": "zYF3UBCfWxTKzkK-CTK--y98RFwa2uXUFXOZAr35AJ-nzfDUvEM8RaoSqFofCSjzWLvd9OWuAGv59jOgE_uLVqZjr52hs32w9YLjL6vct7lh264omqxfpblsIp-yEug8rYNYdfwyM-AR-htkurjMSTK7NmeKODlekwItv1E4u5VfSr3hf8SIq0SbqDjnaW7yrWn0N9p6B37UkPV_Cahrn5_5kPYqHm_zSaghviqQh_RjaH2B0yRSaRKzDZf4VjtlXgrd3AoWxwrkmcKDWy0_nQhlcK2zTNCuu0stInbtJ79EFUKJkAOUhuoZGHuivnXDVGssZpTzNPe54-ajWthEqw"
        }
    ]
}`

clients := ["pAf3YdVriLyTR5r84dvMGL0Cc8Ua" ]

current_time = time.now_ns() / 1000000000

allow if {
	clients[_] = claims.azp
    current_time < claims.exp
}

claims := payload if {
	io.jwt.verify_rs256(input.token, jwks)
	[_, payload, _] := io.jwt.decode(input.token)
}'
```

You should be ready to protect your service using OPA.
As always, you will need to register your service on Consul, so CAPI can discover. 
Here is a sample metadata for your service (Spring Boot using Consul Starter).

```yaml
spring:
  application:
    name: test
  cloud:
    consul:
      enabled: true
      port: 8500
      host: http://localhost
      discovery:
        instance-id: ${info.app.environment}-localhost-${server.port}
        instance-group: ${info.app.environment}
        scheme: http
        hostname: localhost
        port: 8080
        metadata:
          group: dev
          secured: true
          opa-rego: capi/test/dev
        health-check-url: http://localhost:${server.port}/actuator/health
```




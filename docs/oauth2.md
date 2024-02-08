# OAuth2 Integration

There are 2 ways to work with Authorization on CAPI.
* You have control on your oauth2 provider/s, and you are able to manage your token claims, so they can be interpreted by CAPI. In this case you only need to provide CAPI with the Public Keys endpoint of your oauth2 providers (CAPI supports multiple providers) using the property `oauth2.provider.keys`.

```yaml
#example
oauth2:
  provider:
    enabled: false
    keys:
      - http://localhost:8080/realms/realm1/protocol/openid-connect/certs
      - http://localhost:8080/realms/realm2/protocol/openid-connect/certs
```

* Use Keycloak! In this case, CAPI provides manager endpoints (see `Manager`) to create roles and groups directly on Keycloak. In this case you need the following properties:
```yaml
oauth2:
  provider:
    enabled: true
    keys:
      - http://localhost:8080/realms/realm1/protocol/openid-connect/certs
    host: http://localhost:8080
    realm: /realms/your-realm
    clientId: <a client with realm management permissions>
    clientSecret: <the secret used by CAPI to authenticate to Keycloak>
```
### After having Authorization enabled, and if your service is protected, CAPI will only route traffic to your service if the following conditions are met:
* The token was signed by the oauth2 provider configured: `oauth2.provider.keys`.
* The token is not expired.
* The token azp (authorized party) has a role with the same as your service.

Example for a service called `sample`, available on `/capi/sample/dev`:

```json
{
  "exp": 1695021769,
  "iat": 1695021709,
  "iss": "http://localhost:8080/realms/master",
  "azp": "example-client",
  "realm_access": {
    "roles": [
      "test-service:dev",
      "default-roles-master",
      "offline_access",
      "uma_authorization"
    ]
  },
  "client_id": "example-client"
}
```
* If the third step fails, CAPI will check if the claim `subscription` is present and if so, if any group in the subscription list matches the subscriptions-groups of your service. `see Service.serviceMeta.subscription-groups`
Example for a subscription group called `webapp1`:
```json
{
  "exp": 1695022079,
  "iat": 1695022019,
  "iss": "http://localhost:8080/realms/master",
  "azp": "example-client",
  "subscriptions": [
    "/webapp1"
  ],
  "client_id": "example-client"
}
```

## Protect your Service (REST and Websocket)
If you want CAPI to perform authorization before routing the traffic to your Service, you will have to do the following:
* Enable Authorization (See `Authorization`)
* Declare your Service as protected (`Service.serviceMeta.secured`) (This is performed during route creation) (See `Consul Integration`)
* When you declare your Service, you can specify a list of groups (subscriptions) allowed to consume your api. (`Service.serviceMeta.subscriptionGroup`)




# oauth2 Integration

There are 2 ways to work with Authorization on CAPI.

Here we will talk about simple token validation.

* You have control on your oauth2 provider/s, and you are able to manage your token claims, so they can be interpreted by CAPI. In this case you only need to provide CAPI with the Public Keys endpoint of your oauth2 providers (CAPI supports multiple providers) using the property `oauth2.provider.keys`.

```yaml
#example
oauth2:
  provider:
    enabled: true
    keys:
      - http://localhost:8080/realms/realm1/protocol/openid-connect/certs
      - http://localhost:8080/realms/realm2/protocol/openid-connect/certs
```

CAPI works by looking for the `subscriptions` JWT Token Claim, so you will need to make sure that you can add this custom claim to your authorization provider. 


### After having Authorization enabled, and if your service is protected, CAPI will only route traffic to your service if the following conditions are met:
* The token was signed by the oauth2 provider configured: `oauth2.provider.keys`.
* The token is not expired.
* The token azp (authorized party) has a group in the `subscriptions` clain that matchs one the subscription groups in your service metadata (Check [Metadata](#meta)).

Example for a service called `sample`, available on `/capi/sample/dev` with a `subscription-group` called `webapp1`:

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

## To resume
If you want CAPI to perform authorization before routing the traffic to your Service, you will have to do the following:
* Enable CAPI Authorization (Check [Config](#config)).
* Declare your Service as protected (`Service.serviceMeta.secured`) (This is performed during route creation)
* When you declare your Service, you can specify a list of groups (`subscription-group`) allowed to consume your api. (`Service.serviceMeta.subscriptionGroup`)




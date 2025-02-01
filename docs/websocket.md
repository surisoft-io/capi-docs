## CAPI Websocket Support.
You can have CAPI acting as a Websocket Gateway.
The main features of the Websocket Support are:
* Reverse Proxy (Hide your websocket server endpoint)
* Authorization (Supports JWT Access tokens)
* Load Balancing. (Distribute the traffic to your websocket server nodes)

Websocket is disabled by default, to enable, just run CAPI with the following configuration:
```
  websocket:
    enabled: true
    server:
      host: localhost
      port: 8382      
```

Make sure that your CAPI instance is running in one of the following modes:
Full mode means CAPI will search (Consul) for all type of enabled services:
```
capi:
   mode: full
```
Websocket mode means CAPI will only search (Consul) for websocket services.:
```
capi:
   mode: websocket
```

With the following configuration, CAPI will be listening for Websocket requests on localhost port 8382.

#### Important information regarding Websockets.
CAPI will only look into the initial HTTP request, for authorization if needed. After the protocol update, you should manage the connection between your websocket client and your websocket server.
If your client or server drops the connection, you will need to start a new request.

### Example of a happy path using an anonymous (unprotected) web native connection request to CAPI.
```
websocket: WebSocket | undefined;
endpoint: string = "ws://localhost:8381/capi/your-websocket-server/your-version/your-path";
this.websocket = new WebSocket(this.endpoint);
this.websocket.onopen = (event: any) => {
   console.log("Connected to Your Websocket Server via CAPI");
}
```
### Example of a happy path using a protected web native connection request to CAPI. An access token needs to be sent.
```
websocket: WebSocket | undefined;
endpoint: string = "ws://localhost:8381/capi/your-websocket-server/your-version/your-path?access_token=<your JWT access token>";
this.websocket = new WebSocket(this.endpoint);
this.websocket.onopen = (event: any) => {
   console.log("Connected to Your Websocket Server via CAPI");
}
```
#### For authentication CAPI supports the standard Authorization header, or a query parameter with the key `access_token`.
*Important info* about Authorization: CAPI actively supports Keycloak as an oauth2 provider, but you should still be able to use any oauth2 compliant provider. See `Authorization` section to know how CAPI authorizes a request.

#### For CAPI to know that your API is a Websocket, please set `Service.serviceMeta.type` to `websocket`.
(Check [Consul Integration](#meta))


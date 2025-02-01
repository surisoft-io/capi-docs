## CAPI property list

````yaml
capi:
  namespace: local
  mode: full
  strict: false
  public-endpoint: http://localhost:8380/capi/
  
  reverse:
    proxy:
      enabled: false
      host: localhost:8380
  
  # DO NOT CHANGE, unless you want CAPI to follow Redirects.
  # If you want to disable routes from following redirects
  disable:
    redirect: true
  
  # Websocket Gateway Integration
  websocket:
    enabled: true
    server:
      host: localhost
      port: 8382
  # SSE (Server sent events) Integration    
  sse:
    enabled: false
    server:
      port: 8383  

  # Traces Integration
  traces:
    enabled: false
    endpoint: http://localhost:9411
  
  # Consul Integration
  consul:
    kv:
      enabled: false
      host: http://localhost:8500
      token: 
      timer:
        interval: 20000
    token:
    hosts:
      http://localhost:8500
    discovery:
      enabled: true
      timer:
        interval: 20000
  
  # Certificate Management
  # If you want to enable certificate management, please provide a trust store (JKS).
  trust:
    store:
      enabled: false
      path: /some/context/path/cacerts
      password: <somepassword>
      encoded: 
  
  version: ^project.version^
  name: ^project.name^
  spring:
    version: ^project.description^
  
  gateway:
    cors:
      management:
        enabled: true
        allowed-headers:
          Origin,
          Accept,
          X-Requested-With,
          Content-Type,
          Access-Control-Request-Method,
          Access-Control-Request-Headers,
          x-referrer,
          Authorization,
          X-Csrf-Request,
          Cache-Control,
          pragma,
          X-Total-Count,
          Last-Event-ID,
          X-B3-Sampled,
          X-B3-SpanId,
          X-B3-TraceId,
          X-B3-ParentSpanId,
          Vary
    
    # Error handling
    error:
      listener:
        enabled: false
        context: /capi-error
        port: 8389
      endpoint: localhost:8380/capi-error
      ssl: false
  
  # oauth2 integration
  oauth2:
    cookieName:
    provider:
      enabled: true
      keys: http://host/jwks.json
     
  # OPA Integration   
  opa:
    enabled: false
    endpoint: http://localhost:8181
  
````
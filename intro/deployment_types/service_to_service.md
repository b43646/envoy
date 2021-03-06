# 仅服务之间

![../../images/service_to_service.svg](../../images/service_to_service.svg)

The above diagram shows the simplest Envoy deployment which uses Envoy as a communication bus for all traffic internal to a service oriented architecture (SOA). In this scenario, Envoy exposes several listeners that are used for local origin traffic as well as service to service traffic.

## 服务间 egress listener

This is the port used by applications to talk to other services in the infrastructure. For example,*http://localhost:9001*. HTTP and gRPC requests use the HTTP/1.1 *host* header or the HTTP/2*:authority* header to indicate which remote cluster the request is destined for. Envoy handles service discovery, load balancing, rate limiting, etc. depending on the details in the configuration. Services only need to know about the local Envoy and do not need to concern themselves with network topology, whether they are running in development or production, etc.

This listener supports both HTTP/1.1 or HTTP/2 depending on the capabilities of the application.

## 服务间 ingress listener

This is the port used by remote Envoys when they want to talk to the local Envoy. For example,*http://localhost:9211*. Incoming requests are routed to the local service on the configured port(s). Multiple application ports may be involved depending on application or load balancing needs (for example if the service needs both an HTTP port and a gRPC port). The local Envoy performs buffering, circuit breaking, etc. as needed.

Our default configurations use HTTP/2 for all Envoy to Envoy communication, regardless of whether the application uses HTTP/1.1 or HTTP/2 when egressing out of a local Envoy. HTTP/2 provides better performance via long lived connections and explicit reset notifications.

## 可选外部服务 egress listener

Generally, an explicit egress port is used for each external service that a local service wants to talk to. This is done because some external service SDKs do not easily support overriding the *host*header to allow for standard HTTP reverse proxy behavior. For example, *http://localhost:9250*might be allocated for connections destined for DynamoDB. Instead of using *host* routing for some external services and dedicated local port routing for others, we recommend being consistent and using local port routing for all external services.

## 服务发现集成

The recommended service to service configuration uses an external discovery service for all cluster lookups. This provides Envoy with the most detailed information possible for use when performing load balancing, statistics gathering, etc.

## 配置模板

The source distribution includes an example service to service configuration that is very similar to the version that Lyft runs in production. See [here](../../install/ref_configs.html#install-ref-configs) for more information.
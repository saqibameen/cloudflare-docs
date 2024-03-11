---
title: Get started
pcx_content_type: get-started
weight: 3
---

# Get started

Privacy Gateway implementation consists of three main parts:

1. Application Gateway Server/backend configuration (operated by you).
2. Client configuration (operated by you).
3. Connection to a Privacy Gateway Relay Server (operated by Cloudflare).

---

## Before you begin

Privacy Gateway is currently in closed beta. If you are interested, [contact us](https://www.cloudflare.com/lp/privacy-edge/).

---

## Step 1 - Configure your server

As a customer of the Privacy Gateway, you also need to add server support for OHTTP by implementing an application gateway server. The application gateway is responsible for decrypting incoming requests, forwarding the inner requests to their destination, and encrypting the corresponding response back to the client.

The [server implementation](#resources) will handle incoming requests and produce responses, and it will also advertise its public key configuration for clients to access. The public key configuration is generated securely and made available via an API. Refer to the [README](https://github.com/cloudflare/privacy-gateway-server-go#readme) for details about configuration.

Applications can also implement this functionality themselves. Details about [public key configuration](https://datatracker.ietf.org/doc/html/draft-ietf-ohai-ohttp-05#section-3), HTTP message [encryption and decryption](https://datatracker.ietf.org/doc/html/draft-ietf-ohai-ohttp-05#section-4), and [server-specific details](https://datatracker.ietf.org/doc/html/draft-ietf-ohai-ohttp-05#section-5) can be found in the OHTTP specification.  

### Resources

Use the following resources for help with server configuration:

- **Go**:
    - [Sample gateway server](https://github.com/cloudflare/privacy-gateway-server-go)
    - [Gateway library](https://github.com/chris-wood/ohttp-go)
- **Rust**: [Gateway library](https://github.com/martinthomson/ohttp/tree/main/ohttp-server)
- **JavaScript / TypeScript**: [Gateway library](https://github.com/chris-wood/ohttp-js)

---

## Step 2 - Configure your client

As a customer of the Privacy Gateway, you need to set up client-side support for the gateway. Clients are responsible for encrypting requests, sending them to the Cloudflare Privacy Gateway, and then decrypting the corresponding responses.

Additionally, app developers need to [configure the client](#resources-1) to fetch or otherwise discover the gateway’s public key configuration. How this is done depends on how the gateway makes its public key configuration available. If you need help with this configuration, [contact us](https://www.cloudflare.com/lp/privacy-edge/).

### Resources

Use the following resources for help with client configuration:

- **Objective C**: [Sample application](https://github.com/cloudflare/privacy-gateway-client-demo)
- **Rust**: [Client library](https://github.com/martinthomson/ohttp/tree/main/ohttp-client)
- **JavaScript / TypeScript**: [Client library](https://github.com/chris-wood/ohttp-js)

---

## Step 3 - Review your application

After you have configured your client and server, review your application to make sure you are only sending intended data to Cloudflare and the application backend. In particular, application data should not contain anything unique to an end-user, as this would invalidate the benefits that OHTTP provides.

- Applications should scrub identifying user data from requests forwarded through the Privacy Gateway. This includes, for example, names, email addresses, phone numbers, etc.
- Applications should encourage users to disable crash reporting when using Privacy Gateway. Crash reports can contain sensitive user information and data, including email addresses.
- Where possible, application data should be encrypted on the client device with a key known only to the client. For example, iOS generally has good support for [client-side encryption (and key synchronization via the KeyChain)](https://developer.apple.com/documentation/security/certificate_key_and_trust_services/keys). Android likely has similar features available.

---

## Step 4 - Relay requests through Cloudflare

Before sending any requests, you need to first set up your account with Cloudflare. That requires [contacting us](https://www.cloudflare.com/lp/privacy-edge/) and providing the URL of your application gateway server.

Then, make sure you are forwarding requests to a mutually agreed URL with the following conventions.

```txt
https://<APPLICATION_NAME>.privacy-gateway.cloudflare.com/
```

## Querying Privacy Gateway Metrics
Privacy Gateway now supports enhanced monitoring through our GraphQL API, providing detailed insights into your gateway traffic and performance. To access these metrics, ensure you have:
- A relay gateway proxy implementation where Cloudflare acts as the oblivious relay party.
- An API token with Analytics Read permissions.
We offer two GraphQL nodes to retrieve metrics: `ohttpMetricsAdaptive` and `ohttpMetricsAdaptiveGroups`. The first node provides comprehensive request data, while the second facilitates grouped analytics.

### ohttpMetricsAdaptive
The `ohttpMetricsAdaptive` node is designed for detailed insights into individual OHTTP requests with adaptive sampling. This node can help in understanding the performance and load on your server and client setup.

Key Arguments
- `filter`: Apply filters to narrow down your data set. `accountTag` is a required filer. 
- `limit`: Set the maximum number of records to retrieve.
- `orderBy`: Sort your data by various fields such as bytes transferred, datacenter location, time, and status codes.

Available Fields
- `bytesToClient` (uint64!): The number of bytes returned to the client.
- `bytesToGateway` (uint64!): Total bytes received from the client.
- `colo` (string!): Airport code of the Cloudflare data center that served the request.
- `datetime` (Time!): The date and time when the event was recorded.
- `gatewayStatusCode` (uint32!): Status code returned by the gateway.
- `relayStatusCode` (uint32!): Status code returned by the relay.

This node is useful for a granular view of traffic, helping you identify patterns, performance issues, or anomalies in your data flow.

### ohttpMetricsAdaptiveGroups
The `ohttpMetricsAdaptiveGroups` node allows for aggregated analysis of OHTTP request metrics with adaptive sampling. This node is particularly useful for identifying trends and patterns across different dimensions of your traffic and operations.

Key Arguments
- `filter`: Apply filters to narrow down your data set. `accountTag` is a required filer. 
- `limit`: Specify the maximum number of records to return.
- `orderBy`: Choose how to sort your data, with options for various dimensions and metrics.

Available Fields
- `count`: (uint64!): The number of records that meet the criteria.
- `dimensions`: Specifies the grouping dimensions for your data.
- `sum`: Aggregated totals for various metrics, per dimension.

**Dimensions**
You can group your metrics by various dimensions to get a more segmented view of your data:

- `colo` (string!): The airport code of the Cloudflare data center.
- `date` (Date!): The date of OHTTP request metrics.
- `datetimeFifteenMinutes` (Time!): Timestamp truncated to fifteen minutes.
- `datetimeFiveMinutes` (Time!): Timestamp truncated to five minutes.
- `datetimeHour` (Time!): Timestamp truncated to the hour.
- `datetimeMinute` (Time!): Timestamp truncated to the minute.
- `endpoint` (string!): The appId that generated traffic.
- `gatewayStatusCode` (string!): Status code returned by the gateway.
- `relayStatusCode` (int64!): Status code returned by the relay.
**Sum Fields**
Sum fields offer a cumulative view of various metrics over your selected time period:

- `bytesToClient` (int64!): Total bytes sent from the gateway to the client.
- `bytesToGateway` (int64!): Total bytes from the client to the gateway.
- `clientRequestErrors` (int64!): Total number of client request errors.
- `gatewayResponseErrors` (int64!): Total number of gateway response errors.

Utilize the ohttpMetricsAdaptiveGroups node to gain comprehensive, aggregated insights into your traffic patterns, helping you optimize performance and user experience.
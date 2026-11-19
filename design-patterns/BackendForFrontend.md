# Backend for Frontend (BFF)

## What is a BFF?

**Backend for Frontend (BFF)** is an architectural pattern in which a backend API is shaped for a particular user interface or client. A web app and a mobile app may need different response shapes, data, or interaction patterns, so each can use a BFF tailored to its needs.

A BFF is usually an orchestration and presentation layer, not a second source of truth for business data. It calls the relevant domain services and returns a response designed for its client.

```text
Web app    -> Web BFF    -> Catalog / Orders / Profile services
Mobile app -> Mobile BFF -> Catalog / Orders / Profile services
```

A team does not necessarily need a separate BFF for every client. A shared API can be sufficient when clients have similar needs; introduce client-specific BFFs when the differences justify the extra services and operational cost.

## Why use one?

- **Client-specific responses:** Return the fields and formats the client needs instead of making every client adapt a generic response.
- **Reduce over-fetching and under-fetching:** Compose a response with the right amount of data for a screen or workflow.
- **Aggregate service calls:** The BFF can call multiple backend services, often in parallel when calls are independent, and combine their results into one response. This reduces client-side orchestration and network round trips.
- **Centralize client-facing concerns:** Handle client-specific authentication integration, response shaping, and compatibility at a boundary designed for that client.
- **Support team ownership:** A frontend-aligned team can own the BFF and evolve its API with its client, while domain services remain owned by the teams responsible for business capabilities.

## Request flow example

Suppose a mobile home screen needs the signed-in user's name, recent orders, and recommended products. Without a BFF, the app may need to know about and call several services. With a BFF, it makes one request:

```text
Mobile app -> Mobile BFF
                 |-> Profile service
                 |-> Orders service
                 |-> Recommendations service
                 `-> combines a mobile-friendly response
```

The BFF can request independent data concurrently, apply timeouts, handle partial failures according to product requirements, and return a stable client-facing response. It should not silently turn a failed required service call into misleading or incomplete data.

## Security considerations

A BFF is not automatically a security boundary just because it is server-side. It should:

- Authenticate requests and authorize access to the requested user's data.
- Pass identity and authorization context to downstream services appropriately; do not trust user IDs supplied by the client without checking them.
- Keep secrets and service credentials on the server, never in the browser or mobile application.
- Validate inputs, limit request sizes, and avoid exposing internal service details or sensitive fields in responses.
- Apply rate limits and sensible timeouts, and avoid logging tokens or sensitive personal data.

A BFF should enforce client-facing access rules, while domain services should still enforce authorization for their own data and operations.

## BFF vs. API Gateway

An **API Gateway** is commonly a shared edge component for cross-cutting concerns such as routing, TLS termination, and rate limiting. A **BFF** focuses on the API and composition needs of a particular client. They can be used together: requests may pass through a gateway and then reach a client-specific BFF. In smaller systems, one component may perform both roles, but the responsibilities are still distinct.

## Trade-offs and pitfalls

- **More services to operate:** Separate BFFs add deployment, monitoring, scaling, and maintenance work.
- **Duplicated client-specific logic:** Some response mapping may exist in more than one BFF. Keep business rules in domain services rather than copying them across BFFs.
- **Extra network hop:** A BFF adds latency if it does not reduce enough client-side calls or simplify the client.
- **Unbounded fan-out:** Calling many services per request can increase latency and load. Set deadlines, bound concurrency, and consider caching only where correctness allows.
- **Accidental domain layer:** Avoid letting a BFF become the owner of core business rules or the canonical data store.
- **Too many BFFs too soon:** Start with a shared API when client needs are close; split when real differences in release cadence, payloads, or client-specific workflows make the separation valuable.

## When is a BFF a good fit?

Use the pattern when clients have substantially different data or interaction needs, client teams need to evolve independently, or a screen requires composition across several services. It may be unnecessary when there is one client, the API already fits all consumers, or a gateway and a small set of stable APIs are enough.

## Key takeaway

A BFF can make a frontend simpler by moving client-specific response shaping and service aggregation to the backend. It may introduce some duplicated **presentation/orchestration code**, but it should not duplicate the underlying business data or rules. Weigh that benefit against the additional operational and maintenance cost.

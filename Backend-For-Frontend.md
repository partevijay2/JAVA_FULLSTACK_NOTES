# Backend for Frontend (BFF)

A **Backend for Frontend (BFF)** is a dedicated backend layer designed specifically for one frontend experience, such as web or mobile.

## 10 Powerful Keywords to Remember

| Keyword | Meaning |
|---|---|
| **Client-Centric** | Built specifically to serve one UI type, such as Web or Mobile. |
| **Aggregator** | Combines data from multiple microservices into one response. |
| **Tailored** | Shapes and formats JSON precisely for the frontend's needs. |
| **Fan-Out/Fan-In** | Makes parallel calls to downstream backends and merges the results. |
| **Facade** | Hides complex microservice architecture behind a simple API. |
| **Shield** | Acts as a security barrier that protects internal systems. |
| **Token-Handler** | Secures OAuth/JWT tokens using HttpOnly cookies to help prevent XSS. |
| **Decoupled** | Allows frontend teams to iterate without breaking backend contracts. |
| **Over-fetching** | Solves the problem of downloading unnecessary data. |
| **Under-fetching** | Solves the problem of making too many client-side requests. |

## Core Purpose

A BFF provides a frontend-specific API that:

- Aggregates responses from several services.
- Formats data for the UI's exact requirements.
- Reduces over-fetching and under-fetching.
- Hides internal service topology.
- Applies security and authentication policies at a controlled boundary.

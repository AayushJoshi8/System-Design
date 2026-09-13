# E-Commerce System Design

A scalable e-commerce platform like Amazon, designed using microservices, dedicated databases, asynchronous processing, and external payment and notification providers.

## 1. Project Overview

This project focuses on designing the backend architecture of an e-commerce platform that supports product browsing, search, cart management, checkout, payments, inventory management, and order tracking.

The design covers both the high-level architecture and the deep-dive components required to build a reliable and scalable e-commerce system.

## 2. Functional Requirements

* Users can register, log in, and manage their accounts.
* Users can search products by name, title, or category.
* Users can view product details such as price, quantity, description, and reviews.
* Users can select products and add them to their cart.
* Users can checkout and place orders.
* Users can make payments securely.
* Users can track their order status.
* The system can handle multiple purchases of limited-stock items.
* The system sends notifications for order updates and payment events.

## 3. Non-Functional Requirements

* **Scalability:** Support approximately 10 million monthly active users and 10 orders per second as an initial design target.
* **Availability:** Maintain high availability for browsing and shopping operations.
* **Consistency:** Maintain strong consistency for critical operations such as order placement, inventory reservation, and payments.
* **Performance:** Aim for approximately 200 ms response time for normal operations where practical.
* **Fault tolerance:** Handle service failures, database failures, and message-processing failures.
* **Reliability:** Prevent duplicate orders, lost payment records, and incorrect inventory counts.
* **Security:** Protect user accounts, payment information, and personal data.

## 4. Core Entities

* **User:** Stores account details and user information.
* **Product:** Stores product name, price, quantity, category, description, and URLs.
* **Cart:** Stores products selected by a user before checkout.
* **Order:** Stores order ID, user ID, ordered items, total amount, status, and payment reference.
* **Checkout:** Handles the process of converting a cart into an order.
* **Payment:** Stores payment details and payment status.
* **Inventory:** Maintains the available stock for products.
* **Notification:** Stores and manages order and payment notifications.

## 5. API Design

| Method | Endpoint                 | Purpose               |
| ------ | ------------------------ | --------------------- |
| GET    | `/products`              | Get products          |
| GET    | `/products/{id}`         | Get product details   |
| GET    | `/search?q=...`          | Search products       |
| POST   | `/cart`                  | Add a product to cart |
| GET    | `/cart`                  | Get cart              |
| POST   | `/orders`                | Create an order       |
| GET    | `/orders/{id}`           | Get order details     |
| POST   | `/checkout`              | Start checkout        |
| POST   | `/payments`              | Initiate payment      |
| GET    | `/inventory/{productId}` | Check inventory       |

All APIs use the `/api/v1/` prefix for versioning.

## 6. High-Level Architecture

The system follows a microservices architecture with an API Gateway as the entry point.

### Main services

* **API Gateway:** Routes requests, performs authentication/authorization, and applies rate limiting.
* **User Service:** Manages user accounts and authentication-related information.
* **Search Service:** Handles product search using Elasticsearch.
* **Product Service:** Manages product details and product-related operations.
* **Cart Service:** Manages shopping carts.
* **Order Status Service:** Manages order status and tracking.
* **Checkout Service:** Coordinates checkout and order placement.
* **Payment Service:** Processes payments through a payment gateway.
* **Inventory Service:** Manages stock availability and reservations.
* **Notification Service:** Sends order and payment notifications.

### High-Level Flow

Client → API Gateway → Microservices → Dedicated Databases

Each service is responsible for its own business logic and data. The services communicate through APIs and asynchronous messaging where required.

## 7. Deep-Dive Architecture

### Product and Search

The Product Service stores product data in MongoDB. Product changes are captured through CDC (Change Data Capture) and used to update Elasticsearch for efficient searching.

Product images and static files are stored in Amazon S3 and delivered through a CDN.

**Flow:**

Product Service → Product DB → CDC → Elasticsearch

Product Service → S3 → CDN → Client

### Inventory Management

The Inventory Service is responsible for maintaining the source of truth for stock availability.

Redis is used for fast inventory access and coordination, while the Inventory DB stores the persistent inventory records.

During checkout, the system checks availability and uses a Redis lock to prevent conflicting stock updates.

### Checkout and Orders

The Checkout Service coordinates the checkout process.

1. Check product availability.
2. Reserve inventory.
3. Create the order.
4. Process payment.
5. Confirm the order and update inventory.
6. Publish events for notifications and other downstream services.

The Order DB stores the order details, including user ID, ordered items, total, status, and payment ID.

### Payment Processing

The Payment Service communicates with an external payment gateway.

The payment database stores payment records and their statuses. The payment gateway handles the actual payment transaction.

Payment status updates should be processed safely to avoid duplicate charges or inconsistent order states.

### Asynchronous Processing

Kafka is used for asynchronous communication between services.

For example:

Order Service → Kafka → Order Consumer → Inventory Consumer / Notification Service

This allows order-related processing to happen independently and helps absorb traffic spikes.

### Notification Service

The Notification Service receives events such as order confirmation, payment success, and shipment updates.

It can send notifications through external providers:

* Push notifications: FCM or APNs.
* Email: SendGrid or AWS SES.
* SMS: Twilio.

## 8. Database Design

| Service                   | Database / Storage |
| ------------------------- | ------------------ |
| User Service              | MySQL              |
| Product Service           | MongoDB            |
| Search Service            | Elasticsearch      |
| Cart Service              | Cart DB            |
| Order Service             | Order DB           |
| Payment Service           | Payment DB         |
| Inventory Service         | Inventory DB       |
| Inventory caching/locking | Redis              |
| Product images            | Amazon S3          |
| Static content delivery   | CDN                |

Each service owns its data, and services should not directly modify another service's database.

## 9. Technologies Used

* Backend: Node.js, Express.js
* Databases: MySQL, MongoDB
* Search: Elasticsearch
* Cache and distributed locking: Redis
* Message broker: Apache Kafka
* Object storage: Amazon S3
* CDN: CloudFront or equivalent
* Payment gateway: External payment provider
* API style: REST APIs

## 10. Key System Design Concepts

* Microservices architecture
* API Gateway
* Database per service
* Horizontal scaling
* Caching with Redis
* Distributed locking
* Change Data Capture (CDC)
* Event-driven architecture
* Kafka consumers and producers
* Payment idempotency
* Inventory reservation
* Fault tolerance
* Service-level authentication and authorization

## 11. Future Improvements

* Add replicas for high availability.
* Add retry and dead-letter queues for failed events.
* Add monitoring, logging, and distributed tracing.
* Add a dedicated user authentication service.
* Improve inventory reservation with expiry and recovery.
* Add an order state machine.
* Add rate limiting and circuit breakers.
* Add automated testing and deployment pipelines.

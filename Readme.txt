Bazaar Technologies Inventory Tracking System
=============================================

Overview
--------
This project implements an Inventory Tracking System for Bazaar Technologies, evolving from a single-store solution (Stage 1) to a scalable, multi-store system (Stage 3). The system supports product inventory management, store-specific stock tracking, audit logs, and real-time updates, meeting the case study requirements across three stages.

Design Decisions
----------------
Stage 1 (V1):
- Database: SQLite was chosen for its simplicity and lightweight nature, ideal for a single-store setup with no server requirements.
- Model: A single Product table (id, name, sku, quantity) to track inventory.
- API: RESTful endpoints (Express.js) over CLI for flexibility and future scalability.
- Reasoning: Prioritized ease of setup and minimal overhead for a small-scale system.

Stage 2 (V2):
- Database: Switched to PostgreSQL for relational integrity and scalability to support 500+ stores.
- Models: Added Store, Category, Product, and InventoryLog to enable store-specific tracking and auditing.
- Security: Implemented JWT authentication to restrict access to store owners.
- Rate Limiting: Added express-rate-limit (100 requests/min) to prevent API abuse.
- Reasoning: Focused on multi-store support, security, and auditability for a growing system.

Stage 3 (V3):
- Database: Enhanced PostgreSQL with read/write separation (master for writes, slave for reads) to handle high concurrency.
- Caching: Integrated Redis for caching inventory logs, reducing database load.
- Async Events: Used Redis Pub/Sub to publish inventory events (created, updated, deleted) for near real-time stock updates.
- Time Handling: Added moment-timezone for consistent date filtering across regions.
- Reasoning: Optimized for performance, scalability, and real-time updates under high load.

Assumptions
-----------
Stage 1 (V1):
- Single store with fewer than 100 products.
- No concurrent updates (stock changes are sequential).
- Manual removals are treated as stock-outs (same as sales).

Stage 2 (V2):
- Each store manages its own products and categories (no central catalog enforced).
- Authenticated users (store owners) can only access their own store's data.
- Stock changes must be logged for auditing purposes.

Stage 3 (V3):
- High concurrency is handled via database pooling and caching.
- Stock synchronization relies on Redis subscribers (to be fully implemented).
- Cache invalidation (e.g., on log deletion) ensures data consistency.



BASE Url: http://localhost:5000/api/v*(1,2,3)

API Design
----------
Stage 1 (V1)
Base URL: /api/v1
- POST /product/create-product
- GET /product/get-all-products
- GET /product/get-product/:sku
- PUT /product/stock-in/:sku
- PUT /product/stock-out/:sku

Stage 2 (V2)
Base URL: /api/v2
Store Endpoints:
- POST /store/create-store
- POST /store/login
Product Endpoints (JWT Required):
- POST /product/create-product/:storeId
- PUT /product/stock-in/:id
- PUT /product/stock-out/:id
- GET /product/show-products/:storeId
- GET /product/show-product/:id
- GET /product/show-product-by-category/:categoryId
- PUT /product/update-products/:id
Inventory Log Endpoints (JWT Required):
- GET /logs/show-logs/:storeId
- DELETE /logs/delete-log/:id
Category Endpoints (JWT Required)
- POST /category/create-category/:storeId
- PUT /category/update-category/:id
- DELETE /category/delete-category/:id
- GET /category/show-categories/:storeId
- GET /category/show-category/:id


Stage 3 (V3)
Base URL: /api/v2 (same as V2, with enhancements)
Updated Endpoints (JWT Required):
- GET /logs/show-logs/:storeId
  Purpose: Fetch logs with Redis caching.
  Query: ?startDate=2025-04-01&endDate=2025-04-04
  Response: [{ "id": "uuid", "changeType": "IN", "quantity": 5, ... }, ...]
  Note: Checks Redis cache first, falls back to DB if cache miss.
- DELETE /logs/delete-log/:id
  Purpose: Delete a log and clear cache.
  Response: { "message": "Log deleted successfully" }
Async Events: InventoryLog changes (created, updated, deleted) are published to inventory_events channel via Redis Pub/Sub.

Evolution Rationale (V1 -> V3)
-----------------------------
V1 to V2:
- Why Evolve?: Stage 1 supported a single store but couldn't handle 500+ stores, lacked authentication, and had no auditing.
- Changes:
  - Switched from SQLite to PostgreSQL for relational data and scalability.
  - Added Store, Category, and InventoryLog models to support multiple stores and auditing.
  - Introduced JWT authentication for security.
  - Added rate limiting to handle increased API usage.
- Impact: Enabled multi-store support, secure access, and stock change tracking.

V2 to V3:
- Why Evolve?: Stage 2 handled multiple stores but struggled with high concurrency, lacked real-time updates, and had performance bottlenecks for frequent log queries.
- Changes:
  - Implemented PostgreSQL read/write separation to scale reads and writes independently.
  - Integrated Redis for caching inventory logs, reducing database load.
  - Added Redis Pub/Sub for async event publishing, enabling near real-time stock sync.
  - Used moment-timezone for consistent date handling in log filtering.
- Impact: Improved performance, scalability, and real-time capabilities for thousands of stores.

Setup Instructions

Stage 1 GIT repo url: https://github.com/MuhammadOwais03/CSC_Inventory_tracking_system.git
Stage 2 GIT repo url: https://github.com/MuhammadOwais03/Inventory_Tracking_System.git 
Stage 3 GIT repo url: https://github.com/MuhammadOwais03/Inventory_Tracking_System_S3.git
------------------
1. Clone the Repository:
   git clone <repository-url>
   cd <repository-name>

2. Install Dependencies:
   npm install

3. Environment Variables:
    Create a .env file with:
    PORT = 5000
    DB_NAME=inventory_db_1
    DB_USER=postgres
    DB_PASSWORD=owais
    DB_MASTER_HOST=127.0.0.1
    DB_MASTER_PORT=5432
    DB_SLAVE_HOST=127.0.0.1
    DB_SLAVE_PORT=5433


ACCESS_SECRET_KEY = kasjdlkasjlkajslkasjdlksajdlkajsdlk
ACCESS_EXPIRES = 1d

4. Run the Application:
   npm run dev

Dependencies:
- bcrypt: "^5.1.1" (Password hashing for store authentication)
- bcryptjs: "^3.0.2" (Alternative bcrypt implementation)
- cookie-parser: "^1.4.7" (Parse cookies for JWT)
- cors: "^2.8.5" (Enable CORS for API access)
- dotenv: "^16.4.5" (Environment variable management)
- express: "^4.20.0" (API framework)
- express-rate-limit: "^7.5.0" (Rate limiting for API)
- helmet: "^8.1.0" (Security headers for Express)
- jsonwebtoken: "^9.0.2" (JWT authentication)
- moment-timezone: "^0.5.48" (Date handling for log filtering, V3)
- pg: "^8.14.1" (PostgreSQL driver, V2-V3)
- pg-hstore: "^2.3.4" (PostgreSQL hstore support for Sequelize)
- sequelize: "^6.37.7" (ORM for SQLite/PostgreSQL)
- sequelize-cli: "^6.6.2" (Sequelize migrations and CLI)
Note: Redis is used in V3 for caching and Pub/Sub but is not listed in package.json; ensure it is installed (e.g., "redis": "^4.6.7").
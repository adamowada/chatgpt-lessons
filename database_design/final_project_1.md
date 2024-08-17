### Final Project: Designing a Database System for an Online Marketplace

#### **Objective:**

Apply everything learned to design a database system for an online marketplace. This marketplace allows users to buy and sell products, manage orders, and leave reviews.

---

### **1. Gather Requirements Based on a Use Case**

#### **Functional Requirements:**

1. **User Management:**

   - Users must be able to create an account, log in, and manage their profile.
   - Users can act as buyers or sellers.

2. **Product Management:**

   - Sellers can list products with details (e.g., name, description, price, category, stock quantity).
   - Buyers can search for products using various filters (e.g., category, price range, rating).

3. **Order Processing:**

   - Buyers can place orders for products.
   - Orders must track status (e.g., pending, shipped, delivered).
   - Payments need to be securely processed.

4. **Reviews and Ratings:**

   - Buyers can leave reviews and ratings for products.
   - Reviews should be associated with both the product and the user who posted them.

5. **Data Analytics:**
   - The system should support analytics queries, such as total sales by category, top-rated products, and user activity.

#### **Non-Functional Requirements:**

1. **Scalability:**

   - The system must handle a large number of concurrent users and a growing volume of products and orders.
   - Support for peak usage periods (e.g., holiday sales).

2. **Security:**

   - Data encryption at rest and in transit.
   - Role-based access control (RBAC) for managing user permissions.
   - Regular backups and disaster recovery plans.

3. **Performance:**

   - Fast response times for product searches and order placement.
   - Efficient processing of analytical queries.

4. **Compliance:**
   - Compliance with data protection regulations (e.g., GDPR, PCI-DSS).

---

### **2. Model the Data with an ER Diagram**

#### **Entities and Relationships:**

1. **Users:**

   - Fields: `user_id` (PK), `username`, `password_hash`, `email`, `role` (buyer, seller, admin), `created_at`
   - Relationships: Can have multiple `Reviews`, can place multiple `Orders`

2. **Products:**

   - Fields: `product_id` (PK), `name`, `description`, `price`, `category_id` (FK), `stock_quantity`, `seller_id` (FK to Users), `created_at`
   - Relationships: Belongs to one `Category`, can have multiple `Reviews`, can be part of multiple `OrderItems`

3. **Categories:**

   - Fields: `category_id` (PK), `name`
   - Relationships: Can have multiple `Products`

4. **Orders:**

   - Fields: `order_id` (PK), `buyer_id` (FK to Users), `total_amount`, `status`, `created_at`
   - Relationships: A buyer (User) can place multiple orders, an order can have multiple `OrderItems`

5. **OrderItems:**

   - Fields: `order_item_id` (PK), `order_id` (FK), `product_id` (FK), `quantity`, `price_at_purchase`
   - Relationships: Belongs to one `Order`, contains one `Product`

6. **Reviews:**
   - Fields: `review_id` (PK), `product_id` (FK), `user_id` (FK), `rating`, `comment`, `created_at`
   - Relationships: Belongs to one `User` and one `Product`

#### **ER Diagram:**

```
Users    <-- FK -->  Reviews
    \           /
     \-< FK >--/        Products  -- FK --> Reviews
          |
         FK
     Categories    Orders  <-- FK --> OrderItems
                  /
                 /-- FK --< Products
```

---

### **3. Choose Indexing Strategies Based on Expected Queries**

#### **Expected Queries and Indexing:**

1. **Product Search by Category and Price Range:**

   - **Query:** `SELECT * FROM Products WHERE category_id = ? AND price BETWEEN ? AND ?`
   - **Indexes:**
     - Index on `Products(category_id)`
     - Composite index on `Products(price, category_id)`

2. **Order Retrieval by User:**

   - **Query:** `SELECT * FROM Orders WHERE buyer_id = ?`
   - **Indexes:**
     - Index on `Orders(buyer_id)`

3. **Product Reviews:**

   - **Query:** `SELECT * FROM Reviews WHERE product_id = ?`
   - **Indexes:**
     - Index on `Reviews(product_id)`

4. **Top-Rated Products:**

   - **Query:** `SELECT product_id, AVG(rating) as avg_rating FROM Reviews GROUP BY product_id ORDER BY avg_rating DESC LIMIT ?`
   - **Indexes:**
     - Index on `Reviews(product_id)`
     - Index on `Reviews(rating)`

5. **Sales Analytics by Category:**
   - **Query:** `SELECT category_id, SUM(total_amount) as total_sales FROM Orders JOIN OrderItems ON Orders.order_id = OrderItems.order_id JOIN Products ON OrderItems.product_id = Products.product_id GROUP BY category_id`
   - **Indexes:**
     - Index on `OrderItems(order_id)`
     - Composite index on `OrderItems(product_id, order_id)`
     - Index on `Products(category_id)`

---

### **4. Address Scalability, Security, and Backup Requirements**

#### **Scalability:**

1. **Horizontal Scaling (Sharding):**

   - **Strategy:** Shard the `Users`, `Products`, and `Orders` tables across multiple database instances based on user regions or product categories.
   - **Example:** Shard `Products` by `category_id` so that each shard holds products for specific categories.

2. **Replication:**

   - **Strategy:** Use master-slave replication for read-heavy tables like `Products` and `Reviews` to distribute the load.
   - **Example:** The primary database handles writes, and read replicas handle search queries and analytics.

3. **Load Balancing:**
   - **Strategy:** Implement load balancers to distribute incoming traffic evenly across multiple database servers.
   - **Example:** Use a load balancer to distribute search queries across read replicas.

#### **Security:**

1. **Encryption:**

   - **At Rest:** Use Transparent Data Encryption (TDE) to encrypt data stored on disk.
   - **In Transit:** Use TLS to encrypt data transmitted between components.

2. **Role-Based Access Control (RBAC):**

   - **Roles:**
     - `Admin`: Full access to all data and management operations.
     - `Seller`: Access to their own products, orders, and reviews.
     - `Buyer`: Access to purchase products and write reviews.
   - **Example:**

     ```sql
     CREATE ROLE admin;
     CREATE ROLE seller;
     CREATE ROLE buyer;

     GRANT ALL PRIVILEGES ON ALL TABLES TO admin;
     GRANT SELECT, INSERT, UPDATE, DELETE ON Products, Orders, Reviews TO seller
     WHERE seller_id = CURRENT_USER;
     GRANT SELECT, INSERT ON Orders, Reviews TO buyer
     WHERE buyer_id = CURRENT_USER;

     GRANT admin TO alice;  -- Admin user
     GRANT seller TO bob;   -- A seller user
     GRANT buyer TO charlie;-- A buyer user
     ```

3. **Authentication:**
   - **Multi-Factor Authentication (MFA)** for admin and seller roles.
   - **Example:** Use OAuth for third-party login providers (e.g., Google, Facebook).

#### **Backup and Recovery:**

1. **Backup Strategy:**

   - **Full Backups:** Weekly full backups of the entire database.
   - **Incremental Backups:** Daily incremental backups capturing changes since the last full backup.
   - **Differential Backups:** Hourly differential backups capturing changes since the last full backup.

2. **Recovery Strategy:**

   - **Point-in-Time Recovery:** Enable transaction logging to allow recovery to any point in time.
   - **Disaster Recovery:** Replicate backups to an offsite location and test recovery procedures regularly.

3. **Monitoring and Alerts:**
   - **Monitoring Tools:** Use Prometheus and Grafana to monitor database performance and health.
   - **Alerts:** Set up alerts for backup failures, replication lag, high CPU usage, and slow queries.

---

### **5. Present the Design and Justify the Decisions Made**

#### **Design Summary:**

1. **Data Modeling:**

   - **ER Diagram:** Structured around key entities (`Users`, `Products`, `Orders`, `Categories`, `Reviews`) with appropriate relationships.
   - **Decision:** The star schema for simplicity and performance in analytical queries.

2. **Indexing:**

   - **Query Optimization:** Indexed critical fields based on expected queries to ensure fast search and retrieval performance.
   - **Decision:** Chosen indexes based on query patterns to balance read and write performance.

3. **Scalability:**

   - **Sharding and Replication:** Sharded to distribute load by regions and categories. Implemented master-slave replication for read-heavy operations.
   - **Decision:** Horizontal scaling ensures the system can grow with increasing user and product volumes.

4. **Security:**

   - **Encryption and RBAC:** Encrypted data at rest and in transit, and used RBAC to manage access permissions based on user roles.
   - **Decision:** Ensures data protection and adherence to data privacy regulations.

5. **Backup and Recovery:**

   - **Backup Strategy:** Combination of full, incremental, and differential backups with offsite storage for disaster recovery.
   - **Decision:** Comprehensive backup strategy ensures minimal data loss and quick recovery times in case of a failure.

6. **Monitoring and Alerts:**
   - **Tools and Alerts:** Continuous monitoring with Prometheus and Grafana, and alerts for potential issues.
   - **Decision:** Proactive monitoring and alerting to maintain system health and performance.

#### **Justification:**

- **Data Model:** Designed to handle the complexities of an online marketplace with multiple related entities while optimizing for common queries.
- **Indexing Strategy:** Focused on query performance for key operations like product search, order processing, and analytics.
- **Scalability:** Ensures that the system can handle growth in users and data, with mechanisms for distributing load and maintaining performance.
- **Security Measures:** Protects sensitive data and complies with legal requirements, ensuring the system is secure against unauthorized access.
- **Backup and Recovery:** Provides a robust framework for data protection and quick recovery, minimizing downtime and data loss.
- **Monitoring:** Allows for proactive management of system health, leading to higher availability and reliability.

This design leverages comprehensive strategies and best practices to build a scalable, secure, and resilient online marketplace database, ensuring it meets both functional and non-functional requirements.

Sure! Let's choose a **Financial Trading Platform** as the real-world scenario for the final project. This platform will allow users to trade stocks, view market data, and manage their portfolios.

### **1. Gather Requirements Based on a Use Case**

#### **1.1. Functional Requirements:**

1. **User Management:**

   - Users can create accounts, log in, update personal information, and deactivate their accounts.
   - Users can have different roles (e.g., trader, admin).

2. **Trading:**

   - Users can buy and sell stocks.
   - Users can view their transaction history and current portfolio.
   - The system should execute trades and update portfolios in real-time.

3. **Market Data:**

   - The platform should display real-time market data, including stock prices, volumes, and historical data.
   - Users can search for stocks and view detailed information about each stock.

4. **Reporting and Analytics:**

   - Generate monthly statements of account activities for users.
   - Admins can generate reports on trading volumes, user activities, and system performance.

5. **Notifications:**
   - Users receive notifications for completed trades, price alerts, and important updates.

#### **1.2. Non-Functional Requirements:**

1. **Scalability:**

   - The system should handle high volumes of trading activities, especially during market peaks.
   - Data storage should scale horizontally to handle large volumes of historical market data.

2. **Security:**

   - Ensure data security and integrity.
   - Encrypt sensitive user data and transactions.
   - Implement role-based access control (RBAC).

3. **High Availability:**

   - Ensure the platform has minimal downtime and can quickly recover from failures.
   - Implement failover mechanisms and data replication.

4. **Compliance and Auditing:**

   - Maintain detailed logs of all user activities and system operations for audit purposes.
   - Implement data retention policies to comply with financial regulations.

5. **Performance:**
   - Optimize the system for low-latency real-time data processing and querying.

### **2. Model the Data with an ER Diagram**

Let's create an Entity-Relationship (ER) diagram for the financial trading platform.

#### **2.1. Entities:**

1. **User**:

   - Attributes: `user_id (PK)`, `username`, `email`, `password_hash`, `role`, `created_at`, `updated_at`

2. **Stock**:

   - Attributes: `stock_id (PK)`, `ticker_symbol`, `company_name`, `sector`, `industry`

3. **Market Data**:

   - Attributes: `market_data_id (PK)`, `stock_id (FK)`, `timestamp`, `price`, `volume`

4. **Trade**:

   - Attributes: `trade_id (PK)`, `user_id (FK)`, `stock_id (FK)`, `trade_type`, `quantity`, `price`, `trade_date`

5. **Portfolio**:

   - Attributes: `portfolio_id (PK)`, `user_id (FK)`, `stock_id (FK)`, `quantity`

6. **Notification**:

   - Attributes: `notification_id (PK)`, `user_id (FK)`, `message`, `created_at`

7. **Audit Log**:
   - Attributes: `log_id (PK)`, `user_id (FK)`, `action`, `timestamp`, `details`

#### **2.2. Relationships:**

- **User to Trade**: One-to-Many (a user can make multiple trades, but each trade is made by one user).
- **User to Portfolio**: One-to-Many (a user can have multiple stocks in their portfolio).
- **User to Notification**: One-to-Many (a user can receive multiple notifications).
- **User to Audit Log**: One-to-Many (a user can have multiple audit logs).
- **Stock to Trade**: One-to-Many (a stock can be involved in multiple trades).
- **Stock to Market Data**: One-to-Many (a stock can have multiple market data entries).
- **Portfolio to Stock**: Many-to-One (multiple portfolios can include the same stock).

### **ER Diagram Simplified Representation**

```plaintext
User (user_id PK, username, email, password_hash, role, created_at, updated_at)
|
| 1-M
|
Trade (trade_id PK, user_id FK, stock_id FK, trade_type, quantity, price, trade_date)
|
| 1-M
|
Stock (stock_id PK, ticker_symbol, company_name, sector, industry)
|
| 1-M
|
MarketData (market_data_id PK, stock_id FK, timestamp, price, volume)
|
| 1-M
|
Portfolio (portfolio_id PK, user_id FK, stock_id FK, quantity)
|
| 1-M
|
Notification (notification_id PK, user_id FK, message, created_at)
|
| 1-M
|
AuditLog (log_id PK, user_id FK, action, timestamp, details)
```

### **3. Choose Indexing Strategies Based on Expected Queries**

1. **User Table:**

   - Index on `username` for quick login and user searches.
   - Composite index on `email` for email-based lookups.

2. **Stock Table:**

   - Index on `ticker_symbol` for quick stock symbol searches.
   - Composite index on `sector` and `industry` for sector-based queries.

3. **Market Data Table:**

   - Index on `stock_id` and `timestamp` for efficient range queries on historical data.

4. **Trade Table:**

   - Index on `user_id` and `trade_date` for user-specific trade history queries.
   - Composite index on `stock_id` and `trade_date` for stock-specific trade queries.

5. **Portfolio Table:**

   - Index on `user_id` for quick portfolio lookups.
   - Composite index on `stock_id` for stock-based portfolio analysis.

6. **Notification Table:**

   - Index on `user_id` for retrieving user-specific notifications.

7. **Audit Log Table:**
   - Index on `user_id` for user activity tracking.
   - Composite index on `timestamp` for chronological log analysis.

### **4. Address Scalability, Security, and Backup Requirements**

#### **4.1. Scalability:**

1. **Horizontal Scaling**:

   - Use horizontal partitioning (sharding) for the `MarketData` and `Trade` tables based on `timestamp` and `trade_date`, respectively, to manage large volumes of data.
   - Implement read replicas for the `Stock` and `Market Data` tables to distribute read traffic and improve performance.

2. **Load Balancing**:
   - Deploy a load balancer to distribute incoming requests across multiple application servers.
   - Use caching (e.g., Redis) to store frequently accessed data, such as stock information and user session data.

#### **4.2. Security:**

1. **Encryption**:

   - Encrypt sensitive data (e.g., `password_hash`, trades, and user personal information) both at rest and in transit using AES encryption and TLS.

2. **Authentication and Authorization**:

   - Implement robust authentication mechanisms, including multi-factor authentication (MFA).
   - Use role-based access control (RBAC) to restrict access to sensitive data and operations based on user roles.

3. **Auditing and Logging**:
   - Enable detailed logging and auditing of all user actions and system events.
   - Store audit logs securely and ensure that they cannot be tampered with.

#### **4.3. Backup and Recovery:**

1. **Backup Strategy**:

   - Perform daily full backups and hourly incremental backups for all critical data. Store backups in geographically distributed locations to protect against regional failures.
   - Use database snapshots for quick restoration of recent data states.

2. **Recovery Strategy**:

   - Implement point-in-time recovery capabilities to restore the database to a specific point before a failure or data corruption.
   - Regularly test backup and recovery procedures to ensure they work as expected.

3. **High Availability**:
   - Deploy the database in a multi-AZ setup to ensure high availability and automatic failover.
   - Use active-passive failover mechanisms where the passive node takes over if the active node fails.

### **5. Present the Design and Justify the Decisions Made**

This financial trading platform design ensures robust functionality, scalability, security, and high availability.

- **Functional Requirements**: The system supports all critical trading functions, user management, real-time market data, reporting, and notifications.
- **Data Modeling**: The ER diagram captures essential entities and their relationships, ensuring logical data organization and easy scalability.
- **Indexing Strategies**: Carefully chosen indexing strategies based on expected queries ensure optimal query performance and data retrieval times.
- **Scalability**: Horizontal scaling and sharding address the need to handle large volumes of trading activities and market data seamlessly.
- **Security**: Encryption, RBAC, and auditing measures secure sensitive data, restrict access, and ensure compliance with financial regulations.
- **Backup and Recovery**: Comprehensive backup strategies and automated failover mechanisms ensure data protection and high availability, minimizing downtime in case of failures.

This design applies the principles and best practices learned throughout the lecture series to create a resilient and efficient financial trading platform.

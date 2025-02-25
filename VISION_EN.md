# Vision Document



# UI/UX

[Figma Design](https://www.figma.com/design/aV06U5UlwsO6v4UIStAOF7/Untitled?m=auto&t=N4w2qT0imE5Dk7sG-1)

# Problems

```mermaid
---
config:
  theme: base
  look: neo
---
flowchart TD
    A[System Weaknesses] --> B[Not all products scannable]
    A --> C[No mixed payment method]
    A --> D[Two cashiers use same account]
    A --> E[Refunds policy issues]
    A --> F[Adding workers to client en compte]
    B --> B1[Products not tagged or barcodes missing]
    C --> C1[Only cash or card accepted, no split payment]
    D --> D1[Security and accountability concerns]
    E --> E1[No clear guidelines or process]
    F --> F1[No user permission or role differentiation]
    
    style A fill:#f9f,stroke:#333,stroke-width:6px, color:#121111, font-size:28px
    style B fill:#ffcccb, color:#121111, font-size:20px
    style C fill:#ffcccb, color:#121111, font-size:20px
    style D fill:#ffcccb, color:#121111, font-size:20px
    style E fill:#ffcccb, color:#121111, font-size:20px
    style F fill:#ffcccb, color:#121111, font-size:20px
    style B1 fill:#ffeb99, color:#121111, font-size:20px
    style C1 fill:#ffeb99, color:#121111, font-size:20px
    style D1 fill:#ffeb99, color:#121111, font-size:20px
    style E1 fill:#ffeb99, color:#121111, font-size:20px
    style F1 fill:#ffeb99, color:#121111, font-size:20px
    
    linkStyle default stroke:#FFFFFF, stroke-width:2px



```


## Overview

This document presents a comprehensive vision for a digital ordering and payment system designed to facilitate transactions in various points of sale. The system integrates multiple front-end applications with a robust backend architecture to ensure seamless operations.

## Technologies

The system leverages modern technologies for high performance and scalability:

- **Frontend:** Android application for customers, web application for cashiers and admins, and a kiosk-based interface for self-service.
- **Backend:** Symfony-based backend for API management, interfacing with a SQL Server database and Redis for caching.
- **Data Layer:** SQL Server for persistent storage and Redis for fast data retrieval.

## System Design

### System Architecture

The system architecture follows a multi-tier structure where frontend applications communicate with the backend, which manages data processing and storage.

```mermaid
graph TB

    subgraph "Frontend Applications"
        A[Android App]
        W[Cashier/Admin WebApp]
        K[Kiosk App]
    end

    subgraph "Backend"
        S[Symfony Backend]
        
        subgraph "Data Layer"
            DB[(SQL Server)]
            R[(Redis Cache)]
        end
    end

    %% Frontend to Backend connections
    A --> S
    W --> S
    K --> S

    %% Backend to Data connections
    S --> DB
    S --> R

    %% Cache sync
    DB <-.-> R
    
    %% Styling
    classDef frontend fill:#d4e6ff,stroke:#0066cc,color:#0000ff;
    classDef backend fill:#c2f0c2,stroke:#2d862d,color:#000000;
    classDef database fill:#ffe6cc,stroke:#ff8c1a,color:#0000ff;

    class A,W,K frontend
    class S backend
    class DB,R database
```

![](ordering.png)


## Conception

### Entity Relationship Diagram
```mermaid


erDiagram
    USER ||--o{ ORDER : places
    USER ||--|| CLIENT_ACCOUNT : has
    USER ||--|| CART : has
    USER ||--o{ NOTIFICATION : receives
    CLIENT_ACCOUNT ||--o{ CLIENT_ACCOUNT_OPERATION : has
    %% CLIENT_ACCOUNT_OPERATION }o--|| PAYMENT : references
    ORDER ||--o{ ORDER_ITEM : contains
    ORDER ||--|| PAYMENT : has
    ORDER }o--|| POINT_OF_SALE : placedAt
    ORDER_ITEM }|--|| OFFER : includes
    ORDER_ITEM }|--|| OFFER : includes
    OFFER }o--|{ CATEGORY : belongsTo
    OFFER }o--|{ CATEGORY : belongsTo
    MENU }o--o{ OFFER : includes
    MENU ||--|| POINT_OF_SALE : belongsTo
    POINT_OF_SALE ||--|| OPENING_HOURS : hasHours
    PAYMENT }|--|| PAYMENT_STATUS : hasStatus
    CLIENT_ACCOUNT_OPERATION }|--|| OPERATION_TYPE : hasType
    CART ||--|| ORDER : creates
    %% UIASS_CARD }o--|| CLIENT_ACCOUNT : linkedTo

    USER {
        int id PK
        string username
        string password
        string email
        int role_id FK
        boolean isActive
        datetime createdAt
        datetime updatedAt
    }
    
    ROLE {
        int id PK
        string name
    }
    
    CLIENT_ACCOUNT {
        int id PK
        int user_id FK,UK
        string cardNumber
        decimal balance
        boolean isActive
        datetime createdAt
    }
    
    CLIENT_ACCOUNT_OPERATION {
        int id PK
        int account_id FK
        int operator_id FK
        int type_id FK
        decimal amount
        decimal previousBalance
        decimal newBalance
        string description
        int payment_id FK
        datetime createdAt
        string reference
    }
    
    OPERATION_TYPE {
        int id PK
        string name
    }
    
    PAYMENT {
        int id PK
        int order_id FK
        int status_id FK
        decimal amount
        string transactionId
        datetime createdAt
        datetime processedAt
        string notes
        decimal cashReceived
        decimal changeGiven
        string cardNumber
        string cardHolderName
        int account_id FK
        string expirationDate
        string cvv
        string accountReference
        string type

    }


    PAYMENT_STATUS {
        int id PK
        string name
    }
    
    ORDER {
        int id PK
        int customer_id FK
        int cashier_id FK
        int pointOfSale_id FK
        int status_id FK
        int payment_id FK,UK
        datetime createdAt
        datetime updatedAt
        decimal subtotal
        decimal tax
        decimal totalAmount
        string notes
    }
    
    ORDER_STATUS {
        int id PK
        string name
    }
    
    ORDER_ITEM {
        int id PK
        int order_id FK
        int product_id FK 
        int service_id FK
        int quantity
        decimal unitPrice
        decimal subtotal
        datetime createdAt
    }
    
OFFER {
        int id PK
        string name
        string description
        decimal price
        int category_id FK
        boolean isActive
        string imageUrl
        datetime createdAt
        datetime updatedAt
        string type
        string serviceId
        int duration
    }

    CATEGORY {
        int id PK
        string name
        string description
        boolean isActive
        datetime createdAt
        datetime updatedAt
    }
    
    POINT_OF_SALE {
        int id PK
        string name
        string location
        boolean isActive
        int currentMenu_id FK
        int hours_id FK
    }
    
    MENU {
        int id PK
        int pointOfSale_id FK,UK
        boolean isActive
        datetime startDate
        datetime endDate
    }
    
    OPENING_HOURS {
        int id PK
        string schedule
    }
    
    CART {
        int id PK
        int user_id FK,UK
        int order_id FK,UK
        datetime createdAt
        datetime updatedAt
    }
    
    NOTIFICATION {
        int id PK
        int recipient_id FK
        string message
        string type
        boolean isRead
        datetime createdAt
        datetime readAt
    }

    %% PAYMENT <|-- CASH
    %% PAYMENT <|-- UIASS_CARD
    %% PAYMENT <|-- VISA
    %% PAYMENT <|-- CLIENT_ENCOMPTE

```



### Class Diagram

The following diagram represents the core entities of the system, including users, orders, payments, and inventory management. It ensures clear relationships and responsibilities between various system components.


[Diagram Link](https://mermaid.live/edit#pako:eNqtWI1v2jgU_1eiTFVvOzoVaLs2miZlwDZO5UOETtodp80kBqwmcc52WLke__s9x3ZI0kDbba2U2O_Zz8-_9xnubZ8G2HZsP0ScdwlaMhTN4qOjexITYTn3xyHa0FQcO9YxDm-Pt9ujo1ksF1g3HDNrgGK0xBGOxSzORCjy_Sy24O93EguLBHrCBSPx0kphQYwiXKYmsPc7ZZW1OEIk1KQJDbHF4KHncwpjFFuEu74ga0PuICYs-dDzEQsw--tvi8o318QuEnhKImz5DMMwcEWVkSaBYWzN1TIN9NXevsVxGmGGBKHxu3eK6HYH_aEadlzvU783MZNJ92uZcuNNRwM520o4lfxOSABI1_dpCrjVYpihKxEs4-QjFgzTaJ7TA-yTCIXWHIUo9h-BbC8aJYVGib4tgBlymBowGfZXiC3xe3XWby9zHYLUF24kN0vitvaeudj6C5cxQepdBINm-6m5eC5uukmwJeBRQQRFBQmGmDC8JjTl70twGW6Mv5cZGvUAc5-RRJ6mGWO0kbEA3rxRMXEYXy2H4QVmOBOfY1S-xyGnm_SuR25Xjcful0FvODWMDzdDzXA70_5nd9ofaffs9qqU9-61O-z0vrrdP8A3lZStDvUxo9KUFooDawAq1IT9mILZRgsP7UKkNvofRn5IfVSAcI-PZsf6KQOchBzvzB2DkE80ZdxayadmLLFQ-7XuPHdL4HSUnCw18JJrZsfU6l-8YLIbG646BUIj0ec9MeS4gFQlZ1UGjoMCWaUjqV1JXWOY5yH-0G93cUD8XSIVeEnZBpKLGhy-kZYNYpb4hoU_kmgzhr_C_q27hryP5iQkYlNOHUapn73xc1PhXn1rHS0LHMtSOnuYrQFWo7L8O-GK1g8cy8tULPCk-jXkwh1quHPEQQMQ6VhdZczi1lSlDMfqx4CfqRIZDxWgBrkAf5Aat95jEIPFy8IqwPO2yshQ0Ckki7WavKHo-2udn3JBo7yuKRriK5KTDgdmJt8TSKRcxhq8ioy-wBHELIEX_7EUvtcvTEjxdC6oMObIyQLdVSlylVusTsadqcBGOx-FfhrKM-XqPKepw9U1S_GS37IeYoV-1hqVE5lJY4VN_6QoFmD-itop9InjQtrYc-8u4aqEB3rwGLD5TT0t6OG9tFkPFcdxb9jtDz_qyaQ3dif5dNJzu190OzYajK97017X9GtQCa-v5bSQedgTe7KHoD7fe1AQSKvlBmY4omtcImVxCa15nm9kodbOC1b0MedZitCVQnNysNAc3Av5wiD1qGMoAXWhVNtaaeeFM2IOx4BN-sGT4UiU_g9bpWIs6EVasXJdLCn7DAcZdXqel88rbvHB7V-bsWquKh7CV7mHGExkrppgH0OBCKqsFYqX-CNw4sdvdNN3Pe-r_JDIj9jX-xfon2gIFhzu6uGhdvrA6Z_7nvtT55rvubuEKPwLvY3Zt14_rojSvxd3aJSIXadZuyOLiCEVZEFMh6mEFGkHQhq-a0hCcNWjIzgIiljFzXcfGrvOYoLR010e6AVvh-4vD_QIsVs3E1a82ASH2QWgFMovsUzjmd2c2dbJCQxewWCkwrfKkoOSH-xZkn1Da86rIkfLzTJiZdOeI-X2siVUgqlTWCa5Gr4cjE1N3pW1imq61zqwYmwKm6l0FX4n73ZN37Z3QfaxUFWx2IFUF7wqnl_HzNUvNjX1VjUJ_e1_wJGpp0LaJYwKQ8ZydXspqsB8u2Rz2G_K6aSiaP1nvnZfT2xCzK0FZbAOglJmdS5p6gekBQlD50X77PIKzxsQY_QWOy8uLhZ6fPKdBGLltJK7hk9DypwXi0XOCwAMxBiCdvbcOjdijbmNZDy_WvwaycZPfr1kFQRKLpr7Z63gF2Ghja8kzy_brfZPayz_v337ZjdsKLERIoHt2FlundliBS3_zHZgKD9TZjZkMViHUkG9TezbjmApbtiMpsuV7SxQyGGmOiL9i6RZkqD4T0qLU9u5t-9sp_mm-fr04uyqedq6vDhttpoNe2M7J1ft15ct8KLzq3b79PLN5bZh_5vtb78-u2xdNE_fNNtXzXb7vHXRsHFABGUD_XuofG3_B14p650)

```mermaid
---
config:
    layout : elk
---

classDiagram
%%{init :{'layout': 'elk'}}%%
    class User {
        +int id
        +string username
        +string password
        +string email
        +Role role
        +boolean isActive
        +Cart Cart
        +Order[] orders
        +DateTime createdAt
        +DateTime updatedAt
    }
    class Role {
        <<enumeration>>
        ADMIN
        CASHIER
        CARD_CASHIER
        CUSTOMER
    }
    class ClientAccount {
        +int id
        +User user
        +string cardNumber
        +decimal balance
        +boolean isActive
        +DateTime createdAt
        +ClientAccountOperation[] lstOps
        +rechargeBalance()
        +deductAmount()
    }
    class ClientAccountOperation {
        +int id
        +ClientAccount account
        +User operator
        +OperationType type
        +decimal amount
        +decimal previousBalance
        +decimal newBalance
        +string description
        +Payment payment
        +DateTime createdAt
        +string reference
    }
    class OperationType {
        <<enumeration>>
        RELOAD
        PAYMENT
        REFUND
        ACTIVATION
        DEACTIVATION
        BALANCE_ADJUSTMENT
    }
    class PointOfSale {
        +int id
        +string name
        +string location
        +boolean isActive
        +Menu currentMenu
        +OpeningHours hours
        +getActiveProducts()
        +getCurrentOrders()
    }
    class Menu {
        +int id
        +PointOfSale pointOfSale
        +Product[] products
        +boolean isActive
        +DateTime startDate
        +DateTime endDate
        +updateMenu()
    }
    class Product {
        +int id
        +string name
        +string description
        +decimal price
        +Category category
        +boolean isActive
        +string imageUrl
        +DateTime createdAt
        +DateTime updatedAt
        +checkAvailability()
    }

  class Service {
        -serviceId: String
        -name: String
        -description: String
        -basePrice: Decimal
        -duration: Integer
        -availability: Schedule
        +checkAvailability(DateTime)
        +book(DateTime)
    }
    class Category {
        +int id
        +string name
        +string description
        +boolean isActive
        +DateTime createdAt
        +DateTime updatedAt
        +getActiveProducts()
    }
    class Order {
        +int id
        +User customer
        +User cashier
        +PointOfSale pointOfSale
        +OrderStatus status
        +OrderItem[] items
        +Payment payment
        +DateTime createdAt
        +DateTime updatedAt
        +decimal subtotal
        +decimal tax
        +decimal totalAmount
        +string notes
        +calculateTotal()
        +updateStatus()
    }
    class OrderItem {
        +int id
        +Order order
        +Product product
        +int quantity
        +decimal unitPrice
        +decimal subtotal
        +Discount discount
        +DateTime createdAt
        +calculateSubtotal()
    }
    class OrderStatus {
        <<enumeration>>
        PENDING
        PREPARING
        READY
        COMPLETED
        CANCELLED
    }
    class Cart {
        +int id
        +User user
        +Order order
        +DateTime createdAt
        +DateTime updatedAt
        +addItem()
        +removeItem()
        +checkout()
    }

    class Notification {
        +int id
        +User recipient
        +string message
        +string type
        +boolean isRead
        +DateTime createdAt
        +DateTime readAt
        +send()
        +markAsRead()
    }
    class Payment {
        <<abstract>>
        +int id
        +Order order
        +PaymentStatus status
        +decimal amount
        +string transactionId
        +DateTime createdAt
        +DateTime processedAt
        +string notes
        +processPayment()
    }
    class PaymentStatus {
        <<enumeration>>
        PENDING
        PROCESSING
        COMPLETED
        FAILED
        REFUNDED
    }
    class Cash {
        +decimal cashReceived
        +decimal changeGiven
        +processPayment()
    }
    class UIASS_CARD {
        +string cardNumber
        +string cardHolderName
        +ClientAccount account
        +processPayment()
    }
    class VISA {
        +string cardNumber
        +string cardHolderName
        +string expirationDate
        +string cvv
        +processPayment()
    }


 class ClientEnCompte {
        +processPayment()
    }
    User "1" -- "*" Order
    User "1" -- "1" ClientAccount
    User "1" -- "1" Cart
    User "*" -- "1" Order
    Cart "1" -- "1" Order
    User "1" -- "*" Notification
    Order "1" -- "*" OrderItem
    Order "1" -- "1" Payment
    OrderItem "*" -- "1" Service
    OrderItem "*" -- "1" Product
    Product "*" -- "1" Category
    Service "*" -- "1" Category
    Menu "1" -- "1" PointOfSale
    Menu "1" -- "*" Product
    Menu "1" -- "*" Service
    PointOfSale "1" -- "*" Order
    Payment <|-- Cash
    Payment <|-- UIASS_CARD
    Payment <|-- VISA
    Payment <|-- ClientEnCompte
    UIASS_CARD "1" -- "1" ClientAccount
    ClientAccount "1" -- "*" ClientAccountOperation
    %%ClientAccountOperation "*" -- "1" User
    %%ClientAccountOperation "0..1" -- "1" Payment


     style User fill:#3489eb,stroke:#66f,stroke-width:2px,color:#fff,stroke-dasharray: 5 5
     style Product fill:#34eb9f,stroke:#66f,stroke-width:2px,color:#fff,stroke-dasharray: 5 5
     style Service fill:#34eb9f,stroke:#66f,stroke-width:2px,color:#fff,stroke-dasharray: 5 5
     style Order fill:#abc42d,stroke:#66f,stroke-width:2px,color:#fff,stroke-dasharray: 5 5
     style Payment fill:#b8323f,stroke:#66f,stroke-width:2px,color:#fff,stroke-dasharray: 5 5
   
```

### Use Cases

The system caters to different user roles, each having specific responsibilities and access levels. The main user roles include Admins, Customers, Cashiers, and Card Cashiers.

#### Admin Use Cases

Admins have full control over the system, including managing users, products, inventory, and system settings.

```mermaid
flowchart LR
    classDef actor fill:#f96,stroke:#333,stroke-width:2px,color:#000
    classDef usecase fill:#fff,stroke:#333,stroke-width:2px,color:#000
    
    subgraph Actors
        Admin[("Admin")]:::actor
    end
    
    subgraph "Admin Use Cases"
        direction TB
        UC1["Manage Users"]:::usecase
        UC2["Manage Products"]:::usecase
        UC3["Manage Categories"]:::usecase
        UC4["Manage Point of Sales"]:::usecase
        UC5["View Reports"]:::usecase
        UC6["Manage Inventory"]:::usecase
        UC7["Configure Menus"]:::usecase
        UC8["Manage Discounts"]:::usecase
        UC9["View Transaction History"]:::usecase
        UC10["Manage System Settings"]:::usecase
        UC11["Handle User Complaints"]:::usecase
        UC12["Monitor System Performance"]:::usecase
    end
    
    Admin --> UC1
    Admin --> UC2
    Admin --> UC3
    Admin --> UC4
    Admin --> UC5
    Admin --> UC6
    Admin --> UC7
    Admin --> UC8
    Admin --> UC9
    Admin --> UC10
    Admin --> UC11
    Admin --> UC12

  ```

#### Customer Use Cases

Customers interact with the system primarily to view menus, place orders, track orders, and make payments.

```mermaid
flowchart LR
    classDef actor fill:#f96,stroke:#333,stroke-width:2px,color:#000
    classDef usecase fill:#fff,stroke:#333,stroke-width:2px,color:#000
    
    subgraph Actors
        Customer[("Customer")]:::actor
    end
    
    subgraph "Customer Use Cases"
        direction TB
        UC1["View Menu"]:::usecase
        UC2["Place Order"]:::usecase
        UC3["Track Order"]:::usecase
        UC4["Make Payment"]:::usecase
        UC5["View Order History"]:::usecase
        UC6["Manage Cart"]:::usecase
        UC7["Check UIASS Card Balance"]:::usecase
        UC8["View Notifications"]:::usecase
        UC9["Apply Discount"]:::usecase
        UC10["View Receipt"]:::usecase
    end
    
    Customer --> UC1
    Customer --> UC2
    Customer --> UC3
    Customer --> UC4
    Customer --> UC5
    Customer --> UC6
    Customer --> UC7
    Customer --> UC8
    Customer --> UC9
    Customer --> UC10

```

#### Cashier Use Cases

Cashiers process orders, accept payments, and manage transactions within the system.

```mermaid
flowchart LR
    classDef actor fill:#f96,stroke:#333,stroke-width:2px,color:#000
    classDef usecase fill:#fff,stroke:#333,stroke-width:2px,color:#000
    
    subgraph Actors
        Cashier[("Cashier")]:::actor
    end
    
    subgraph "Cashier Use Cases"
        direction TB
        UC1["Process Orders"]:::usecase
        UC2["Manage Cart"]:::usecase
        UC3["Process Payments"]:::usecase
        UC4["Generate Tickets"]:::usecase
        UC5["Print Receipts"]:::usecase
        UC6["View Order Queue"]:::usecase
        UC7["Update Order Status"]:::usecase
        UC8["Check Inventory"]:::usecase
        UC9["Apply Discounts"]:::usecase
        UC10["Handle Refunds"]:::usecase
    end
    
    Cashier --> UC1
    Cashier --> UC2
    Cashier --> UC3
    Cashier --> UC4
    Cashier --> UC5
    Cashier --> UC6
    Cashier --> UC7
    Cashier --> UC8
    Cashier --> UC9
    Cashier --> UC10


  ```

#### Card Cashier Use Cases

Card cashiers handle UIASS card-related operations such as issuing, recharging, and blocking/unblocking cards.

```mermaid
flowchart LR
    classDef actor fill:#f96,stroke:#333,stroke-width:2px,color:#000
    classDef usecase fill:#fff,stroke:#333,stroke-width:2px,color:#000
    
    subgraph Actors
        CardCashier[("Card Cashier")]:::actor
    end
    
    subgraph "Card Cashier Use Cases"
        direction TB
        UC1["Issue UIASS Cards"]:::usecase
        UC2["Recharge Cards"]:::usecase
        UC3["Check Card Balance"]:::usecase
        UC4["Block/Unblock Cards"]:::usecase
        UC5["View Transaction History"]:::usecase
        UC6["Generate Card Reports"]:::usecase
        UC7["Handle Card Issues"]:::usecase
    end
    
    CardCashier --> UC1
    CardCashier --> UC2
    CardCashier --> UC3
    CardCashier --> UC4
    CardCashier --> UC5
    CardCashier --> UC6
    CardCashier --> UC7

```

## Sequence Diagrams

Sequence diagrams illustrate the dynamic interactions between users and the system.

### UIASS Card Order Process

This sequence diagram details the steps involved when a customer places an order using a UIASS card.

```mermaid
sequenceDiagram
    participant C as Customer
    participant K as Kiosk/POS
    participant API as API Server
    participant DB as Database
    participant CA as ClientAccount
    participant N as Notification

    C->>K: Select Point of Sale
    K->>API: Get Available Menu
    API->>DB: Fetch Active Menu
    DB-->>API: Menu Items
    API-->>K: Display Menu

    C->>K: Add Items to Cart
    K->>API: Update Cart
    API->>DB: Save Cart Items
    DB-->>API: Cart Updated
    API-->>K: Show Cart Total

    C->>K: Present UIASS Card
    K->>API: Validate Card
    API->>CA: Check Balance
    CA-->>API: Balance Status
    
    alt Sufficient Balance
        API->>CA: Deduct Amount
        CA->>DB: Update Balance
        API->>DB: Create Order
        DB-->>API: Order Created
        API->>DB: Generate Ticket
        DB-->>API: Ticket Created
        API->>DB: Generate Receipt
        API->>N: Send Notification
        N-->>C: Order Confirmation
        API-->>K: Print Ticket & Receipt
    else Insufficient Balance
        CA-->>API: Insufficient Funds
        API-->>K: Show Error
        K-->>C: Request Different Payment
    end

```

### Mobile Order Placement

This diagram describes the sequence of actions when a customer places an order through the mobile application.

```mermaid
sequenceDiagram
    participant C as Customer
    participant MA as Mobile App
    participant API as API Server
    participant DB as Database
    participant N as Notification System
    participant L as Location System

    C->>MA: Open App
    MA->>API: Authenticate User
    API->>DB: Verify Credentials
    DB-->>API: User Data
    API-->>MA: Auth Token

    C->>MA: Select Location
    MA->>L: Get Location Menu
    L->>DB: Fetch Available Products
    DB-->>L: Products List
    L-->>MA: Menu Items

    C->>MA: Add Items to Cart
    MA->>API: Update Cart
    API->>DB: Save Cart Items
    DB-->>API: Cart Updated
    API-->>MA: Cart Status

    C->>MA: Checkout
    MA->>API: Create Order
    API->>DB: Save Order
    DB-->>API: Order Created
    API->>N: Notify Location
    N-->>L: New Order Alert

    C->>MA: Select Payment Method
    MA->>API: Process Payment
    API->>DB: Update Payment Status
    DB-->>API: Payment Confirmed
    API-->>MA: Order Confirmation
    N->>C: Order Confirmation Notification
```


### Cashier Order Processing

The cashier processes orders manually, ensuring order accuracy and efficient payment handling.

```mermaid
sequenceDiagram
    participant C as Customer
    participant K as Cashier
    participant POS as POS System
    participant API as API Server
    participant DB as Database
    participant P as Payment System

    C->>K: Request to Order
    K->>POS: Login to System
    POS->>API: Authenticate Cashier
    API-->>POS: Auth Confirmed

    K->>POS: Select Products
    POS->>API: Check Availability
    API->>DB: Query Stock
    DB-->>API: Stock Status
    API-->>POS: Available Items

    K->>POS: Add Items to Order
    POS->>API: Calculate Total
    API-->>POS: Order Summary

    C->>K: Choose Payment Method
    alt Cash Payment
        K->>POS: Process Cash Payment
        POS->>API: Record Payment
    else Card Payment
        C->>P: Present Card
        P->>API: Process Card Payment
        API->>DB: Verify Balance
        DB-->>API: Balance Confirmed
        API-->>P: Payment Approved
    end

    API->>DB: Update Order Status
    DB-->>API: Order Confirmed
    API-->>POS: Print Receipt
    K->>C: Provide Receipt
```

## Payment Workflow

The payment process involves multiple methods, including cash, VISA, and UIASS cards. The flowchart below outlines the decision-making process for each payment method.

```mermaid
flowchart TD
    Start([Start Payment]) --> Method{Payment Method}
    
    Method -->|UIASS Card| Balance{Check Balance}
    Method -->|Cash/VISA| Process[Cashier Processes Payment]
    
    Balance -->|Sufficient| Auth[Authorize UIASS Card]
    Balance -->|Insufficient| Fail[Payment Failed]
    
    Auth --> Complete[Complete Transaction]
    Process --> Complete
    
    Complete --> Receipt[Generate Receipt]
    Fail --> Retry[Retry Payment Method]
    
    style Start fill:#90EE90,color:#000000
    style Complete fill:#98FB98,color:#000000
    style Fail fill:#FFB6C1,color:#000000
    style Retry fill:#FFE4B5,color:#000000
```

## Conclusion

This vision document outlines the structure, components, and functionality of the proposed system. With a scalable architecture, clear user roles, and a streamlined transaction process, the system aims to provide an efficient and user-friendly ordering and payment experience.

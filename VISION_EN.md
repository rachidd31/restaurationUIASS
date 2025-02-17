



# Vision Document

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


![[ordering.png]]

### Ordering Process

The ordering process involves customers placing orders through mobile apps, kiosks, or a cashier-assisted interface. Orders are processed by the backend and stored in the database. The flow ensures data consistency and optimized performance.

![[ordering.png]]

## Conception

### Class Diagram

The following diagram represents the core entities of the system, including users, orders, payments, and inventory management. It ensures clear relationships and responsibilities between various system components.

```mermaid


classDiagram
%%{init :{'theme':'dark'}}%%
     
    class User {
        +int id
        +string username
        +string password
        +string email
        +Role role
        +boolean isActive
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
        +DateTime lastReload
        +rechargeBalance()
        +deductAmount()
    }

    class MethodPayment {
        <<enumeration>>
        CASH
        VISA
        UIASS_CARD
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

    class Inventory {
        +int id
        +Product product
        +PointOfSale pointOfSale
        +int quantity
        +updateStock()
        +checkAvailability()
    }

    class Product {
        +int id
        +string name
        +string description
        +decimal price
        +Category category
        +boolean isActive
        +string imageUrl
        +checkAvailability()
    }

    class Category {
        +int id
        +string name
        +string description
        +boolean isActive
        +getActiveProducts()
    }

    class Order {
        +int id
        +Ticket ticket
        +User customer
        +User cashier
        +PointOfSale pointOfSale
        +OrderStatus status
        +OrderItem[] items
        +MethodPayment paymentMethod
        +DateTime createdAt
        +decimal totalAmount
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

    class Ticket {
        +int id
        +string ticketNumber
        +Order order
        +DateTime createdAt
        +generateTicketNumber()
    }

    class Receipt {
        +int id
        +Order order
        +string receiptNumber
        +DateTime issuedAt
        - +decimal totalAmount
        +decimal taxAmount
        +generateReceipt()
    }

    class Cart {
        +int id
        +User user
        +PointOfSale pointOfSale
        +OrderItem[] items
        +addItem()
        +removeItem()
        +checkout()
    }

    class Discount {
        +int id
        +string code
        +decimal percentage
        +boolean isActive
        +DateTime validFrom
        +DateTime validTo
        +validateDiscount()
    }

    class Notification {
        +int id
        +User recipient
        +string message
        +string type
        +boolean isRead
        +DateTime createdAt
        +send()
        +markAsRead()
    }
    

    User "1" -- "*" Order
    User "1" -- "1" ClientAccount
    User "1" -- "1" Cart
    User "1" -- "*" Notification
    Order "1" -- "1" Ticket
    Order "1" -- "1" Receipt
    Order "1" -- "*" OrderItem
    OrderItem "*" -- "1" Product
    OrderItem "*" -- "0..1" Discount
    Product "*" -- "1" Category
    Product "1" -- "*" Inventory
    PointOfSale "1" -- "1" Menu
    PointOfSale "1" -- "*" Inventory
    PointOfSale "1" -- "*" Order
    Cart "*" -- "1" PointOfSale
   
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

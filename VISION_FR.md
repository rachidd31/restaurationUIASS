# Technologies

<svg version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" 
	 viewBox="0 0 58 58" xml:space="preserve">
<rect style="fill:#ECF0F1;" width="58" height="58"/>
<rect style="fill:#546A79;" width="58" height="12"/>
<circle style="fill:#ED7161;" cx="6" cy="6" r="3"/>
<circle style="fill:#F0C419;" cx="15" cy="6" r="3"/>
<circle style="fill:#4FBA6F;" cx="24" cy="6" r="3"/>
<rect x="5" y="17" style="fill:#8697CB;" width="48" height="36"/>
<g>
	<path style="fill:#FFFFFF;" d="M30,29h9c0.552,0,1-0.448,1-1s-0.448-1-1-1h-9c-0.552,0-1,0.448-1,1S29.448,29,30,29z"/>
	<path style="fill:#FFFFFF;" d="M48,37H30c-0.552,0-1,0.448-1,1s0.448,1,1,1h18c0.552,0,1-0.448,1-1S48.552,37,48,37z"/>
	<path style="fill:#FFFFFF;" d="M30,34h18c0.552,0,1-0.448,1-1s-0.448-1-1-1H30c-0.552,0-1,0.448-1,1S29.448,34,30,34z"/>
	<path style="fill:#FFFFFF;" d="M48,42H30c-0.552,0-1,0.448-1,1s0.448,1,1,1h18c0.552,0,1-0.448,1-1S48.552,42,48,42z"/>
</g>
<rect x="9" y="27" style="fill:#556080;" width="18" height="17"/>
</svg>

<svg width="400" height="180">
  <rect width="300" height="100" style="fill:rgb(2,2,2);stroke-width:2;stroke:rgb(0,0,0)" />
  <text x="150" y="50" font-size="20" fill="white" text-anchor="middle">Login Form</text>
</svg>

<svg viewBox="0 0 10 10" xmlns="http://www.w3.org/2000/svg" style="background-color: #21ff">
  <rect width="10" height="10">
    <animate
      attributeName="rx"
      values="0;5;0"
      dur="10s"
      repeatCount="indefinite" />
  </rect>
</svg>


*Cette section liste les technologies utilisées dans le système.*

# System Design

*Cette section présente la conception globale du système et son architecture.*

## System Architecture Diagram

*Ce diagramme illustre l'architecture générale du système, montrant comment les différentes applications frontend (Android, Web, Kiosque) interagissent avec le backend Symfony et les bases de données.*

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

# Conception

*Cette section détaille la conception du système, y compris les diagrammes de classes et les cas d'utilisation.*

## Classes Diagram

*Ce diagramme de classes montre la structure complète du système, définissant toutes les entités et leurs relations. Il inclut les utilisateurs, les comptes clients, les points de vente, les produits, les commandes et plus encore.*

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

    [... rest of the class diagram ...]
```

## Use Cases

*Cette section décrit les différents cas d'utilisation pour chaque type d'utilisateur du système.*

### Admin:

*Ce diagramme montre toutes les actions possibles pour un administrateur, comme la gestion des utilisateurs, des produits, et des points de vente.*

```mermaid
flowchart LR
    [... Admin use case diagram ...]
```

### Customer:

*Ce diagramme illustre les actions qu'un client peut effectuer, comme voir le menu, passer une commande, et suivre son statut.*

```mermaid
flowchart LR
    [... Customer use case diagram ...]
```

### Cashier:

*Ce diagramme présente les fonctionnalités disponibles pour un caissier, notamment le traitement des commandes et des paiements.*

```mermaid
flowchart LR
    [... Cashier use case diagram ...]
```

### Card Cashier:

*Ce diagramme montre les actions spécifiques au caissier de carte, comme la gestion des cartes UIASS.*

```mermaid
flowchart LR
    [... Card Cashier use case diagram ...]
```

## Sequence diagram

*Cette section présente les diagrammes de séquence qui montrent comment les différents composants du système interagissent.*

### UIASS Card Order Process Sequence

*Ce diagramme montre la séquence d'actions lors d'une commande avec une carte UIASS.*

```mermaid
sequenceDiagram
    [... UIASS Card sequence diagram ...]
```

### Mobile Order Placement Sequence

*Ce diagramme illustre le processus de commande via l'application mobile.*

```mermaid
sequenceDiagram
    [... Mobile order sequence diagram ...]
```

### Cashier Order Processing Sequence

*Ce diagramme détaille le processus de traitement des commandes par le caissier.*

```mermaid
sequenceDiagram
    [... Cashier processing sequence diagram ...]
```

# Payment

*Cette section décrit le processus de paiement avec les différentes méthodes disponibles (carte UIASS, espèces, VISA).*

```mermaid
flowchart TD
    [... Payment flowchart ...]
```

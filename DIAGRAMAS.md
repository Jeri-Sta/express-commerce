# Diagramas — Express Commerce

## Diagrama ER (Entidade-Relacionamento)

Entidades centrais: **Usuário**, **Produto**, **Pedido** e **Item de Pedido**.

![Diagrama ER](docs/diagrama-er.png)

```mermaid
erDiagram
    USER {
        string id
        string name
        string email
        string passwordHash
        datetime createdAt
    }
    PRODUCT {
        string id
        string title
        string description
        float price
        string category
        string condition
        string sellerId
        string imageUrl
        int stock
    }
    ORDER {
        string id
        string buyerId
        float total
        string status
        datetime createdAt
    }
    ORDER_ITEM {
        string orderId
        string productId
        int quantity
        float unitPrice
    }

    USER ||--o{ PRODUCT : "vende"
    USER ||--o{ ORDER : "compra"
    ORDER ||--|{ ORDER_ITEM : "contém"
    PRODUCT ||--o{ ORDER_ITEM : "referenciado em"
```

---

## Diagrama de Fluxo — Busca e Adicionar ao Carrinho

Fluxo completo do usuário desde a busca de um produto até a adição ao carrinho.

![Diagrama de Fluxo](docs/diagrama-fluxo.png)

```mermaid
flowchart TD
    A([Usuário acessa Home]) --> B[Digite termo na barra de busca]
    B --> C{Termo informado?}
    C -- Não --> D[Exibe todos os produtos]
    C -- Sim --> E["GET /api/products?q=termo"]
    D --> F[Renderiza cards de produtos]
    E --> F
    F --> G["Clica em 'Adicionar ao Carrinho'"]
    G --> H{Está autenticado?}
    H -- Não --> I[Redireciona para login]
    I --> J[POST /api/users/login]
    J --> K[Salva JWT no localStorage]
    K --> G
    H -- Sim --> L["POST /api/cart/items {productId, qty}"]
    L --> M{Produto disponível em estoque?}
    M -- Não --> N[Exibe erro: sem estoque]
    M -- Sim --> O[Item adicionado ao carrinho]
    O --> P[Atualiza badge do carrinho na UI]
```

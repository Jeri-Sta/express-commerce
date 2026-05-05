# ADR — Express Commerce: Marketplace de Eletrônicos Usados

## Stack & Decisões de Arquitetura

| Camada        | Escolha                  | Justificativa                                            |
| ------------- | ------------------------ | -------------------------------------------------------- |
| Runtime       | Node.js + TypeScript     | Tipagem estática, melhor DX e manutenibilidade           |
| Framework     | Express.js               | Minimalista, flexível, ecossistema maduro                |
| Persistência  | In-memory store          | Simplicidade inicial; substituível por SQLite/PostgreSQL |
| Frontend      | HTML + CSS + JS estático | Servido pelo próprio Express via `express.static`        |
| Auth          | JWT via `jsonwebtoken`   | Stateless, sem necessidade de sessão no servidor         |
| Hash de senha | `bcryptjs`               | Padrão de mercado para hash seguro                       |

---

## Estrutura de Diretórios

```
express-commerce/
├── package.json
├── tsconfig.json
├── .env.example
├── src/
│   ├── server.ts            # ponto de entrada
│   ├── app.ts               # configuração do Express
│   ├── data/
│   │   └── store.ts         # in-memory store (users, products, orders, cart)
│   ├── types/
│   │   └── index.ts         # interfaces: User, Product, Order, CartItem
│   ├── middlewares/
│   │   ├── auth.ts          # verificação JWT
│   │   └── errorHandler.ts  # handler de erros global
│   ├── controllers/
│   │   ├── userController.ts
│   │   ├── productController.ts
│   │   ├── cartController.ts
│   │   └── checkoutController.ts
│   └── routes/
│       ├── users.ts
│       ├── products.ts
│       ├── cart.ts
│       └── checkout.ts
└── public/
    ├── index.html           # Home com barra de busca + cards
    ├── css/
    │   └── styles.css
    └── js/
        └── app.js           # fetch na API + renderização de cards
```

---

## API REST — Endpoints

### Usuários — `POST /api/users`

| Método | Rota                  | Descrição                        | Auth |
| ------ | --------------------- | -------------------------------- | ---- |
| POST   | `/api/users/register` | Cadastro com nome, email e senha | Não  |
| POST   | `/api/users/login`    | Autenticação, retorna JWT        | Não  |

### Produtos — `/api/products`

| Método | Rota                | Descrição                                | Auth |
| ------ | ------------------- | ---------------------------------------- | ---- |
| GET    | `/api/products`     | Listagem com filtro `?q=termo&category=` | Não  |
| GET    | `/api/products/:id` | Detalhe do produto                       | Não  |
| POST   | `/api/products`     | Criar produto                            | Sim  |

### Carrinho — `/api/cart`

| Método | Rota                         | Descrição                                | Auth |
| ------ | ---------------------------- | ---------------------------------------- | ---- |
| GET    | `/api/cart`                  | Ver carrinho do usuário autenticado      | Sim  |
| POST   | `/api/cart/items`            | Adicionar item `{ productId, quantity }` | Sim  |
| DELETE | `/api/cart/items/:productId` | Remover item do carrinho                 | Sim  |

### Checkout — `/api/checkout`

| Método | Rota            | Descrição                                   | Auth |
| ------ | --------------- | ------------------------------------------- | ---- |
| POST   | `/api/checkout` | Criar pedido a partir do carrinho e zerá-lo | Sim  |

---

## Tipos Centrais — `src/types/index.ts`

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  passwordHash: string;
  createdAt: Date;
}

interface Product {
  id: string;
  title: string;
  description: string;
  price: number;
  category: string;
  condition: 'novo' | 'seminovo' | 'usado';
  sellerId: string;
  imageUrl: string;
  stock: number;
}

interface CartItem {
  productId: string;
  quantity: number;
}

interface Order {
  id: string;
  buyerId: string;
  items: CartItem[];
  total: number;
  status: 'pendente' | 'confirmado' | 'cancelado';
  createdAt: Date;
}
```

---

## Home Page Estática — `public/`

- `index.html` — navbar (logo + badge do carrinho + botão login), barra de busca centralizada e grid de cards de produtos
- `css/styles.css` — design moderno com CSS Grid, variáveis CSS e layout responsivo
- `js/app.js` — chama `GET /api/products?q=` com debounce na digitação e renderiza cards com imagem, título, preço, condição e botão "Adicionar ao Carrinho"

---

## Dependências

### Runtime

```
express
jsonwebtoken
bcryptjs
cors
dotenv
uuid
```

### Dev

```
typescript
ts-node-dev
@types/express
@types/node
@types/jsonwebtoken
@types/bcryptjs
@types/cors
@types/uuid
```

# FSW Donalds

Aplicação web de pedidos para restaurantes, desenvolvida com Next.js. O cliente escolhe um restaurante, seleciona entre consumo no local ou retirada, monta uma sacola, informa nome e CPF e pode pagar pelo Stripe. A consulta de pedidos é feita pelo CPF informado.

> A página inicial (`/`) ainda exibe conteúdo de demonstração (“Hello World”); o fluxo principal da aplicação está nas rotas de restaurante.

## Tecnologias

- **Next.js 15** com App Router e **React 19**
- **TypeScript** e **Tailwind CSS**
- **Prisma 7** com adaptador PostgreSQL (`pg`)
- **Stripe** para checkout e atualização de status por webhook
- **React Hook Form** e **Zod** para formulários e validação
- Componentes de interface baseados em Radix UI, além de `lucide-react` e `sonner`

## Requisitos

- Node.js e npm
- Banco de dados PostgreSQL acessível pela aplicação
- Chaves do Stripe para habilitar o checkout e o webhook

## Configuração e execução

1. Instale as dependências:

   ```bash
   npm ci
   ```

   O `postinstall` do projeto gera o Prisma Client.

2. Configure as variáveis abaixo em um arquivo `.env` na raiz. Não versione esse arquivo nem compartilhe valores secretos:

   | Variável | Uso |
   | --- | --- |
   | `DATABASE_URL` | String de conexão com PostgreSQL |
   | `STRIPE_SECRET_KEY` | Chave secreta usada no servidor para criar sessões de checkout e validar eventos |
   | `STRIPE_WEBHOOK_SECRET_KEY` | Segredo de assinatura do endpoint de webhook do Stripe |
   | `NEXT_PUBLIC_STRIPE_PUBLIC_KEY` | Chave pública carregada pelo navegador para redirecionar ao checkout |

3. Garanta que o banco esteja acessível e com o schema Prisma aplicado.

4. Inicie o servidor de desenvolvimento:

   ```bash
   npm run dev
   ```

   A aplicação ficará disponível em [http://localhost:3000](http://localhost:3000).

Para produção, use `npm run build` e depois `npm start`. O comando `npm run lint` executa o lint configurado no projeto.

## Rotas e fluxos

| Rota | Descrição |
| --- | --- |
| `/` | Página inicial de demonstração. |
| `/[slug]` | Apresenta o restaurante identificado pelo `slug` e permite escolher consumo no local (`DINE_IN`) ou retirada (`TAKE_AWAY`). |
| `/[slug]/menu?consumption_method=DINE_IN` | Exibe categorias e produtos do restaurante. O parâmetro também aceita `TAKE_AWAY`. |
| `/[slug]/menu/[productId]` | Detalhes do produto e seleção da quantidade para adicionar à sacola. |
| `/[slug]/orders` | Solicita um CPF válido e mostra os pedidos associados a esse CPF. |
| `/api/webhooks/stripe` | Endpoint `POST` que recebe eventos do Stripe e atualiza o status do pedido. |

Restaurante e produto inexistentes, assim como métodos de consumo inválidos no menu, resultam em página não encontrada.

### Fluxo de pedido e pagamento

1. A pessoa escolhe como deseja consumir e navega pelo menu.
2. Os produtos e quantidades ficam em um contexto React de sacola durante a sessão.
3. Para concluir, o formulário valida nome e CPF e a aplicação cria um pedido `PENDING` no PostgreSQL.
4. O servidor cria uma sessão de checkout Stripe para pagamento com cartão ou boleto.
5. O webhook atualiza o pedido para `PAYMENT_CONFIRMED` quando recebe `checkout.session.completed` e para `PAYMENT_FAILED` em `charge.failed`.
6. A tela de pedidos exibe status, itens e total após a busca pelo CPF.

Os demais status definidos no schema são `IN_PREPARATION` e `FINISHED`.

## Organização de `src/app`

```text
src/app/
├── api/webhooks/stripe/route.ts       # Webhook do Stripe
├── [slug]/
│   ├── page.tsx                       # Escolha do método de consumo
│   ├── components/                    # Opções de consumo
│   ├── menu/
│   │   ├── page.tsx                   # Categorias e produtos
│   │   ├── [productId]/               # Detalhes do produto
│   │   ├── actions/                   # Criação do pedido e checkout
│   │   ├── components/                # Menu, sacola e finalização
│   │   ├── contexts/cart.tsx          # Estado da sacola
│   │   └── helpers/cpf.ts             # Validação e normalização de CPF
│   └── orders/                        # Consulta e listagem de pedidos
├── globals.css                        # Estilos globais e tokens de tema
├── layout.tsx                         # Layout raiz, fontes e provedores
└── page.tsx                           # Página inicial
```

O layout raiz configura a fonte Poppins, o contexto da sacola e as notificações. O schema de restaurantes, categorias, produtos e pedidos está em `prisma/schema.prisma`.

## Scripts npm

| Comando | Ação |
| --- | --- |
| `npm run dev` | Inicia o servidor de desenvolvimento do Next.js. |
| `npm run build` | Gera a compilação de produção. |
| `npm start` | Inicia a aplicação compilada. |
| `npm run lint` | Executa o lint do Next.js. |
| `npm run postinstall` | Gera o Prisma Client após a instalação das dependências. |


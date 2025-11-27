
---

# 🧦 Backend - Sistema de Estoque “meia meia meia”

Este é o backend RESTful em **Node.js + Express + PostgreSQL** do sistema de controle de estoque da fábrica **“meia meia meia”**.  
Ele fornece autenticação simples de divulgadores, cadastro e gerenciamento de produtos (modelos de meias), além do registro de movimentações de entrada e saída com atualização automática de saldo.

---

## 🚀 Tecnologias

![Node.js](https://img.shields.io/badge/Node.js-43853D?style=for-the-badge&logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?style=for-the-badge&logo=express&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?style=for-the-badge&logo=postgresql&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)

---

## 🧰 Pré-requisitos

- **Node.js** (v18 ou superior)
- **PostgreSQL** (rodando localmente)
- Banco de dados chamado **`saep_db`**

---

## 📦 Instalação e execução

```bash
git clone https://github.com/seu-usuario/meia-meia-meia-backend.git
cd meia-meia-meia-backend
npm install
node server.js
````

O servidor iniciará por padrão em **[http://localhost:3000](http://localhost:3000)**

---

## 🔗 Rotas disponíveis

| Método     | Rota                         | Descrição                                                  |
| ---------- | ---------------------------- | ---------------------------------------------------------- |
| **POST**   | `/usuarios`                  | Cadastra novo divulgador                                   |
| **POST**   | `/auth/login`                | Login simples (retorna `{ id, nome, email }`)              |
| **GET**    | `/produtos?q=`               | Lista produtos (ordem alfabética; busca opcional)          |
| **GET**    | `/produtos/:id`              | Obtém um produto específico                                |
| **POST**   | `/produtos`                  | Cria um novo produto                                       |
| **PUT**    | `/produtos/:id`              | Atualiza um produto existente                              |
| **DELETE** | `/produtos/:id`              | Exclui um produto                                          |
| **POST**   | `/movimentacoes`             | Registra entrada/saída, atualiza o saldo e grava histórico |
| **GET**    | `/movimentacoes?produto_id=` | Lista o histórico completo ou filtrado por produto         |
| **GET**    | `/health`                    | Verifica se o backend está ativo                           |

---

## 🗃️ Estrutura do banco de dados

Banco: **`saep_db`**

```sql
CREATE TABLE IF NOT EXISTS usuarios (
  id     SERIAL PRIMARY KEY,
  nome   TEXT NOT NULL,
  email  TEXT NOT NULL UNIQUE,
  senha  TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS produtos (
  id              SERIAL PRIMARY KEY,
  nome            TEXT NOT NULL,
  quantidade      INTEGER NOT NULL DEFAULT 0,
  estoque_minimo  INTEGER NOT NULL DEFAULT 0
);

CREATE TABLE IF NOT EXISTS movimentacoes (
  id                 SERIAL PRIMARY KEY,
  produto_id         INTEGER NOT NULL REFERENCES produtos(id),
  usuario_id         INTEGER NOT NULL REFERENCES usuarios(id),
  tipo               TEXT NOT NULL,             -- 'entrada' | 'saida'
  quantidade         INTEGER NOT NULL,
  data_movimentacao  TIMESTAMP NOT NULL DEFAULT NOW(),
  observacao         TEXT
);
```

---

## 🌱 Dados iniciais (seeds)

```sql
-- Usuários (divulgadores)
INSERT INTO usuarios (nome, email, senha) VALUES
  ('Ana Souza',  'ana@example.com',   '123'),
  ('Bruno Lima', 'bruno@example.com', '123'),
  ('Carla Dias', 'carla@example.com', '123')
ON CONFLICT (email) DO NOTHING;

-- Produtos (modelos oficiais da "meia meia meia")
INSERT INTO produtos (nome, quantidade, estoque_minimo) VALUES
  ('meia meia meia arrastão', 40, 10),
  ('499,5 (meia meia meia 3/4)', 60, 15),
  ('000 (meia meia meia de cano invisível)', 25, 12)
ON CONFLICT DO NOTHING;

-- Movimentações (histórico inicial)
-- Entradas iniciais (Ana)
INSERT INTO movimentacoes (produto_id, usuario_id, tipo, quantidade, data_movimentacao, observacao) VALUES
  ((SELECT id FROM produtos WHERE nome='meia meia meia arrastão'),
   (SELECT id FROM usuarios WHERE email='ana@example.com'),
   'entrada', 30, NOW() - INTERVAL '2 days', 'Compra inicial'),
  ((SELECT id FROM produtos WHERE nome='499,5 (meia meia meia 3/4)'),
   (SELECT id FROM usuarios WHERE email='ana@example.com'),
   'entrada', 50, NOW() - INTERVAL '2 days', 'Compra inicial'),
  ((SELECT id FROM produtos WHERE nome='000 (meia meia meia de cano invisível)'),
   (SELECT id FROM usuarios WHERE email='ana@example.com'),
   'entrada', 20, NOW() - INTERVAL '2 days', 'Compra inicial');

-- Saídas (Bruno)
INSERT INTO movimentacoes (produto_id, usuario_id, tipo, quantidade, data_movimentacao, observacao) VALUES
  ((SELECT id FROM produtos WHERE nome='meia meia meia arrastão'),
   (SELECT id FROM usuarios WHERE email='bruno@example.com'),
   'saida', 6, NOW() - INTERVAL '1 day', 'Retirada para evento'),
  ((SELECT id FROM produtos WHERE nome='499,5 (meia meia meia 3/4)'),
   (SELECT id FROM usuarios WHERE email='bruno@example.com'),
   'saida', 15, NOW() - INTERVAL '1 day', 'Retirada para feira'),
  ((SELECT id FROM produtos WHERE nome='000 (meia meia meia de cano invisível)'),
   (SELECT id FROM usuarios WHERE email='bruno@example.com'),
   'saida', 4, NOW() - INTERVAL '1 day', 'Retirada para divulgação');

-- Reposição (Carla)
INSERT INTO movimentacoes (produto_id, usuario_id, tipo, quantidade, observacao) VALUES
  ((SELECT id FROM produtos WHERE nome='meia meia meia arrastão'),
   (SELECT id FROM usuarios WHERE email='carla@example.com'),
   'entrada', 10, 'Devolução de kits'),
  ((SELECT id FROM produtos WHERE nome='499,5 (meia meia meia 3/4)'),
   (SELECT id FROM usuarios WHERE email='carla@example.com'),
   'entrada', 20, 'Devolução de kits'),
  ((SELECT id FROM produtos WHERE nome='000 (meia meia meia de cano invisível)'),
   (SELECT id FROM usuarios WHERE email='carla@example.com'),
   'entrada', 8, 'Devolução de kits');
```

---

## 🧪 Teste rápido

1. Inicie o PostgreSQL e rode os comandos SQL acima no banco `saep_db`.
2. Execute:

   ```bash
   node server.js
   ```
3. No navegador ou Insomnia/Postman, teste:

   * `GET http://localhost:3000/health`
   * `POST http://localhost:3000/auth/login` com:

     ```json
     { "email": "ana@example.com", "senha": "123" }
     ```
4. Acesse o frontend e entre com o mesmo login.

---

## 💡 Observações

* A busca `/produtos?q=` agora **não usa `unaccent()`**, para evitar dependência de extensões PostgreSQL.
* O alerta de estoque é disparado **somente quando `quantidade < estoque_minimo`**.
* O backend **sempre confirma** a movimentação, exibindo o alerta de estoque logo após.
* O projeto segue o padrão mínimo exigido pela **prova SAEP**, totalmente compatível com o frontend React criado para ela.

---

## 📄 Licença

Projeto sob licença MIT.

---

Feito com 💙 por [rafaellindemann](https://github.com/rafaellindemann)

---



# Cloud-DevOps

## Sumário
- [Descrição Geral](#descrição-geral)
- [Tecnologias Utilizadas](#tecnologias-utilizadas)
- [Estrutura do Repositório](#estrutura-do-repositório)
- [Como Funciona](#como-funciona)
- [Configuração](#configuração)
- [Execução](#execução)
- [Endpoints Principais](#endpoints-principais)
- [Dicas e Observações](#dicas-e-observações)

---

## Descrição Geral
Este projeto é uma API backend para controle de unidades, setores, salas e reservas de salas. A aplicação foi desenvolvida com TypeScript + Express + Prisma e usa PostgreSQL como banco de dados relacional.

O objetivo principal é permitir:
- cadastro de unidades e setores;
- cadastro de salas com capacidade;
- criação de reservas com validação de conflitos de horário;
- consulta e remoção dos registros via endpoints HTTP.

---

## Tecnologias Utilizadas
- **TypeScript**: linguagem principal da aplicação
- **Node.js**: runtime do backend
- **Express**: servidor HTTP e rotas da API
- **Prisma ORM**: acesso ao banco de dados e modelos tipados
- **PostgreSQL**: banco relacional principal
- **Docker / Docker Compose**: provisionamento do banco e containers auxiliares
- **pg + @prisma/adapter-pg**: conexão PostgreSQL com Prisma
- **CORS**: liberação de acesso entre origem do cliente e API

---

## Estrutura do Repositório
- `src/server.ts`: contém a configuração do Express, conexão com o banco e os endpoints da API.
- `prisma/schema.prisma`: definição dos modelos do banco e do datasource PostgreSQL.
- `init.sql`: script SQL inicial para criar as tabelas `unidade`, `setor`, `sala` e `reserva`.
- `docker-compose.yml`: define os serviços de banco de dados, pgAdmin e API.
- `Dockerfile`: imagem usada para executar a aplicação Node.js.
- `package.json`: dependências e scripts de execução.
- `.env`: arquivo com a string de conexão do banco em ambiente local.
- `tsconfig.json`: configuração do TypeScript.

---

## Como Funciona
A aplicação inicia em `src/server.ts`, onde:

1. o app Express é montado;
2. o `PrismaClient` é configurado com um `Pool` do PostgreSQL;
3. as rotas de CRUD são registradas;
4. a API escuta na porta `3000`.

Os modelos principais do schema são:
- `unidade`
- `setor`
- `sala`
- `reserva`

A regra de negócio mais importante fica em `POST /reservas`: antes de criar uma reserva, o sistema verifica se já existe algum conflito de horário para a mesma sala na mesma data.

---

## Configuração

### 1. Pré-requisitos
Certifique-se de ter instalado:
- Node.js 20+ (ou versão compatível)
- npm
- Docker e Docker Compose (para rodar PostgreSQL e pgAdmin facilmente)

### 2. Instalação das dependências
```bash
npm install
```

### 3. Variáveis de ambiente
O projeto já contém um arquivo `.env` com a conexão local para o banco:

```env
DATABASE_URL="postgresql://admin:adminpassword@localhost:5433/reserva_salas?schema=public"
```

Esse valor é usado quando a aplicação roda no host local. Dentro do container Docker, a URL é configurada para usar o hostname `db` no compose.

### 4. Banco de dados
A forma mais simples de subir o banco é com Docker Compose:

```bash
docker compose up -d db
```

Se quiser subir também o pgAdmin:

```bash
docker compose up -d db pgadmin
```

O arquivo `init.sql` já cria as tabelas iniciais do sistema ao inicializar o PostgreSQL.

### 5. Prisma
Se você alterar o schema e quiser sincronizar o cliente:

```bash
npx prisma generate
```

Para empurrar alterações do schema para o banco local:

```bash
npx prisma db push
```

---

## Execução

### Opção 1: execução local com Node.js
```bash
npm run dev
```

Essa forma usa o comando:

```json
"dev": "tsx watch src/server.ts"
```

Ou seja, a aplicação sobe em modo watch e recarrega automaticamente quando houver alterações.

### Opção 2: execução via Docker Compose
No projeto, o `docker-compose.yml` inclui os serviços:
- `db` → PostgreSQL
- `pgadmin` → interface visual para o banco
- `api` → backend da aplicação
- `frontend` → serviço front-end (se o diretório estiver presente)

Para subir a API em container:

```bash
docker compose up --build api
```

> Observação: o serviço `frontend` está configurado em `./frontend`, mas esse diretório não existe neste repositório. Se você quiser usar o compose completo, é necessário criar a pasta do frontend ou remover esse serviço do arquivo `docker-compose.yml`.

---

## Endpoints Principais
A API expõe as rotas para as entidades do sistema.

### Unidades
- `POST /unidades`
- `GET /unidades`
- `PUT /unidades/:id`
- `DELETE /unidades/:id`

### Setores
- `POST /setores`
- `GET /setores`
- `PUT /setores/:id`
- `DELETE /setores/:id`

### Salas
- `POST /salas`
- `GET /salas`
- `PUT /salas/:id`
- `DELETE /salas/:id`

### Reservas
- `POST /reservas`
- `GET /reservas`
- `DELETE /reservas/:id`

### Exemplo de payload para reserva
```json
{
  "sala_id": 1,
  "responsavel": "João da Silva",
  "data_reserva": "2026-08-15",
  "hora_inicio": "09:00:00",
  "hora_fim": "10:30:00"
}
```

A API rejeita reservas conflitantes na mesma sala, no mesmo dia, quando os intervalos de horário se sobrepõem.

---

## Dicas e Observações
- A URL do banco está configurada para `localhost:5433` em ambiente local, e `db` dentro do Docker.
- O projeto usa `PrismaPg` com `@prisma/adapter-pg`, então a conexão com PostgreSQL é feita de forma moderna e compatível com Prisma 7.
- O banco já vem com a estrutura de tabelas mínima em `init.sql`, então você pode iniciar rapidamente sem precisar rodar migrations manuais.
- Para acompanhar o banco visualmente, acesse o pgAdmin em:
  - `http://localhost:5050`

---

## Resumo Rápido
Se quiser rodar o projeto localmente em poucos passos:

```bash
npm install
docker compose up -d db pgadmin
npm run dev
```

A API ficará disponível em:

```text
http://localhost:3000
```

E o PostgreSQL ficará acessível em:

```text
localhost:5433
```

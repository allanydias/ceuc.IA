# CEUC.IA — Trilha de carreira para mães

Aplicação que gera uma trilha de carreira personalizada para mães, cruzando
respostas de um onboarding com um modelo de IA (Groq), feita para o HackaWoman 2026.

## Estrutura

```
Ceuc.IA/
├── frontend/
│   ├── index.html      # telas da aplicação (splash, login, perguntas, resultado, cronograma)
│   ├── styles.css      # identidade visual (roxo/lilás, fonte Inter)
│   └── app.js          # lógica de onboarding e chamadas à API
└── backend/
    ├── server.js         # servidor Node.js + Express (API REST)
    ├── trilhaEngine.js    # motor de geração da trilha via IA (Groq)
    ├── prisma/
    │   └── schema.prisma  # modelo do banco (User, QuestionnaireResponse, CareerPath)
    ├── prisma.config.ts   # configuração do Prisma 7 (datasource, migrations)
    ├── package.json
    └── .env.exemplo       # modelo de variáveis de ambiente (sem valores reais)
```

## Tecnologias

- **Front-end**: HTML, CSS e JavaScript puro
- **Back-end**: Node.js + Express
- **Banco de dados**: PostgreSQL (via Prisma ORM 7, com adapter `@prisma/adapter-pg`)
  - Ambiente de desenvolvimento: [Neon](https://neon.tech) (Postgres serverless gratuito)
  - Ambiente de produção: [Magalu Cloud](https://magalu.cloud) (DBaaS PostgreSQL)
- **IA**: [Groq](https://console.groq.com) (modelo `openai/gpt-oss-20b`)
- **Autenticação**: JWT + bcrypt

## Como rodar localmente

### 1. Pré-requisitos
- Node.js instalado
- Uma connection string de PostgreSQL (Neon é o mais rápido para testar: [neon.tech](https://neon.tech))
- Uma chave de API do Groq ([console.groq.com](https://console.groq.com))

### 2. Configurar variáveis de ambiente

Dentro de `backend/`, crie um arquivo `.env` baseado no `.env.exemplo`:

```
DATABASE_URL="postgresql://usuario:senha@host/neondb?sslmode=require"
GROQ_API_KEY="gsk_sua_chave_aqui"
JWT_SECRET="uma_frase_secreta_qualquer"
PORT=3001
```

### 3. Instalar dependências e preparar o banco

```bash
cd backend
npm install
npx prisma generate
npx prisma migrate dev --name init
```

### 4. Rodar o backend

```bash
npm start
```

O servidor sobe em `http://localhost:3001`.

### 5. Rodar o front-end

Em outro terminal:

```bash
cd frontend
npx serve .
```

Abra o link gerado (geralmente `http://localhost:3000` ou similar) no navegador.

## API (backend)

| Método | Rota                     | O que faz                                  |
|--------|--------------------------|---------------------------------------------|
| POST   | `/api/auth/cadastro`     | Cria uma nova usuária                       |
| POST   | `/api/auth/login`        | Autentica e retorna um token JWT            |
| POST   | `/api/trilha`            | Gera a trilha via IA e salva no banco (rota protegida por JWT) |


## 🚀 Deploy e Infraestrutura (Produção)

A aplicação está no ar e pode ser acessada de qualquer dispositivo através do link seguro: 
**[https://201.23.89.40.sslip.io/](https://201.23.89.40.sslip.io/)**

O deploy da aplicação foi realizado na **Magalu Cloud**, com a infraestrutura configurada do zero. O processo envolveu:

* **Provisionamento de Nuvem:** Criação da Instância (Máquina Virtual Ubuntu) e configuração de uma instância de Banco de Dados (DBaaS PostgreSQL).
* **Rede e Segurança:** Configuração de sub-redes (VPC), regras de firewall (Security Groups e UFW) para liberação das portas HTTP/HTTPS, e acesso remoto seguro via chaves SSH.
* **Gerenciamento de Processos:** Implementação do **PM2** no back-end para manter a API Node.js rodando continuamente em *background*.
* **Proxy Reverso e Roteamento:** Configuração do **Nginx** para servir os arquivos estáticos do front-end e interceptar rotas `/api`, repassando-as internamente para o back-end.
* **Criptografia SSL/TLS:** Emissão e configuração de certificado de segurança utilizando o **Certbot (Let's Encrypt)**, garantindo a comunicação criptografada de ponta a ponta (HTTPS) e redirecionamento automático.

Para produção, a `DATABASE_URL` do `.env` é substituída pela connection string do
DBaaS PostgreSQL da Magalu Cloud (a aplicação, rodando dentro da mesma rede, acessa
o banco via IP privado). Nesse cenário, as migrations são aplicadas com:

```bash
npx prisma migrate deploy
```

# FocusQuest Monorepo

Monorepo para orquestração, execução e avaliação do sistema FocusQuest — uma plataforma experimental de rastreamento ocular baseada em visão computacional. O repositório centraliza os serviços frontend e backend, que são mantidos como Git Submodules, e disponibiliza uma execução reprodutível via Docker Compose para uso acadêmico e de pesquisa.

> Sistema gamificado de avaliação de atenção, integrando visão computacional, WebSockets e Arduino. Uso experimental e acadêmico.

---

## Visão Geral do Projeto

O FocusQuest é uma plataforma que combina uma aplicação cliente gamificada (Next.js) com um servidor de backend (Node.js) para conduzir experimentos de avaliação atencional inspirados no Teste de Desempenho Contínuo (CPT). O sistema captura dados de rastreamento ocular, recebe eventos de um Arduino (botões/entradas físicas), processa métricas cognitivas em tempo real e persiste resultados para análise posterior.

Principais objetivos do monorepo:
- Centralizar a execução do sistema (frontend + backend)
- Facilitar deploy e reprodução experimental com Docker Compose
- Integrar frontend e backend através de Git Submodules para modularidade e controle de versão
- Suportar avaliações acadêmicas e análise de métricas atencionais

---

## Funcionalidades Principais

- Rastreamento ocular (eye-tracking) integrado ao jogo
- Múltiplas fases de atenção (sustentada, seletiva e dividida)
- Integração com Arduino via porta serial (eventos físicos)
- Cálculo de métricas cognitivas: tempo de reação médio, desvio padrão, acertos, omissões, comissões
- Game / dashboard interativo (Next.js) com ranking e áudio imersivo
- Comunicação em tempo real via Socket.IO (WebSockets)
- Persistência e análise com MongoDB (Mongoose)
- Cache/coordenação com Redis
- Execução e orquestração com Docker / Docker Compose

---

## Estrutura do Monorepo

Estrutura de alto nível:

```
focusquest-monorepo/
├── frontend/        # submodule: FocusQuest-web (Next.js)
├── backend/         # submodule: backend-rastreamento-ocular (Node.js)
├── docker-compose.yml
├── .gitmodules
└── README.md
```

O `frontend/` e o `backend/` são mantidos como Git Submodules para permitir desenvolvimento independente e versionamento separado.

---

## Tecnologias Utilizadas

Frontend (submodule `FocusQuest-web`):
- Next.js, React
- TypeScript
- Tailwind CSS
- Framer Motion
- API de WebSockets / Socket.IO no cliente

Backend (submodule `backend-rastreamento-ocular`):
- Node.js, Express
- Socket.IO (servidor)
- MongoDB (Mongoose)
- Redis
- Comunicação serial com Arduino (SerialPort)
- Docker (imagens e integração via Compose)

Infra / Orquestração:
- Docker, Docker Compose
- Git Submodules

Hardware:
- Arduino (dispositivo para inputs físicos: botões, sensores)

---

## Git Submodules — Por que e como

O monorepo usa submodules para manter o frontend e o backend em repositórios separados, preservando histórico e permitindo deploys independentes.

Como clonar corretamente o repositório (incluindo submodules):

```bash
git clone --recurse-submodules https://github.com/Grupo-Lira/focusquest-monorepo.git
```

Se já clonou sem submodules:

```bash
git submodule update --init --recursive
```

---

## Pré-requisitos (ambiente de desenvolvimento)

- Git
- Docker
- Docker Compose
- Node.js (apenas necessário se executar serviços localmente sem Docker)
- Acesso a uma porta serial para conectar o Arduino (se usar hardware)

Recomenda-se executar tudo via Docker Compose para garantir reprodutibilidade entre máquinas.

---

## Como Executar (modo rápido)

1. Clone o monorepo com submodules:

```bash
git clone --recurse-submodules https://github.com/Grupo-Lira/focusquest-monorepo.git
cd focusquest-monorepo
```

2. (Opcional) Inicialize/atualize submodules:

```bash
git submodule update --init --recursive
```

3. Build e suba os serviços com Docker Compose:

```bash
docker compose up --build
```

Ou em background:

```bash
docker compose up --build -d
```

Portas típicas (padrões usados no desenvolvimento):
- Frontend (Next.js): http://localhost:3000
- Backend (Node.js / Socket.IO): http://localhost:4000
- MongoDB: 27017
- Redis: 6379

Confirme os mapeamentos no arquivo `docker-compose.yml` do monorepo (pode variar).

---

## Docker & Arquitetura

- O `docker-compose.yml` centraliza a execução de frontend e backend, além de dependências (MongoDB, Redis).
- Arquitetura facilita reprodução dos experimentos, deploys em ambientes de laboratório e integração contínua.
---

## Fluxo do Sistema

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         FLUXO DE DADOS DO FOCUSQUEST                        │
└─────────────────────────────────────────────────────────────────────────────┘

      CLIENTE                          SERVIDOR                    PERSISTÊNCIA
  ┌─────────────┐     WebSocket      ┌──────────────┐     ┌──────────────────┐
  │  Navegador  │◄────────────────►  │  Backend     │────►│  MongoDB/Redis   │
  │  (Frontend  │    Socket.IO       │  (Node.js)   │     │  (Análise e      │
  │  Next.js)   │                    │  Express     │     │  Armazenamento)  │
  └─────────────┘                    └──────────────┘     └──────────────────┘
        ▲                                    ▲
        │                                    │
        │ Eye-tracker                        │ Serial Port
        │ (Câmera/API)                       │
        │                                    ▼
        │                             ┌──────────────────────┐
        │                             │      Arduino         │
        └─────────────────────────────│  (Botões & Sensores) │
                                      └──────────────────────┘
```

**Fluxo de dados:**
1. Usuário interage com o frontend (Next.js) via navegador
2. Frontend coleta coordenadas de olhar (eye-tracker) e eventos do usuário
3. Dados são enviados ao backend via Socket.IO (WebSocket)
4. Backend recebe eventos, correlaciona com sinais vindos do Arduino (botões, sensores)
5. Backend calcula métricas cognitivas em tempo real
6. Resultados são persistidos no MongoDB e gerenciados pelo Redis
7. Respostas são retornadas ao frontend para feedback ao usuário

---

## Estrutura dos Submodules

- `FocusQuest-web` (frontend): aplicação Next.js gamificada com suporte a rastreamento ocular, game logic, telas de calibração, ranking e integração de áudio.
- `backend-rastreamento-ocular` (backend): servidor Node.js que expõe Socket.IO, conecta com MongoDB e Redis, e integra-se ao Arduino via SerialPort para eventos físicos.

---

## Checklist rápido antes de rodar

- [ ] Clonar com submodules
- [ ] Verificar `docker-compose.yml` para mapeamento de portas
- [ ] Ajustar variáveis de ambiente (ex.: string de conexão do MongoDB, porta serial do Arduino)
- [ ] Executar `docker compose up --build`

---

## Autoria

- Amanda Costa
- Arthur Fudali
- Diego Baltazar
- Giovana Albanês
- Igor Leite

---

## Nota

- Consulte os READMEs específicos dos submodules para instruções detalhadas do frontend e backend.

---


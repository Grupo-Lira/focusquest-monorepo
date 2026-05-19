# FocusQuest Monorepo

Monorepo para orquestração, execução e avaliação do sistema FocusQuest — uma plataforma experimental de rastreamento ocular baseada em visão computacional. O repositório centraliza os serviços frontend e backend, e disponibiliza uma execução reprodutível via Docker Compose para facilitar o desenvolvimento, testes e avaliação acadêmica.

## Visão Geral do Projeto

O FocusQuest é uma plataforma que combina uma aplicação cliente gamificada (Next.js) com um servidor de backend (Node.js) para conduzir experimentos de avaliação atencional inspirados no Teste de Desempenho Contínuo (CPT). O sistema captura dados de rastreamento ocular, recebe eventos de um Arduino (botões/entradas físicas), processa métricas cognitivas em tempo real e persiste resultados para análise posterior.

---

Link do Projeto Hospedado na GCP:
-  https://focusquest-frontend-kxhuw4rwka-uc.a.run.app/ 

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

## Nota

- Este repositório tem como objetivo centralizar a execução, integração e implantação do ecossistema FocusQuest, facilitando a configuração do ambiente para desenvolvimento, testes e avaliação acadêmica.

- Para informações detalhadas sobre arquitetura, funcionalidades e configuração específica para desenvolvimento de cada serviço, consulte os READMEs individuais dos submodules `frontend` e `backend`.


---

## Estrutura do Monorepo

Estrutura de alto nível:

```
focusquest-monorepo/
├── frontend/        # submodule: FocusQuest-web (Next.js)
├── backend/         # submodule: backend-rastreamento-ocular (Node.js)
├── compose.yml
├── .gitmodules
└── README.md
```

Links dos submodules:
- Frontend:
    - Repositório: https://github.com/Grupo-Lira/FocusQuest-web.git
- Backend:
    - Repositório: https://github.com/Grupo-Lira/backend-rastreamento-ocular.git

O `frontend/` e o `backend/` são mantidos como Git Submodules para permitir desenvolvimento independente e versionamento separado.

---

## Tecnologias Utilizadas

Frontend:
- Next.js, React
- TypeScript
- Tailwind CSS
- Framer Motion
- API de WebSockets / Socket.IO no cliente

Backend:
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

Confirme os mapeamentos no arquivo `compose.yml` do monorepo (pode variar).

---

🕹️ Como Jogar
1 - Selecione 'Crie uma conta' (Essa opção é disponibilizada para os doutores).
2 - Preencha os campos 'email', 'senha', 'confirmar senha' e cadastre-se 'Cadastrar'.
3 - Faça o login com os dados da conta criada.
4 - No menu geral escolha a opção 'Calibração' (Sem esse passo as fases não vão funcionar corretamente).
5 - Após finalizar a 'Calibração' as fases estão liberadas, inicie pela fase 1.
6 - Ao iniciar a fase 1 vai ser disponibilizada a opção de Selecionar Paciente.
7 - Se o banco de dados não estiver populado, será preciso criar um perfil de paciente.
8 - No menu, selecione 'Fichas' e escolha a opção 'Criar nova ficha', preencha os campos e confirme.
9 - Após a criação do paciente, selecione a fase 1 e escolha o paciente criado (O sistema entende que é aquele paciente que irá jogar).

Importante: A fase 2 só funciona corretamente se conectada com um arduíno.


Clique em "Iniciar Jornada" na tela inicial.
Inicie no primeiro planeta no menu.
Encontre e fixe o olhar nas estrelas enquanto evita distrações.
Complete o desafio para desbloquear o próximo nível.

---

## Docker & Arquitetura

- O `compose.yml` centraliza a execução de frontend e backend, além de dependências (MongoDB, Redis).
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

- [ ] Possuir uma webcam conectada ao dispositivo.
- [ ] Clonar com submodules
- [ ] Verificar `compose.yml` para mapeamento de portas
- [ ] Ajustar variáveis de ambiente (ex.: string de conexão do MongoDB, porta serial do Arduino)
- [ ] Executar `docker compose up --build`

## Autoria

- Amanda Costa
- Arthur Fudali
- Diego Baltazar
- Giovana Albanês
- Igor Leite

---

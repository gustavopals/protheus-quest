# CODEX.MD — Protheus Quest

> Documento mestre para desenvolvimento orientado por IA (vibe coding).
> Objetivo: permitir que um agente implemente o projeto de ponta a ponta com mínima ambiguidade.

---

## 1) Visão do produto

**Nome:** Protheus Quest
**Tipo:** Browser game de simulação + estratégia + card game técnico
**Tema:** Operação e manutenção de ERP inspirado em cenários reais (Protheus-like)
**Objetivo do jogador:** manter a empresa operando, resolver incidentes e evoluir carreira técnica.

### 1.1 Objetivos de negócio

- Criar um jogo nichado para aprendizado técnico em ERP.
- Habilitar uso em treinamento corporativo e educacional.
- Permitir expansão para multiplayer e ranking.

### 1.2 Objetivos educacionais

- Ensinar arquitetura de ERP (AppServer, DBAccess, Banco).
- Ensinar troubleshooting e priorização técnica.
- Ensinar impacto de parametrização, customização e SQL.

### 1.3 Público-alvo

- Analistas de sistemas ERP (junior a sênior).
- Consultores e estudantes de tecnologia de gestão.
- Times de treinamento de consultorias e universidades.

---

## 2) Escopo do MVP

### 2.1 Escopo **IN** (primeira versão)

- Modo single-player.
- Tabuleiro com 5 módulos: Compras, Estoque, Faturamento, Financeiro, Banco de Dados.
- 20 cartas de ação técnica.
- 10 eventos/incidentes principais.
- Sistema de recursos: Tempo, Conhecimento, Dinheiro, Reputação.
- Missões com solução correta/alternativas e impacto.
- Progressão básica de carreira (Estagiário → Arquiteto).
- Dashboard web com turnos.

### 2.2 Escopo **OUT** (fora do MVP)

- Multiplayer em tempo real.
- PvP assíncrono.
- Loja/economia avançada.
- IA generativa de eventos dinâmicos em produção.

---

## 3) Stack técnica alvo

## 3.1 Monorepo

- `pnpm` workspaces (recomendado).
- Repositório único com frontend + backend + packages compartilhados.

### 3.2 Frontend

- Angular (v19+ recomendado).
- UI: Angular Material **ou** Tailwind + componentes próprios simples.
- Estado: Signals (preferencial) + serviços.
- Roteamento com lazy loading por feature.

### 3.3 Backend

- Node.js (LTS atual).
- Express.js.
- Prisma ORM.
- Zod para validação de input.
- JWT para autenticação (MVP simples).

### 3.4 Banco

- **Produção:** PostgreSQL.
- **Desenvolvimento inicial:** SQLite (opcional, para acelerar setup local).
- Prisma com suporte via variável de ambiente (um provider por schema).

### 3.5 Qualidade e DevEx

- TypeScript em todo o projeto.
- ESLint + Prettier.
- Testes: Vitest/Jest (backend) e Angular Test Runner/Jest (frontend).
- CI: GitHub Actions.
- Docker Compose para infra local (API + Postgres + Adminer opcional).

---

## 4) Estrutura de pastas proposta

```txt
protheus-quest/
  apps/
    web/                      # Angular
      src/
        app/
          core/
          shared/
          features/
            auth/
            game/
            modules/
            cards/
            events/
            profile/
            admin/
    api/                      # Express + Prisma
      src/
        config/
        core/
          http/
          errors/
          logger/
          middleware/
        modules/
          auth/
          users/
          game-sessions/
          modules/
          cards/
          events/
          missions/
          turns/
          leaderboard/
        prisma/
          schema.prisma
          migrations/
          seed.ts
  packages/
    shared-types/             # Tipos e contratos compartilhados
    game-engine/              # Regras puras do jogo (determinísticas)
    config/                   # tsconfig/eslint/prettier compartilhados
  docs/
    adr/                      # Architectural Decision Records
    api/
    game-design/
  .github/
    workflows/
  CODEX.MD
  README.md
  pnpm-workspace.yaml
```

---

## 5) Arquitetura funcional

### 5.1 Componentes

- **Web App (Angular):** interface de jogo, decisões de turno, visualização de status.
- **API (Express):** orquestração de sessão, persistência e autenticação.
- **Game Engine (package):** regras de cálculo de impacto, resolução de eventos e progressão.
- **Banco (Prisma):** armazenamento de usuários, sessões, estado de jogo, cartas e eventos.

### 5.2 Princípios de arquitetura

- Regras de domínio **fora** do controller HTTP.
- Engine determinística e testável (sem dependência de Express/Prisma).
- Contratos DTO versionados.
- Feature modules com responsabilidade clara.

### 5.3 Fluxo principal de turno

1. Jogador inicia/continua sessão.
2. API gera incidentes/eventos do turno com seed pseudoaleatória.
3. Frontend exibe opções (cartas/ações).
4. Jogador escolhe ação.
5. Engine resolve impacto em recursos + módulos + progresso.
6. API persiste estado e retorna resultado narrativo + numérico.

---

## 6) Modelo de domínio (alto nível)

### 6.1 Entidades principais

- `User`
- `Profile`
- `GameSession`
- `Turn`
- `ModuleState` (Compras/Estoque/Faturamento/...)
- `Card`
- `Deck`
- `Event`
- `Mission`
- `Decision`
- `Outcome`
- `Skill`
- `Inventory` (cartas/habilidades)

### 6.2 Recursos globais da sessão

- `time` (0-100)
- `knowledge` (0-100)
- `budget` (0-100)
- `reputation` (0-100)

Condição de derrota (MVP sugerida): qualquer recurso <= 0.
Condição de vitória: completar X missões críticas + manter recursos mínimos por Y turnos.

---

## 7) Prisma ORM — especificação inicial

## 7.1 Estratégia de banco

### Opção A (recomendada): Postgres desde o início

- Menos retrabalho de migração.
- Melhor para evolução multiplayer/ranking.

### Opção B: SQLite no início + migração para Postgres

- Mais rápido para protótipo local.
- Exige plano de transição validado cedo.

## 7.2 Modelos iniciais (MVP)

- `User(id, email, passwordHash, role, createdAt)`
- `GameSession(id, userId, status, currentTurn, difficulty, seed, createdAt, updatedAt)`
- `SessionResources(id, sessionId, time, knowledge, budget, reputation)`
- `ModuleState(id, sessionId, moduleType, health, riskLevel, lastIncidentAt)`
- `Card(id, code, name, category, rarity, basePower, riskModifier, description)`
- `SessionCard(id, sessionId, cardId, quantity, cooldownTurns)`
- `Event(id, code, title, moduleType, severity, payloadJson)`
- `Turn(id, sessionId, number, startedAt, finishedAt)`
- `Decision(id, turnId, eventId, chosenCardId, chosenAction, responseTimeMs)`
- `Outcome(id, decisionId, deltaTime, deltaKnowledge, deltaBudget, deltaReputation, notes)`
- `Mission(id, code, title, moduleType, objectiveJson, rewardJson, penaltyJson)`
- `SessionMission(id, sessionId, missionId, status, progressJson)`

## 7.3 Índices e constraints

- Unique: `User.email`, `Card.code`, `Event.code`, `Mission.code`.
- Índices: `GameSession.userId`, `Turn.sessionId + number`, `ModuleState.sessionId + moduleType`.
- Integridade: soft delete apenas se necessário, priorizar consistência referencial.

## 7.4 Seeds

- Seed de 20 cartas base.
- Seed de 10 eventos base.
- Seed de 10 missões base.
- Seed de dificuldades (`easy`, `normal`, `hard`).

---

## 8) Regras de jogo (engine)

### 8.1 Fórmula de impacto (exemplo base)

```txt
impactScore = (card.basePower + knowledgeFactor + moduleAffinity) - (event.severity * riskPenalty)
```

Onde:

- `knowledgeFactor = floor(knowledge / 10)`
- `moduleAffinity = +10` se carta combina com módulo, senão `0`
- `riskPenalty = 1 + (moduleRiskLevel / 100)`

### 8.2 Resultado por faixa

- `impactScore >= 60` → sucesso excelente
- `30..59` → sucesso parcial
- `10..29` → mitigação mínima
- `< 10` → falha com efeito colateral

### 8.3 Efeitos colaterais possíveis

- Quebra em módulo adjacente.
- Aumento de risco no próximo turno.
- Redução de reputação por atraso.

### 8.4 Progressão de carreira

- XP por decisão correta + missão concluída.
- Níveis:
  - 1 Estagiário
  - 2 Analista Jr
  - 3 Analista Pl
  - 4 Consultor
  - 5 Arquiteto ERP

Desbloqueios por nível: novas cartas, redução de penalidades, ações avançadas.

---

## 9) Especificação de API (REST)

Base URL: `/api/v1`

### 9.1 Auth

- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/refresh`
- `GET /auth/me`

### 9.2 Sessão de jogo

- `POST /sessions` cria sessão
- `GET /sessions/:id` detalhes da sessão
- `POST /sessions/:id/start-turn` inicia turno
- `POST /sessions/:id/decisions` envia decisão
- `POST /sessions/:id/end-turn` fecha turno
- `GET /sessions/:id/state` estado resumido

### 9.3 Catálogo

- `GET /cards`
- `GET /events`
- `GET /missions`
- `GET /modules`

### 9.4 Leaderboard

- `GET /leaderboard/global`
- `GET /leaderboard/friends` (pós-MVP)

### 9.5 Padrão de resposta

```json
{
  "success": true,
  "data": {},
  "meta": { "requestId": "..." },
  "error": null
}
```

### 9.6 Erros

- `400` validação
- `401` não autenticado
- `403` sem permissão
- `404` não encontrado
- `409` conflito de estado
- `422` regra de domínio inválida
- `500` erro interno

---

## 10) Frontend Angular — blueprint

### 10.1 Rotas

- `/login`
- `/dashboard`
- `/session/:id`
- `/session/:id/turn`
- `/cards`
- `/missions`
- `/profile`
- `/admin` (feature flag)

### 10.2 Estrutura por feature

- `features/game`:
  - `session-shell`
  - `resource-panel`
  - `module-board`
  - `incident-card`
  - `action-hand`
  - `turn-log`

### 10.3 Estado de UI

- `authState`
- `sessionState`
- `turnState`
- `catalogState`

### 10.4 UX mínima do MVP

- Layout dashboard corporativo simples.
- Painel de recursos sempre visível.
- Lista de incidentes ativos com severidade.
- Mão de cartas com cooldown e risco.
- Feedback imediato após decisão (texto + impacto numérico).

---

## 11) Segurança e conformidade

- Senhas com hash (`argon2` recomendado).
- JWT com expiração curta + refresh token.
- Rate limit em auth e endpoints críticos.
- Sanitização e validação rigorosa no backend.
- CORS restrito por ambiente.
- Logs sem dados sensíveis.

---

## 12) Observabilidade e operação

- Logging estruturado (`pino` recomendado).
- `requestId` por requisição.
- Health endpoints:
  - `GET /health/live`
  - `GET /health/ready`
- Métricas mínimas:
  - latência p95
  - erro por endpoint
  - taxa de conclusão de turno

---

## 13) Testes e qualidade

### 13.1 Pirâmide de testes

- Unitário (engine e regras): **alta prioridade**.
- Integração (API + Prisma test DB): prioridade média/alta.
- E2E (fluxo login → turno): prioridade média.

### 13.2 Casos críticos para automatizar

- Resolução de incidente por faixa de impacto.
- Penalidade por carta incorreta.
- Progressão de nível por XP.
- Integridade de transação em fim de turno.
- Autorização por usuário/sessão.

### 13.3 Qualidade de código

- Cobertura alvo inicial: 70% (engine acima de 85%).
- Lint e type-check obrigatórios no CI.
- Conventional Commits recomendados.

---

## 14) DevOps e ambientes

### 14.1 Ambientes

- `local`
- `staging`
- `production`

### 14.2 Variáveis de ambiente (API)

- `NODE_ENV`
- `PORT`
- `DATABASE_URL`
- `JWT_ACCESS_SECRET`
- `JWT_REFRESH_SECRET`
- `CORS_ORIGIN`
- `LOG_LEVEL`

### 14.3 Docker Compose local

Serviços sugeridos:

- `api`
- `db` (postgres)
- `adminer` (opcional)

### 14.4 CI pipeline mínima

1. install
2. lint
3. typecheck
4. test
5. build
6. (staging) migrate + deploy

---

## 15) Roadmap por fases (início ao fim)

## Fase 0 — Foundation (Dia 1-2)

- Criar monorepo.
- Setup lint/format/tsconfig.
- Setup Angular app + Express API.
- Setup Prisma + primeira migração.
- CI básico.

**DoD:** projeto sobe local com `web` e `api`.

## Fase 1 — Núcleo do jogo (Dia 3-7)

- Implementar entidades principais no backend.
- Implementar package `game-engine` com regras puras.
- Criar endpoints de sessão/turno/decisão.
- Seed inicial de cartas/eventos/missões.

**DoD:** jogo executa 1 turno completo via API.

## Fase 2 — Interface jogável (Semana 2)

- Dashboard de sessão.
- Render de incidentes + cartas.
- Decisão e feedback visual.
- Histórico básico do turno.

**DoD:** usuário joga loop completo no browser.

## Fase 3 — Progressão e balanceamento (Semana 3)

- XP, níveis e desbloqueios.
- Ajuste de dificuldade (`easy/normal/hard`).
- Missões com estados e recompensas.

**DoD:** progressão funcional e minimamente balanceada.

## Fase 4 — Robustez (Semana 4)

- Testes críticos automatizados.
- Melhorias de logs e healthcheck.
- Hardening de auth e validações.

**DoD:** pronto para staging.

## Fase 5 — Pré-lançamento (Semana 5)

- UX polish mínimo.
- Leaderboard global simples.
- Deploy staging + checklist de release.

**DoD:** release candidata validada.

## Fase 6 — Pós-MVP

- Multiplayer assíncrono.
- Editor de cartas/eventos para admin.
- Modo desafio diário.

---

## 16) Backlog inicial (priorizado)

### Epic A — Plataforma técnica

- A1: Monorepo com workspaces.
- A2: Pipeline CI.
- A3: Docker local.
- A4: Config de ambiente por stage.

### Epic B — Domínio do jogo

- B1: Modelo Prisma inicial.
- B2: Engine de resolução de eventos.
- B3: Sistema de recursos e derrota/vitória.
- B4: Missões e progressão.

### Epic C — Experiência do usuário

- C1: Login/registro.
- C2: Dashboard de sessão.
- C3: Turno interativo.
- C4: Tela de resultado.

### Epic D — Qualidade e operação

- D1: Testes unitários engine.
- D2: Integração API.
- D3: Observabilidade.
- D4: Hardening segurança.

---

## 17) ADRs sugeridos (Architecture Decision Records)

Criar documentos em `docs/adr`:

- ADR-001: Monorepo com `pnpm`.
- ADR-002: Angular + Express (vs alternativas).
- ADR-003: Engine separada em package compartilhado.
- ADR-004: Postgres como alvo de produção.
- ADR-005: JWT stateless no MVP.

---

## 18) Guia de implementação para agente de IA (vibe coding)

### 18.1 Regras de execução

- Sempre iniciar por task pequena, testável e com DoD claro.
- Nunca alterar múltiplos módulos sem checklist.
- Antes de editar: mapear impacto (`files`, `tests`, `contracts`).
- Após editar: rodar lint + testes relevantes.

### 18.2 Template de tarefa para IA

```md
## Task
[descrição objetiva]

## Contexto
[módulo/arquivo/restrições]

## Critérios de aceite
- [ ] critério 1
- [ ] critério 2

## Não fazer
- [limites claros]

## Verificação
- comando 1
- comando 2
```

### 18.3 Prompt padrão para feature

```txt
Implemente a feature [NOME] no módulo [X] respeitando arquitetura modular.
Requisitos: [lista].
Não altere contratos existentes sem versionamento.
Adicione/atualize testes unitários e de integração necessários.
Execute lint, typecheck e testes do módulo alterado.
Retorne: resumo das mudanças, arquivos alterados, riscos e próximos passos.
```

### 18.4 Prompt padrão para correção de bug

```txt
Investigue e corrija o bug [ID/Título].
1) reproduza localmente,
2) identifique causa raiz,
3) aplique correção mínima,
4) adicione teste que falhava antes e passa após fix.
Não faça refatorações fora do escopo.
```

### 18.5 Política de PR

- PR pequeno (ideal < 400 linhas líquidas).
- 1 objetivo por PR.
- Checklist obrigatório de teste e risco.
- Descrever rollback simples.

---

## 19) Convenções de código

- TypeScript strict.
- Nomes explícitos (sem abreviações ambíguas).
- Controllers finos, services ricos, engine pura.
- DTO separado de entidades persistidas.
- Erros de domínio tipados.

Padrão de commit sugerido:

- `feat(game): add turn resolution service`
- `fix(api): validate invalid mission transition`
- `test(engine): cover impact score edge cases`

---

## 20) Plano de dados de conteúdo (cartas/eventos)

### 20.1 Categorias de cartas

- SQL
- ADVPL
- Configuração
- Suporte
- Infraestrutura

### 20.2 Severidade de eventos

- Low
- Medium
- High
- Critical

### 20.3 Exemplo de carta (JSON)

```json
{
  "code": "CARD_ADVPL_PATCH",
  "name": "Customização ADVPL",
  "category": "ADVPL",
  "basePower": 50,
  "riskModifier": 10,
  "description": "Resolve problema crítico, mas aumenta risco futuro."
}
```

### 20.4 Exemplo de evento (JSON)

```json
{
  "code": "EVT_DBACCESS_DOWN",
  "title": "DBAccess caiu",
  "moduleType": "DATABASE",
  "severity": "HIGH",
  "payload": {
    "deadlineTurns": 2,
    "possibleActions": ["restart_service", "restart_server", "open_vendor_ticket"]
  }
}
```

---

## 21) Estratégia de balanceamento

- Rodadas de teste com seed fixa para comparabilidade.
- Ajustar pesos por telemetria interna:
  - taxa de vitória por dificuldade
  - taxa de sucesso por categoria de carta
  - causas mais frequentes de derrota
- Alvo inicial: 55-65% de win rate no modo normal para usuário iniciante.

---

## 22) Internacionalização e conteúdo

- Idioma inicial: Português-BR.
- Estruturar textos para i18n desde início (`pt-BR`, `en-US` futuro).
- Não hardcode de strings em componentes críticos.

---

## 23) Riscos e mitigação

- **Risco:** escopo crescer cedo demais.
  **Mitigação:** travar escopo do MVP por 5 semanas.
- **Risco:** engine acoplada ao backend.
  **Mitigação:** package isolada + testes unitários fortes.
- **Risco:** retrabalho de banco por troca SQLite→Postgres.
  **Mitigação:** decidir até fim da Fase 1.
- **Risco:** UX complexa demais no início.
  **Mitigação:** dashboard simples e funcional.

---

## 24) Checklist de pronto para produção (MVP)

- [ ] Build e testes verdes no CI.
- [ ] Migrações versionadas e reproduzíveis.
- [ ] Health checks funcionando.
- [ ] Logs e erros padronizados.
- [ ] Métricas mínimas coletadas.
- [ ] Política de backup/restauração definida.
- [ ] Documentação de operação em `docs/`.

---

## 25) Próximos passos imediatos (ordem recomendada)

1. Criar o novo repositório monorepo com `apps/web` e `apps/api`.
2. Subir base Angular + Express + Prisma.
3. Definir banco do MVP (Postgres recomendado).
4. Implementar `game-engine` com 3 eventos e 5 cartas primeiro.
5. Integrar UI mínima para ciclo de 1 turno.
6. Expandir para catálogo completo (20 cartas / 10 eventos).

---

## 26) Definição de sucesso do projeto

- MVP jogável e estável em browser.
- Usuário consegue iniciar sessão, jogar turnos e concluir missão.
- Regras reproduzíveis e testadas.
- Arquitetura preparada para evolução sem reescrita.

---

## 27) Observação de domínio

Este jogo é inspirado em cenários de ERP e troubleshooting corporativo.
Evitar uso de marcas, assets ou conteúdos protegidos sem autorização formal no material final distribuído.

---

## 28) Comandos de referência (quando o repo existir)

```bash
pnpm install
pnpm -r build
pnpm -r test
pnpm --filter api prisma migrate dev
pnpm --filter api prisma db seed
pnpm --filter web start
pnpm --filter api dev
```

---

## 29) Modo de trabalho recomendado (vibe coding)

- Ciclos curtos de 30-90 minutos.
- Sempre com objetivo único e mensurável.
- Ao fim de cada ciclo:
  - atualizar changelog técnico,
  - registrar decisão arquitetural,
  - criar próxima task atômica.

---

**Fim do documento mestre.**
Use este arquivo como fonte primária de contexto para agentes de IA, planejamento técnico e execução do roadmap.

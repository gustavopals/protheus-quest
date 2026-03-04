# CODEX.MD - Protheus Quest

Documento mestre para orientar desenvolvimento humano + agente de IA com baixa ambiguidade.
Objetivo: permitir execução ponta a ponta de um novo repositório com front Angular, backend Node.js/Express, Prisma ORM e banco PostgreSQL ou SQLite no início.

## 1. Como usar este documento

- Este arquivo eh a fonte principal de contexto tecnico e de produto.
- Use junto com [chat-game-ideia.md](/opt/protheus-quest/chat-game-ideia.md) para regras de design do jogo e desafios Protheus.
- Em cada task de implementacao, copiar a secao `Template de Task para IA` (secao 20).
- Evitar tarefas grandes sem critero de aceite e sem DoD.

## 2. Visao de produto

### 2.1 Proposta

Protheus Quest eh um jogo web de estrategia tecnica onde o jogador administra incidentes de ERP em turnos curtos.

### 2.2 Objetivo do usuario final

- Se divertir com decisoes de alto impacto tecnico.
- Aprender troubleshooting em cenarios inspirados no ecossistema Protheus.
- Evoluir de suporte basico para arquiteto ERP.

### 2.3 Objetivo de negocio

- Entregar MVP jogavel em browser em ate 5-8 semanas.
- Criar base escalavel para conteudo educacional e desafios recorrentes.

### 2.4 Objetivo educacional

- Ensinar conexoes entre modulos (Compras, Estoque, Faturamento, Financeiro, Fiscal).
- Ensinar diagnostico tecnico (parametros, regras, integracoes, banco e performance).
- Ensinar priorizacao sob restricao de tempo e recursos.

## 3. Principios de arquitetura

- Separacao clara entre `API HTTP`, `dominio do jogo` e `persistencia`.
- Regras da engine devem ser deterministicas e testaveis sem Express.
- Contratos de API estaveis e versionados (`/api/v1`).
- Backend stateless (JWT), com estado de jogo persistido em banco.
- Frontend com estado previsivel (Signals + services por feature).
- Adotar simplicidade no MVP e evitar over-engineering.

## 4. Escopo funcional

### 4.1 MVP (in)

- Cadastro/login.
- Sessao de jogo single-player.
- Loop de turno completo: incidente -> decisao -> resultado -> persistencia.
- 5 modulos de jogo: Compras, Estoque, Faturamento, Financeiro, Banco.
- 20 cartas base.
- 30 desafios/eventos base.
- 3 dificuldades (`easy`, `normal`, `hard`).
- Progressao de nivel com desbloqueio de cartas.
- Historico de turnos e score final.

### 4.2 Fora do MVP (out)

- Multiplayer realtime.
- IA generativa para criar eventos em producao.
- Marketplace de cartas.
- Editor visual completo de regras.

## 5. Stack e versoes alvo

### 5.1 Runtime

- Node.js LTS (22+ recomendado).
- pnpm 9+.
- TypeScript strict.

### 5.2 Frontend

- Angular 19+.
- Angular Router com lazy loading.
- Angular Signals para estado local por feature.
- UI: Angular Material ou componentes custom simples.

### 5.3 Backend

- Express 5+.
- Prisma ORM.
- Zod para validacao.
- JWT + refresh token.
- Pino para logs.

### 5.4 Banco

- Producao: PostgreSQL.
- Inicio rapido opcional: SQLite.

### 5.5 Qualidade

- ESLint + Prettier.
- Vitest/Jest para backend e engine.
- Testes do Angular para frontend.
- GitHub Actions para CI.

## 6. Estrutura de repositorio (monorepo)

```txt
protheus-quest/
  apps/
    web/                         # Angular
    api/                         # Express + Prisma
  packages/
    game-engine/                 # Regras deterministicas do jogo
    shared-types/                # DTOs, enums e tipos compartilhados
    config/                      # configs comuns (tsconfig/eslint/prettier)
  docs/
    adr/
    api/
    game-design/
  docker/
    compose/
  scripts/
  CODEX.MD
  chat-game-ideia.md
  README.md
  pnpm-workspace.yaml
```

## 7. Decisoes importantes (ADR resumido)

- ADR-001: Monorepo com pnpm workspaces.
- ADR-002: Angular + Express por velocidade de entrega e curva de aprendizado.
- ADR-003: Engine de jogo isolada em package compartilhado.
- ADR-004: PostgreSQL como destino final de producao.
- ADR-005: SQLite permitido apenas para bootstrap local rapido.

## 8. Estrategia de banco (Postgres ou SQLite)

### 8.1 Recomendacao pratica

Se o time puder rodar Docker local, comecar direto em PostgreSQL para reduzir retrabalho.

### 8.2 Quando usar SQLite no inicio

- Protótipo muito inicial.
- Sem infraestrutura local.
- Time pequeno validando mecanicas.

### 8.3 Plano de transicao SQLite -> Postgres

1. Congelar mudancas de schema.
2. Exportar dados de seed e fixtures.
3. Ajustar `schema.prisma` para provider PostgreSQL.
4. Gerar novas migracoes.
5. Reaplicar seed em Postgres.
6. Validar fluxo completo com smoke test.

### 8.4 Convencao recomendada para schema

- `apps/api/prisma/schema.prisma` (ativo).
- `apps/api/prisma/schema.sqlite.prisma` (backup de bootstrap, opcional).

## 9. Modelo de dominio (MVP)

### 9.1 Entidades principais

- `User`: identidade e autenticacao.
- `Profile`: progresso global do jogador.
- `GameSession`: estado de uma partida.
- `SessionResource`: recursos da sessao (tempo, conhecimento, dinheiro, reputacao).
- `ModuleState`: estado dos modulos do ERP por sessao.
- `Turn`: unidade temporal de jogo.
- `Event`: incidente/desafio apresentado ao jogador.
- `Card`: acao tecnica disponivel.
- `Decision`: escolha do jogador no turno.
- `Outcome`: resultado calculado pela engine.
- `Mission`: objetivos compostos.
- `SessionMission`: progresso por missao.

### 9.2 Recursos globais

- `time` (0-100)
- `knowledge` (0-100)
- `budget` (0-100)
- `reputation` (0-100)

Derrota: qualquer recurso <= 0.
Vitoria baseline: completar N missoes criticas sem perder sessao.

### 9.3 Modulos de jogo

- `PURCHASES`
- `INVENTORY`
- `BILLING`
- `FINANCE`
- `DATABASE`

## 10. Regras da game engine

### 10.1 Requisitos da engine

- Pura (sem acesso a banco, sem chamadas HTTP).
- Deterministica por `seed` + `state` + `action`.
- Testavel com snapshots e casos de borda.

### 10.2 Formula base de impacto

```txt
impact = (cardPower + knowledgeBonus + moduleAffinity + comboBonus)
         - (eventSeverityWeight + moduleRiskPenalty + randomVariance)
```

Definicoes:
- `knowledgeBonus = floor(knowledge / 12)`
- `moduleAffinity = +8` quando carta combina com modulo alvo
- `randomVariance = valor pseudoaleatorio baseado em seed (-5..+5)`

### 10.3 Faixas de resultado

- `>= 65`: sucesso excelente.
- `40-64`: sucesso parcial controlado.
- `20-39`: mitigacao minima.
- `< 20`: falha com efeito colateral.

### 10.4 Efeito colateral possivel

- Aumento de risco no modulo atual.
- Impacto negativo em modulo adjacente.
- Perda de reputacao por atraso.

### 10.5 Progressao de carreira

- XP por turno resolvido + bonus por eficiencia.
- Niveis: Estagiario -> Analista Jr -> Analista Pl -> Consultor -> Arquiteto.
- Cada nivel desbloqueia cartas, perks e reducao de penalidade.

## 11. API REST (contrato inicial)

Base URL: `/api/v1`

### 11.1 Auth

- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/refresh`
- `GET /auth/me`

### 11.2 Sessao

- `POST /sessions`
- `GET /sessions/:id`
- `POST /sessions/:id/start-turn`
- `POST /sessions/:id/decisions`
- `POST /sessions/:id/end-turn`
- `GET /sessions/:id/state`

### 11.3 Catalogo

- `GET /cards`
- `GET /events`
- `GET /missions`
- `GET /modules`

### 11.4 Ranking

- `GET /leaderboard/global`

### 11.5 Formato padrao de resposta

```json
{
  "success": true,
  "data": {},
  "meta": {
    "requestId": "req_123"
  },
  "error": null
}
```

### 11.6 Erros padrao

- `400` validacao
- `401` autenticacao
- `403` autorizacao
- `404` recurso nao encontrado
- `409` conflito de estado
- `422` regra de dominio invalida
- `500` erro interno

## 12. Backend: organizacao por modulo

```txt
apps/api/src/
  config/
  core/
    errors/
    http/
    logger/
    middleware/
  modules/
    auth/
    users/
    sessions/
    turns/
    events/
    cards/
    missions/
    leaderboard/
  prisma/
```

### 12.1 Padrao interno por modulo

- `*.route.ts`
- `*.controller.ts`
- `*.service.ts`
- `*.repository.ts` (se necessario)
- `*.schema.ts` (zod)
- `*.mapper.ts` (DTO)
- `*.spec.ts`

### 12.2 Regras de implementacao

- Controller fino (so parse + response).
- Service com regra de negocio.
- Repositorio isolando acesso Prisma quando o modulo crescer.
- Nenhuma regra de engine dentro de controller.

## 13. Frontend Angular: blueprint

### 13.1 Rotas

- `/login`
- `/dashboard`
- `/session/:id`
- `/session/:id/turn`
- `/cards`
- `/missions`
- `/profile`

### 13.2 Estrutura por feature

```txt
apps/web/src/app/
  core/
  shared/
  features/
    auth/
    dashboard/
    game/
      session-shell/
      module-board/
      incident-panel/
      card-hand/
      turn-result/
      turn-log/
    cards/
    missions/
    profile/
```

### 13.3 Estado de tela (signals)

- `authState`
- `sessionState`
- `turnState`
- `catalogState`
- `profileState`

### 13.4 UX do MVP

- Dashboard limpo e legivel.
- Barras de recursos sempre visiveis.
- Incidente com severidade destacada.
- Escolha de carta em 1 clique.
- Resultado textual + numerico imediato.

## 14. Conteudo inicial (seed)

- 20 cartas divididas em categorias: SQL, ADVPL, Configuracao, Suporte, Infra.
- 30 eventos/desafios inspirados em operacao ERP.
- 15 missoes iniciais (com dificuldade progressiva).
- 3 perfis de dificuldade global.

### 14.1 Estrategia de dados

- Conteudo em JSON versionado (`apps/api/src/content/*.json`).
- Seed Prisma le JSON e popula tabelas.
- IDs por `code` estavel (nao por nome exibido).

## 15. Seguranca

- Password hash com Argon2.
- JWT access curto e refresh controlado.
- Rate limit em login e endpoints sensiveis.
- CORS por ambiente (whitelist).
- Sanitizacao e validacao com Zod.
- Logs sem segredos.

## 16. Observabilidade e operacao

- Logger estruturado (Pino).
- `requestId` por requisicao.
- Health endpoints:
  - `GET /health/live`
  - `GET /health/ready`
- Metricas minimas:
  - latencia p95 por rota
  - erro por rota
  - taxa de conclusao de turno

## 17. Estrategia de testes

### 17.1 Piramide

- Unitario: engine e regras de dominio (prioridade maxima).
- Integracao: API + Prisma + banco de teste.
- E2E: login -> iniciar sessao -> turno -> resultado.

### 17.2 Cobertura alvo

- Geral: 70%+
- `game-engine`: 85%+

### 17.3 Casos obrigatorios

- Faixas de impacto da engine.
- Falha por recurso zerado.
- Persistencia de turno completa.
- Autorizacao (usuario nao acessa sessao alheia).
- Erro de contrato invalido retorna status correto.

## 18. CI/CD e ambientes

### 18.1 Ambientes

- `local`
- `staging`
- `production`

### 18.2 Pipeline minima

1. install
2. lint
3. typecheck
4. test
5. build
6. migrate (staging/prod)
7. deploy

### 18.3 Variaveis de ambiente (api)

- `NODE_ENV`
- `PORT`
- `DATABASE_URL`
- `JWT_ACCESS_SECRET`
- `JWT_REFRESH_SECRET`
- `CORS_ORIGIN`
- `LOG_LEVEL`

## 19. Roadmap por etapas (inicio ao fim)

## Etapa 0 - Setup do repo (Dia 1)

- Criar monorepo pnpm.
- Configurar `apps/web` e `apps/api`.
- Lint/format/typecheck padrao.

DoD:
- comando de build e lint funcionando em ambos apps.

## Etapa 1 - Banco e autenticacao (Dia 2-3)

- Definir provider inicial (Postgres ou SQLite).
- Modelos `User`, `Profile` e auth JWT.
- Migração e seed minima.

DoD:
- registro/login funcionando + teste de integracao verde.

## Etapa 2 - Dominio base do jogo (Dia 4-7)

- Modelos de sessao, turno, carta, evento, decisao, outcome.
- Primeira versao da game-engine em package.
- Endpoints de sessao e decisao.

DoD:
- ciclo de 1 turno funcionando via API.

## Etapa 3 - Front jogavel (Semana 2)

- Dashboard de sessao.
- Tela de turno e escolha de carta.
- Exibicao de resultado e historico.

DoD:
- usuario completo consegue jogar no browser.

## Etapa 4 - Conteudo e balanceamento (Semana 3)

- Expandir para 20 cartas, 30 eventos e 15 missoes.
- Ajustar pesos e risco por telemetria local.

DoD:
- dificuldade normal com taxa de vitoria inicial alvo (50-65%).

## Etapa 5 - Robustez e qualidade (Semana 4)

- Testes criticos, logs estruturados e health checks.
- Hardening de validacoes e autorizacao.

DoD:
- pipeline CI verde e smoke test estavel.

## Etapa 6 - Pre-lancamento (Semana 5-6)

- Polimento UX.
- Leaderboard global simples.
- Deploy em staging com checklist de release.

DoD:
- release candidate validada.

## Etapa 7 - Pos-MVP

- Multiplayer assincrono.
- Desafio diario.
- Editor interno de eventos/cartas.

## 20. Vibe coding: regras para agente de IA

### 20.1 Modo de execucao

- Sempre executar tarefas atomicas.
- Nunca misturar feature + refactor amplo no mesmo PR.
- Sempre incluir impacto, riscos e validacao no output.
- Rodar testes relevantes antes de finalizar task.

### 20.2 Template de Task para IA

```md
## Tarefa
[objetivo claro]

## Contexto
[arquivos, modulo e restricoes]

## Criterios de aceite
- [ ] criterio 1
- [ ] criterio 2
- [ ] criterio 3

## Fora de escopo
- [item 1]
- [item 2]

## Validacao
- [comando 1]
- [comando 2]
```

### 20.3 Prompt padrao para feature

```txt
Implemente a feature [NOME] no modulo [X] seguindo a arquitetura descrita no CODEX.MD.
Respeite contratos existentes e adicione testes unitarios/integracao necessarios.
Retorne: resumo tecnico, arquivos alterados, comandos executados, riscos e proximos passos.
```

### 20.4 Prompt padrao para bugfix

```txt
Investigue o bug [ID/Titulo], reproduza localmente, identifique causa raiz,
aplique correcao minima e adicione teste de regressao.
Nao faca refatoracoes fora do escopo.
```

### 20.5 Prompt padrao para code review

```txt
Revise as mudancas com foco em regressao funcional, risco de seguranca,
consistencia de contratos e cobertura de testes.
Liste findings por severidade, com arquivo e linha.
```

## 21. Definicoes operacionais

### 21.1 Definition of Ready (DoR)

Uma task so inicia quando tiver:
- objetivo claro
- escopo limitado
- criterio de aceite testavel
- arquivos/modulos alvo identificados

### 21.2 Definition of Done (DoD)

Uma task so encerra quando tiver:
- implementacao completa
- testes relevantes passando
- lint/typecheck ok
- documentacao minima atualizada
- risco residual declarado

## 22. Backlog inicial priorizado

### Epic A - Plataforma

- A1: bootstrap monorepo.
- A2: CI baseline.
- A3: docker compose para api + db.

### Epic B - Core game

- B1: modelos Prisma base.
- B2: engine de impacto e colateral.
- B3: endpoints de turno e decisao.

### Epic C - UX jogavel

- C1: auth e dashboard.
- C2: modulo de turno.
- C3: resultado e historico.

### Epic D - Conteudo

- D1: pack inicial de cartas/eventos/missoes.
- D2: balanceamento por seed.

### Epic E - Confiabilidade

- E1: testes de regressao.
- E2: observabilidade minima.
- E3: checklist de release.

## 23. Convencoes de codigo e commits

- TypeScript strict sem `any` desnecessario.
- Controllers finos, services explicitos.
- Erros de dominio tipados.
- Commits no padrao Conventional Commits.

Exemplos:
- `feat(engine): add turn impact calculator`
- `fix(api): prevent session ownership leak`
- `test(web): cover turn outcome rendering`

## 24. Riscos e mitigacoes

- Escopo crescer cedo demais.
  Mitigacao: congelar MVP ate etapa 6.
- Acoplamento da engine ao backend.
  Mitigacao: manter engine em package puro + testes.
- Migracao de banco custosa.
  Mitigacao: decidir provider final no maximo ate fim da etapa 2.
- UX confusa.
  Mitigacao: simplificar telas e medir tempo de decisao por turno.

## 25. Checklist de release MVP

- [ ] Build, lint e testes verdes no CI.
- [ ] Migracoes aplicaveis em ambiente limpo.
- [ ] Health checks e logs ativos.
- [ ] Fluxo completo de jogo validado manualmente.
- [ ] Documentacao de operacao e rollback pronta.

## 26. Comandos de referencia

```bash
pnpm install
pnpm -r lint
pnpm -r typecheck
pnpm -r test
pnpm -r build
pnpm --filter api prisma migrate dev
pnpm --filter api prisma db seed
pnpm --filter api dev
pnpm --filter web start
```

## 27. Proximos passos recomendados agora

1. Criar o monorepo com estrutura da secao 6.
2. Subir `apps/api` com auth e Prisma.
3. Decidir provider inicial (Postgres recomendado).
4. Implementar primeira versao da engine com 5 cartas e 8 eventos.
5. Integrar primeiro loop jogavel no Angular.

Fim.

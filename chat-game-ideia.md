# chat-game-ideia.md - Design do jogo Protheus Quest

Guia de design para um jogo divertido, simples de desenvolver e com forte aprendizado tecnico.
Complementa o [CODEX.MD](/opt/protheus-quest/CODEX.MD) com foco em gameplay, narrativa e desafios.

## 1. Norte do jogo

## 1.1 Objetivo central

Criar um jogo de estrategia em turnos curtos onde o jogador resolve incidentes de ERP e aprende raciocinio tecnico sem ficar preso em uma interface complexa.

## 1.2 Pilar de design

- Simples de jogar em 2-5 minutos por sessao.
- Dificil de dominar no medio prazo.
- Recompensador para quem aprende as causas reais dos problemas.
- Visual leve e funcional (graficos simples).

## 1.3 Experiencia desejada

- Diversao imediata pela pressao de tempo e escolhas de risco.
- Sensacao de progresso por desbloqueios.
- Aprendizado por repeticao contextualizada.

## 2. Formato do jogo

- Plataforma: browser.
- Genero: estrategia por turnos + card tactics.
- Sessao curta: 10 a 20 turnos por partida.
- Publico: analistas, consultores, estudantes de ERP.

## 3. Direcao visual (grafico simples)

## 3.1 Regra geral

Focar em dashboard 2D limpo. Nada de mapa 3D, animacoes pesadas ou sistemas de combate complexos.

## 3.2 Estilo visual recomendado

- Layout de painel corporativo.
- Cartoes de modulo com icones simples.
- Barras de recurso horizontais.
- Cores de severidade:
  - verde (ok)
  - amarelo (alerta)
  - vermelho (critico)

## 3.3 Elementos minimos de tela

- Topo: recursos (`tempo`, `conhecimento`, `dinheiro`, `reputacao`).
- Centro: incidente atual e opcoes.
- Rodape: mao de cartas.
- Lateral: fila de incidentes e objetivos da missao.

## 3.4 Animacoes leves

- Fade curto ao abrir incidente.
- Pulse no recurso que foi afetado.
- Shake leve em falha critica.

## 4. Loop principal de gameplay

1. Sorteio de incidente baseado em modulo e dificuldade.
2. Jogador escolhe uma carta (ou acao basica).
3. Engine calcula resultado + risco colateral.
4. Recursos sao atualizados.
5. Jogador recebe feedback tecnico.
6. Avanca para o proximo turno.

## 5. Mecanicas para diversao + aprendizado

## 5.1 Mecanicas de diversao

- Escolhas de alto risco: resolver rapido com chance de colateral.
- Combos simples de cartas em 2 turnos.
- Eventos em cadeia (um modulo afeta outro).
- Bonus por streak de resolucao correta.

## 5.2 Mecanicas de aprendizado

- Feedback explicando causa provavel do problema.
- Dica liberada apos erro repetido.
- Glossario tecnico curto por evento.
- Missao de diagnostico sem resposta imediata (estimula raciocinio).

## 5.3 Mecanicas de progressao

- XP por decisao correta.
- Niveis de carreira:
  - Estagiario
  - Analista Jr
  - Analista Pl
  - Consultor
  - Arquiteto ERP
- Desbloqueio gradual de cartas e perks.

## 6. Sistema de recursos

- `tempo`: energia operacional por turno.
- `conhecimento`: capacidade de usar cartas avancadas.
- `dinheiro`: margem para medidas caras.
- `reputacao`: confianca dos usuarios e diretoria.

Regra de tensao:
- recursos baixos aumentam chance de incidente critico.
- reputacao baixa reduz margem para erro.

## 7. Sistema de cartas (versao simples)

## 7.1 Categorias iniciais

- SQL
- ADVPL
- Configuracao
- Suporte
- Infra

## 7.2 Exemplo de cartas

- `Revisar Indice SQL`: melhora performance, custa tempo moderado.
- `Patch ADVPL Rapido`: resolve agora, aumenta risco futuro.
- `Ajustar Parametro MV`: impacto alto em modulo especifico.
- `War Room com Usuarios`: aumenta reputacao, consome tempo.
- `Reiniciar Servico`: acao emergencial com risco de recorrencia.

## 7.3 Combos recomendados

- `Diagnostico SQL` + `Indice Otimizado`: bonus de mitigacao.
- `Analise Log` + `Patch Controlado`: reduz chance de colateral.
- `Comunicacao com Usuario` + `Ajuste Funcional`: bonus de reputacao.

## 8. Sistema de eventos/incidentes

## 8.1 Taxonomia

- Operacional (erros de processo)
- Tecnico (infra, banco, appserver)
- Fiscal/legislativo (regras e validacoes)
- Integracao (APIs, filas, rotinas)

## 8.2 Severidade

- Low
- Medium
- High
- Critical

## 8.3 Eventos em cadeia

Exemplo:
- Nota rejeitada no faturamento -> atraso de recebimento -> queda de reputacao no financeiro.

## 9. Ideias de desafios Protheus (catalogo inicial)

## 9.1 Compras

- Pedido bloqueado por alcada incorreta.
- Fornecedor sem condicao de pagamento valida.
- TES de entrada divergente.
- Pedido duplicado por rotina manual.
- Integracao de cotacao falhou.

## 9.2 Estoque

- Saldo negativo em produto critico.
- Custo medio inconsistente apos ajuste.
- Inventario fisico divergente do sistemico.
- Enderecamento invalido no armazem.
- Transferencia interna nao confirmada.

## 9.3 Faturamento/Fiscal

- NF-e rejeitada por CFOP invalido.
- NF-e rejeitada por CST inconsistente.
- Serie de documento incorreta.
- TES de saida incompatível com operacao.
- Regra fiscal aplicada ao cliente errado.

## 9.4 Financeiro

- Titulo duplicado no contas a pagar.
- Baixa automatica aplicada em aberto errado.
- Juros/multa calculados incorretamente.
- Conciliacao bancaria parcial.
- Fluxo de caixa com buraco por integracao falha.

## 9.5 Banco/Infra

- DBAccess indisponivel.
- AppServer com fila travada.
- Query pesada degradando rotinas.
- Deadlock em fechamento do dia.
- Job noturno nao executou.

## 9.6 Customizacao e sustentacao

- Patch sobrescreveu customizacao.
- Fonte ADVPL sem compatibilidade em release nova.
- Campo custom sem indice degradou desempenho.
- Ponto de entrada executando regra duplicada.
- Erro intermitente por dependencia externa.

## 10. Modo de desafio educativo

## 10.1 Desafio "Diagnostico"

- Jogador recebe sintomas.
- Escolhe hipoteses de causa raiz.
- Depois escolhe acao tecnica.
- Ganha bonus se diagnostico e acao estiverem corretos.

## 10.2 Desafio "SLA"

- Resolver X incidentes em Y turnos.
- Penalidade forte por atraso.
- Recompensa de reputacao alta.

## 10.3 Desafio "Mudanca de release"

- Entram eventos extras durante janela de deploy.
- Jogador decide rollback ou hotfix.
- Ensina tradeoff de risco operacional.

## 10.4 Desafio "Auditoria"

- Objetivo: zerar inconsistencias de processo.
- Eventos com foco em rastreabilidade e compliance.

## 11. Missao exemplo (completo)

Missao: `Fechamento sem caos`

Contexto:
- Ultimo dia do mes.
- 4 incidentes em fila.
- Recursos medianos.

Objetivo:
- Finalizar 6 turnos com reputacao >= 40.
- Evitar colapso em Financeiro e Banco.

Recompensa:
- +XP alto
- desbloqueio da carta `Plano de Contingencia`

## 12. Balanceamento inicial para MVP

- Taxa de sucesso alvo em `normal`: 50%-65% para iniciante.
- Incidentes criticos por partida: 2 a 4.
- Cartas "fortes" com cooldown maior.
- Penalidade em cascata limitada para nao frustrar cedo.

## 13. Estrategia de conteudo por sprints

## Sprint 1

- 8 eventos basicos.
- 5 cartas.
- 1 missao principal.

## Sprint 2

- 12 eventos adicionais.
- 7 cartas novas.
- 3 missoes por modulo.

## Sprint 3

- 10 eventos avancados.
- 8 cartas finais do MVP.
- calibracao de dificuldade.

## 14. Feedback pedagogico (pos-turno)

Sempre mostrar:
- o que o jogador escolheu.
- por que funcionou ou falhou.
- qual conceito tecnico foi aplicado.
- qual seria uma alternativa segura.

Formato curto (3-4 linhas) para manter ritmo do jogo.

## 15. Mecanicas extras opcionais pos-MVP

- Desafio diario com seed fixa.
- Ranking semanal por eficiencia.
- Biblioteca de casos reais anonimizados.
- Modo "apenas diagnostico" para treino rapido.

## 16. Checklist de design antes de implementar

- [ ] O evento eh divertido e didatico?
- [ ] Existe pelo menos 1 resposta segura e 1 arriscada?
- [ ] O feedback ensina algo objetivo?
- [ ] O jogador entende visualmente o impacto?
- [ ] O tempo de decisao continua curto?

## 17. Prompt pronto para IA gerar novos desafios

```txt
Crie [N] eventos novos para o jogo Protheus Quest no formato JSON.
Cada evento deve conter: code, title, moduleType, severity, symptoms,
rootCauseHint, suggestedActions, rightAction, wrongActionImpact, learningPoint.
Regras: manter linguagem objetiva, dificuldade progressiva e foco em troubleshooting ERP.
```

## 18. Resultado esperado do design

- Jogador se diverte pela pressao de escolha e progressao.
- Jogador aprende conceitos praticos de sustentacao ERP.
- Equipe consegue evoluir conteudo rapido sem aumentar complexidade tecnica.

Fim.

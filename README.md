![Banner do Forge](assets/forge-banner.svg)

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.14-3776AB?logo=python&logoColor=white" alt="Python 3.14">
  <img src="https://img.shields.io/badge/projetos-13-6f42c1" alt="13 projetos">
  <img src="https://img.shields.io/badge/status-conclu%C3%ADdo-2ea44f" alt="Status concluído">
  <img src="https://img.shields.io/badge/depend%C3%AAncias-uv-DE5FE9?logo=uv" alt="Dependências gerenciadas com uv">
</p>

Cada projeto isola uma decisão real de arquitetura de serviços e prova a resposta
com teste e evidência, não só com código que compila. Cada pasta tem ambiente,
dependências, testes e evidência próprios; abrir uma não exige carregar as outras.

## Projetos

### Base

- **[P00 · Ambiente reproduzível](./P00-ambiente-reproduzivel-com-uv/README.md)**.
  Garante que o ambiente nasce igual depois de apagado, com `uv`, Ruff, Pyright
  e pytest. A instalação limpa passa; falha de import e de tipo é pega antes de
  rodar.

### Dados, concorrência, API e processo

- **[P01 · Validação tipada de JSONL](./P01-validacao-tipada-com-pydantic-e-jsonl/README.md)**.
  Pydantic separa registro válido de inválido sem esconder o que falhou: seis
  linhas de entrada viram duas válidas e quatro rejeitadas.
- **[P02 · Asyncio, threads e processos](./P02-asyncio-threads-e-processos/README.md)**.
  Compara as três estratégias de concorrência no mesmo problema: `asyncio.gather`
  fecha I/O em 0,25s contra 2,51s sequencial; processos fecham CPU em 0,79s
  contra 2,26s direto.
- **[P03 · Contratos e injeção no FastAPI](./P03-fastapi-contratos-e-injecao-de-dependencias/README.md)**.
  Separa contrato HTTP, regra de negócio e persistência com `Depends`; os status
  `401`, `404`, `409`, `422` e `503` aparecem na fronteira certa.
- **[P04 · Lifespan e tarefas de fundo](./P04-lifespan-middleware-e-background-tasks/README.md)**.
  Testa se um job sobrevive a um crash da API: o crash mata o job preso ao
  processo, o shutdown gracioso fecha os recursos antes de sair.

### Resiliência e integrações externas

- **[P05 · Timeout, retry, backoff e jitter](./P05-timeout-retry-backoff-e-jitter/README.md)**.
  Decide quando repetir uma chamada é seguro: falha transitória repete, `400` e
  operação insegura não.
- **[P06 · Circuit breaker e token bucket](./P06-circuit-breaker-e-token-bucket/README.md)**.
  Decide quando parar de chamar um provedor: três falhas abrem o circuito, uma
  rajada maior que o bucket é recusada localmente.
- **[P07 · Paginação por cursor](./P07-paginacao-por-cursor/README.md)**.
  Pagina uma coleção que muda durante a leitura: `OFFSET` repete um item, o
  cursor preserva a janela original.
- **[P08 · Webhook, HMAC, inbox e idempotência](./P08-webhook-hmac-inbox-e-idempotencia/README.md)**.
  Aceita redelivery sem repetir o efeito: três entregas produzem um registro e
  um efeito só, mesmo com falha simulada entre persistência e efeito.

### PostgreSQL: constraints, planos e isolamento

- **[P09 · Constraints e migrations](./P09-postgresql-constraints-e-migrations/README.md)**.
  Faz o banco rejeitar estado impossível: quatro escritas inválidas falham, a
  migration sobe e desce sem correção manual.
- **[P10 · EXPLAIN ANALYZE e índices](./P10-explain-analyze-e-indices/README.md)**.
  Escolhe índice a partir do plano, não do palpite: acesso alinhado usa
  `Index Only Scan`; N+1 faz 21 consultas contra 1 do `JOIN`.
- **[P11 · RLS e connection pool](./P11-postgresql-rls-e-connection-pool/README.md)**.
  Impede que o contexto de tenant vaze pela conexão reaproveitada: sem RLS o
  escopo vaza; a transação segura limpa o contexto após commit e rollback.

### Mensageria

- **[P12 · Kafka versus RabbitMQ](./P12-kafka-versus-rabbitmq/README.md)**.
  Decide se o problema pede replay ou confirmação: um novo grupo de
  consumidores relê os seis eventos no Kafka; o RabbitMQ redelivera até
  receber `ack`.

A ordem é sugestão, não dependência: quem já conhece o básico pode começar direto
pelo problema que quer investigar.

## Limites

O Forge comprova mecanismos em cenários pequenos e controlados. Não mede volume
de produção, disponibilidade distribuída, custo de nuvem, operação de clusters ou
comportamento multi-região.

# P02: Corrida I/O versus CPU

Este projeto compara formas de executar operações que esperam I/O e cálculos que usam
CPU. Os dois grupos são medidos separadamente porque resolvem problemas diferentes.

## Como o programa funciona

O primeiro experimento simula dez chamadas externas com 250 milissegundos de espera
cada. Ele executa essas chamadas em sequência, com `asyncio.gather` e com uma função
bloqueante dentro do código assíncrono.

O segundo experimento executa quatro cálculos iguais. Ele compara execução direta,
threads e processos. Uma tarefa de heartbeat tenta rodar a cada 10 milissegundos para
mostrar quando o event loop deixa de responder.

| Tipo de carga | Estratégias comparadas |
| --- | --- |
| espera de I/O | sequencial, `asyncio.gather`, chamada bloqueante |
| cálculo em Python | direto, `asyncio.to_thread`, processos |

Todas as estratégias de um mesmo grupo recebem as mesmas entradas e precisam produzir
os mesmos resultados.

O Pydantic lê os parâmetros do YAML e o tabulate monta a tabela final. Essas
abstrações ficam nas bordas do experimento. O arquivo de estratégias mantém visíveis
as chamadas que demonstram espera concorrente, bloqueio, threads e processos.

## Conceito abordado

[Concorrência](https://pt.wikipedia.org/wiki/Concorr%C3%AAncia_(ci%C3%AAncia_da_computa%C3%A7%C3%A3o))
é avançar mais de uma operação no mesmo intervalo. Ajuda quando o programa passa tempo
esperando rede, disco ou banco. O
[`asyncio`](https://docs.python.org/pt-br/3/library/asyncio.html) coordena essas
esperas por meio de um event loop.

[Paralelismo](https://pt.wikipedia.org/wiki/Computa%C3%A7%C3%A3o_paralela) é executar
cálculos ao mesmo tempo. No CPython tradicional, o
GIL (Global Interpreter Lock) limita a execução
simultânea de código Python em threads. Processos usam interpretadores separados e
podem aproveitar vários núcleos, mas custam mais para criar e para trocar dados entre
si.

## Para que isso serve em produção

A escolha errada deixa uma API lenta mesmo quando há recursos disponíveis.

Exemplo: um endpoint consulta dez serviços independentes. Se esperar cada resposta
antes de iniciar a próxima, as latências se somam. Com chamadas HTTP assíncronas, ele
inicia todas e aguarda o conjunto. Se esse endpoint também gerar um relatório com
cálculo pesado, esse cálculo não deve rodar direto no event loop. Ele deve ir para um
processo ou para um worker separado.

## Como executar

```bash
uv sync --locked
uv run ruff check .
uv run pyright
uv run pytest
uv run run-race --scenario scenario.yaml --output output/results.json
```

O último comando executa cada estratégia cinco vezes, calcula a mediana e grava os
dados em `output/results.json`.

## Resultado observado

No ambiente descrito em `evidence/results.md`, a execução produziu:

| Situação | Estratégia | Tempo mediano | Maior atraso do heartbeat |
| --- | --- | ---: | ---: |
| I/O | sequencial | 2,51 s | 0,01 s |
| I/O | `asyncio.gather` | 0,25 s | menos de 0,01 s |
| I/O | chamada bloqueante | 2,50 s | 2,49 s |
| CPU | direta | 2,26 s | 2,41 s |
| CPU | `asyncio.to_thread` | 2,34 s | 0,10 s |
| CPU | processos | 0,79 s | 0,13 s |

O `gather` reduziu o tempo das esperas concorrentes. As threads melhoraram a resposta
do event loop, mas não aceleraram o cálculo. Os processos foram mais rápidos neste
cenário. Os números variam conforme o computador.

## Limite do projeto

O pool de processos é recriado em cada repetição, então parte do tempo medido vem da
inicialização. O teste é local, curto e sintético. Ele não define a melhor estratégia
para outra carga ou ambiente.

## Resumo da ópera

Use operações assíncronas quando o programa espera I/O. Não execute código bloqueante
ou cálculo pesado no event loop. Threads preservam a capacidade de resposta. Processos
paralelizam cálculo em Python quando o ganho compensa o custo.

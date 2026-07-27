# P00: Ambiente reproduzível

Este projeto mostra como recriar um ambiente Python sem depender dos pacotes que já
estão instalados na máquina.

## Como o programa funciona

O projeto registra a versão do Python, as dependências e as ferramentas de qualidade.
O `uv` usa esses arquivos para criar a `.venv`, instalar as versões corretas e executar
lint, verificação de tipos e testes.

```text
.python-version + pyproject.toml + uv.lock
                    |
                    v
          ambiente virtual novo
                    |
                    v
          lint + tipos + testes
```

A `.venv` pode ser apagada a qualquer momento, porque ela é só o resultado desses
arquivos.

## Conceito abordado

O conceito é reprodutibilidade de ambiente: outra pessoa consegue instalar as mesmas
versões e rodar as mesmas verificações usando só os arquivos versionados no Git.

O Typer é uma dependência direta, porque a aplicação precisa dele para rodar a CLI.
Ruff, Pyright e pytest são dependências de desenvolvimento: verificam o projeto, mas
não participam da execução da aplicação.

| Arquivo | Função |
| --- | --- |
| `.python-version` | seleciona o Python usado nesta pasta |
| `pyproject.toml` | declara o projeto, as dependências diretas e as ferramentas |
| `uv.lock` | fixa as versões exatas dos pacotes instalados |
| `.venv` | guarda o ambiente gerado e não entra no Git |

## Para que isso serve em produção

Uma equipe precisa rodar o mesmo código em notebooks, integração contínua e
containers. Versões diferentes de dependências podem mudar o comportamento do
programa ou quebrar o deploy.

Exemplo: uma biblioteca publica uma versão nova com uma mudança incompatível. Um
projeto sem lock instala essa versão automaticamente e a pipeline passa a falhar. Com
o `uv.lock`, todos continuam na versão validada até alguém atualizar de forma
explícita.

## Como executar

```bash
uv sync --locked
uv run ruff check .
uv run pyright
uv run pytest
uv run p00 Forge
```

`uv sync --locked` cria a `.venv`. O `uv run` executa cada ferramenta dentro desse
ambiente, então não é preciso ativá-lo manualmente.

Para apagar o ambiente e provar que ele pode ser reconstruído:

```bash
rm -rf .venv .pytest_cache .ruff_cache
uv sync --locked
uv run ruff check .
uv run pyright
uv run pytest
```

## Falha controlada

```bash
uv run pyright --project experiments/pyright-missing.json
uv run pyright --project experiments/pyright-type.json
```

O experimento contém uma dependência ausente e uma atribuição de texto onde o código
espera um número. Cada comando deve falhar e apontar, respectivamente,
`reportMissingImports` e `reportAssignmentType`. A falha do verificador é o próprio
objetivo deste experimento.

## Resultado observado

No ambiente registrado em `evidence/environment.txt`: Python 3.14.6, uv 0.11.29,
pyenv 2.8.1. A recriação da `.venv` passou pelo Ruff, pelo Pyright e por dois testes.
O Pyright também detectou as duas falhas controladas, como esperado.

## Limite do projeto

O teste cobre só o ambiente local. Ele não comprova compatibilidade com outros
sistemas operacionais, outras versões do Python, nem publicação de pacotes.

## Resumo

Os arquivos versionados definem o ambiente. A `.venv` é descartável. Em um projeto
real, essa separação reduz diferenças entre desenvolvimento, testes e deploy.

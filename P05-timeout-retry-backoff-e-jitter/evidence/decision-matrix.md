# Matriz de decisão

| Resultado do provedor | Leitura segura | Criação não idempotente |
| --- | --- | --- |
| `2xx` | devolver sucesso | devolver sucesso |
| `400` | parar, erro definitivo | parar, erro definitivo |
| `429` | repetir dentro do limite | parar, operação insegura |
| `5xx` | repetir dentro do limite | parar, operação insegura |
| timeout | repetir dentro do limite | parar, operação insegura |
| falha transitória na última tentativa | parar, limite esgotado | não chega a uma nova tentativa |

A leitura usa `GET`. Pode ser repetida sem criar um novo efeito.

A criação usa `POST` e, neste laboratório, não recebe uma chave de idempotência.
Por isso, um timeout na criação é tratado como erro: pode ser que o provedor já
tenha concluído a operação, e repetir a chamada criaria um segundo efeito. Nesse
caso o cliente para e não tenta de novo.

# Corretor de cartões ENA e SNAD/PROFMAT: arquivos e API do servidor

## 1. Arquivos publicados

Todos na mesma pasta, servidos por HTTPS:

| Arquivo | Quem mantém | Obrigatório |
|---|---|---|
| `index.html` | desenvolvimento | sim |
| `gabarito.json` | administrador | sim |
| `config.json` | administrador | não (há padrões) |
| `tesseract/` | desenvolvimento | não (sem a pasta, o OCR é baixado do jsDelivr) |

O ENA e o SNAD usam o mesmo `index.html`, publicado em pastas separadas (por exemplo, `corretor/` e `corretor-snad/`), cada uma com o seu `config.json` e o seu `gabarito.json`. O app lê os dois JSON sempre que é aberto, sem cache. O app não funciona aberto direto do disco (`file://`): precisa estar num servidor web.

### gabarito.json

```json
{ "1": "B", "2": "C", "3": "B", "...": "...", "30": "E" }
```

As chaves vão de `"1"` a `"30"`. Os valores aceitos são `A` a `E`, ou `X` (também `ANULADA`) para questão anulada, que dá o ponto a todos. Se o arquivo faltar, não for JSON válido ou tiver alguma questão ausente ou inválida, o app bloqueia a correção e mostra o motivo. Uma alteração no gabarito vale para todos os cartões na próxima vez que o app for aberto, porque as notas são recalculadas.

### config.json

```json
{
  "exame": "ENA",
  "prova": "ENA 2027",
  "notaMaxima": 30,
  "rigorLeitura": "normal",
  "dpiMinimo": 150,
  "dpiRecomendado": 200,
  "armazenamento": {
    "modo": "api",
    "api": "https://www.profmat-sbm.org.br/corretor/api",
    "token": "",
    "credenciais": "include"
  }
}
```

| Campo | Significado | Padrão |
|---|---|---|
| `exame` | `ENA` ou `SNAD`. Define o modelo do cartão, as regras de classificação do edital e a base de dados do navegador | `ENA` |
| `prova` | nome exibido no app e nos arquivos exportados | `ENA 2027` (ou `SNAD 2027`) |
| `notaMaxima` | nota = acertos ÷ 30 × notaMaxima | `30` |
| `rigorLeitura` | `rigido`, `normal` ou `sensivel` | `normal` |
| `dpiMinimo` | PDFs com alguma página abaixo disso são recusados (com 5% de tolerância) | `150` |
| `dpiRecomendado` | valor pedido nas mensagens ao usuário | `200` |
| `armazenamento.modo` | `local`: tudo só no navegador. `api`: envia tudo ao servidor | `local` |
| `armazenamento.api` | endereço base da API (sem barra no fim) | — |
| `armazenamento.token` | se preenchido, enviado como `Authorization: Bearer <token>` | vazio |
| `armazenamento.credenciais` | `include` envia os cookies de sessão do site (para usar o login existente) | `same-origin` |

## 2. API

### Regras gerais

- **Identificadores:** os ids de lote, cartão e planilha são gerados pelo app (`[A-Za-z0-9_-]`, até 80 caracteres). Por isso todas as gravações são `PUT /recurso/{id}`. Repetir o mesmo `PUT` deve sobrescrever, nunca duplicar: o app reenvia o que falhou.
- **Envio de arquivos:** é feito em `multipart/form-data`.
- **Sucesso:** qualquer resposta `2xx`. O corpo pode ser `{"id": "..."}` ou vazio.
- **Falha:** em qualquer outra resposta, o app guarda o item e tenta de novo depois (a cada 30 a 60 segundos e ao reabrir). Com `401` ou `403`, mostra "Servidor recusou o acesso".
- **Escopo:** cada usuário deve ver e alterar só os próprios cartões (por exemplo, os do seu polo). O administrador vê todos. O app não filtra nada: o servidor decide pelo login.

### Endpoints

#### 1. Enviar lote — `PUT /lotes/{id}`

Um lote é cada PDF aceito pelo app.

| Campo (multipart) | Conteúdo |
|---|---|
| `arquivo` | o PDF original, como foi enviado pelo usuário |
| `nome_arquivo` | nome do arquivo |
| `paginas` | número de páginas |

Resposta: `200` ou `201`.

#### 2. Salvar cartão — `PUT /cartoes/{id}`

O app chama este endpoint quando o cartão é lido e de novo a cada correção na revisão.

| Campo (multipart) | Conteúdo |
|---|---|
| `dados` | JSON do cartão (abaixo) |
| `imagem` | JPEG do cartão retificado (cerca de 150 KB). Só vai no primeiro envio: nos seguintes, mantenha a imagem já gravada |

Resposta: `200`.

Exemplo do campo `dados`:

```json
{
  "id": "muxcb0t4gv1l",
  "exame": "ENA",
  "lote": "muxcb0q9k2aa",
  "arquivo": "polo_unb_sala4.pdf",
  "pagina": 3,
  "nome": "BEATRIZ ALMEIDA FONTOURA",
  "cpf": "171.056.262-59",
  "ausente": false,
  "eliminado": false,
  "acertos": 21,
  "nota": 21,
  "respostas": ["C", "A", "", "DUPLA", "..."],
  "revisado": true,
  "alertas": ["Dupla marcação — questão zerada: Q4"],
  "lista": "Confere com a lista",
  "lido_no_cartao": { "nome": "BEATRIZ ALMEIDA FONTOURA", "cpf": "171.056.262-59" },
  "atualizado": "2027-10-18T14:03:22.511Z",
  "estado": { "...": "uso interno do app" }
}
```

- **`exame`:** `ENA` ou `SNAD`, conforme o `config.json` da pasta. Se as duas pastas usarem a mesma API, o servidor deve separar os cartões por este campo (ou usar um endereço de API diferente para cada exame).
- **`respostas`:** 30 itens, cada um com a letra lida, `""` (em branco) ou `"DUPLA"`. Uma questão com dupla marcação vale zero.
- **`estado`:** é usado pelo app para reabrir o cartão em outro computador. Guarde-o como recebido, sem interpretar.
- **Banco de dados:** os demais campos servem para o servidor montar relatórios sem depender do app.

#### 3. Listar cartões — `GET /cartoes`

O app chama este endpoint ao abrir, para trazer os cartões que não estão naquele navegador.

Resposta: `200` com um array JSON.

```json
[
  { "id": "muxcb0t4gv1l", "dados": { "...": "o mesmo JSON recebido no PUT" }, "imagem_url": "cartoes/muxcb0t4gv1l/imagem" }
]
```

`imagem_url` pode ser absoluta ou relativa ao endereço base da API.

#### 4. Imagem do cartão — `GET /cartoes/{id}/imagem`

Resposta: `200` com `image/jpeg`, ou o endereço que vier em `imagem_url`. Exige o mesmo login dos demais endpoints.

#### 5. Apagar cartão — `DELETE /cartoes/{id}`

Resposta: `204`. Se o cartão já não existir, responda `204` ou `404`: o app considera apagado nos dois casos.

#### 6. Salvar planilha — `PUT /planilhas/{id}`

O app chama este endpoint a cada exportação. O usuário continua recebendo o arquivo para baixar.

| Campo (multipart) | Conteúdo |
|---|---|
| `arquivo` | o `.xlsx` de resultados ou o `.pdf` com os cartões anotados |
| `tipo` | `planilha` ou `cartoes_anotados` |
| `nome_arquivo` | nome do arquivo |

Resposta: `200`.

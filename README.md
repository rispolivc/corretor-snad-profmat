# Corretor de cartões-resposta — ENA e SNAD / PROFMAT 2027

App web, em um único `index.html`, que corrige os cartões-resposta a partir de PDFs digitalizados e gera as planilhas de notas e de classificação. O mesmo `index.html` atende os dois exames:

| Exame | Edital | Pasta |
|---|---|---|
| **ENA 2027** (mestrado) | nº 23/2026 | `corretor/` |
| **SNAD 2027** (doutorado, Profmat-D), Etapa 1 | nº 24/2026 | `corretor-snad/` |

O exame é escolhido pelo campo `"exame"` do `config.json` (`"ENA"` ou `"SNAD"`). Cada pasta tem o seu `config.json` e o seu `gabarito.json`. Os cartões de cada exame ficam guardados separadamente no navegador, então ENA e SNAD não se misturam, mesmo no mesmo site.

## Arquivos de cada pasta

| Arquivo | Para que serve |
|---|---|
| `index.html` | o app (igual nas duas pastas) |
| `gabarito.json` | gabarito oficial (`"1":"B", …`; `X` ou `ANULADA` para questão anulada). Enquanto estiver incompleto, a correção fica bloqueada |
| `config.json` | exame, nome da prova, nota máxima, rigor da leitura, dpi mínimo e armazenamento (local ou servidor) |
| `tesseract/` | OCR de nome e CPF (opcional; sem a pasta, o app usa a CDN) |

## Como usar

1. **Carregar PDF de cartões.** Use digitalização em mesa (scanner), com pelo menos 200 dpi. PDFs abaixo de 150 dpi são recusados por inteiro.
2. **Carregar lista de inscritos (.xlsx), opcional.** É a planilha do SCA. O app confere nome e CPF de cada cartão com a lista e lê nascimento, rede pública/privada, quadro permanente, raça/cor, gênero e, no SNAD, o tempo de docência na rede pública.
3. **Revisar.** Os cartões com alerta aparecem em "Revisar". Clique na alternativa para corrigir e marque como revisado.
4. **Exportar.** O app pede o número de vagas da instituição, já preenchido pelo Anexo 2 do edital, que pode ser alterado se a CAN mudar as vagas.

## O que o app lê

- **Cartão:** o modelo do ENA 2027 ou o do SNAD 2027. O do SNAD tem as linhas "Curso" e "Local de Prova" no cabeçalho e a mesma grade de respostas.
- **Nome e CPF:** por OCR, inclusive em faixas de fundo escuro.
- **Respostas A–E:** alinhamento automático por rotação e escala, com tolerância a rasuras e às letras impressas dentro dos balões.
- **Dupla marcação:** inclui rasura, X e traço em outra alternativa. A questão vale 0.
- **Campo cinza da comissão pintado, inteiro ou em parte:** a questão é anulada para o candidato e vale 0.
- **Questão anulada no gabarito:** o ponto vai para todos.
- **Ausente e Eliminado.**

Toda leitura duvidosa gera alerta. Nada é corrigido em silêncio.

## O que o app gera

**Sem lista de inscritos:** planilha `.xlsx` com nome, CPF, acertos, nota, situação, alertas e a resposta de cada questão.

**Com a lista do SCA:** a própria planilha do SCA, com a formatação original, acrescida de:

- **Aba original** com as colunas **Nota**, **Lista** (ENA) ou **Resultado da Etapa 1** (SNAD), e **Posição**, e as linhas já na ordem de classificação.
- **Aba Resultados**, de uso interno, com o resultado, posição, nome, CPF mascarado, CPF, nascimento, rede, quadro permanente, tempo de docência (SNAD), raça/cor, gênero, nota, situação, critério de desempate e respostas.

### ENA 2027 (edital nº 23/2026)

1. **Lista 1:** rede pública, aprovados, dentro das vagas.
2. **Lista 2:** rede pública, aprovados além das vagas.
3. **Lista 3:** somente rede privada, aprovados.
4. Aprovados sem rede informada (conferir), reprovados (menos de 50%), eliminados e ausentes.

Quem atua nas redes pública e privada entra como rede pública. Desempate (item 7.1.1): 60 anos ou mais na data da prova (o mais velho primeiro), quadro permanente da rede pública, data de nascimento mais antiga.

### SNAD 2027, Etapa 1 (edital nº 24/2026)

1. **Classificado para a Etapa 2:** habilitados (50% ou mais de acertos) até 5 vezes o número de vagas da instituição, **incluídos os empates** na nota de corte (item 4.1.2).
2. **Habilitado (fora do limite da Etapa 2).**
3. **Eliminado na Etapa 1 (menos de 50%)**, eliminados e ausentes (itens 4.1.1 e 7.4).

A posição é única para todos os habilitados. Desempate (item 4.2.1): 60 anos ou mais na data da prova (o mais velho primeiro), quadro permanente da rede pública, maior tempo de docência em Matemática na rede pública, data de nascimento mais antiga.

**Outros arquivos (os dois exames):** PDF com todos os cartões anotados e PDF com gráficos de frequência por questão.

## Atenção

- A aba Resultados tem nome e CPF completo. **Não publique essa aba.**
- A lista de notas para divulgação (ENA item 6.5, SNAD item 7.5) leva só CPF mascarado e nota.
- As cotas de cada instituição não são aplicadas.
- No SNAD, as listas finais (Listas 1, 2 e 3 do item 11.5) dependem das notas das Etapas 2 e 3 e não são geradas pelo app.
- No modo local, os dados ficam só no navegador. Exporte antes de limpar o navegador.
- Não commite planilhas nem PDFs de candidatos no repositório.

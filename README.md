# Auto Conexo

Automação em **UiPath** que resolve o [Conexo](https://conexo.ws) — o jogo diário
de agrupar 16 palavras em 4 grupos de 4 — por **força bruta**. O robô abre o
puzzle do dia no Chrome e vai testando combinações de quatro peças até o
tabuleiro se resolver sozinho.

## Como funciona

Todo o processo está em `Main.xaml`:

1. **Monta a URL do dia.** Um `Assign` faz
   `data = Today.Date.ToString("yyyy-MM-dd")` e o Application Card abre
   `"https://conexo.ws/pt/previous/" + data` no Chrome. Usar a rota `/previous/`
   com a data de hoje garante que sempre cai no puzzle correto, sem depender de
   redirect da home.

2. **Varre o tabuleiro.** Dentro de um `While (True)` há um `Retry Scope` com
   `NumberOfRetries = 9999999` e **sem condição de saída** — ou seja, ele só
   reexecuta quando o corpo lança exceção. Dentro dele ficam **quatro
   `For Each UI Element` aninhados**, todos sobre as `div` da `.board-wrapper`
   (o container das 16 peças).

3. **Testa a combinação.** Cada nível guarda sua peça e o rótulo dela:

   ```
   nível 1 → elemento_1 / elemento_1_label
   nível 2 → elemento_2 / elemento_2_label
   nível 3 → elemento_3 / elemento_3_label
   nível 4 → CurrentElement (a peça sendo iterada)
   ```

   No miolo, um `For Each` sobre `{elemento_1, elemento_2, elemento_3}` clica nas
   três primeiras peças e, logo depois, um `Click` em `CurrentElement` com
   `DelayAfter = 2` fecha a quarta — completando o palpite e dando tempo do jogo
   reagir.

4. **Se acerta, o tabuleiro muda.** Quando um grupo é resolvido, as peças são
   remanejadas e as referências de UI que os loops guardavam ficam inválidas.
   Isso lança exceção, o `Retry Scope` reinicia a varredura do zero e o robô
   volta a testar combinações — agora sobre um tabuleiro menor. Repetindo até
   acabar. **A quebra do seletor é o mecanismo de progresso, não um bug.**

## Pré-requisitos

- **UiPath Studio 26.0** ou superior (projeto criado na `26.0.186.0`)
- Dependências (restauradas automaticamente ao abrir o projeto):
  - `UiPath.System.Activities` 25.6.1
  - `UiPath.UIAutomation.Activities` 25.10.13
- **Google Chrome** com a extensão do UiPath instalada
- Target framework: **Windows**

## Como executar

1. Abra a pasta no UiPath Studio e deixe restaurar as dependências.
2. Rode `Main.xaml`. O Chrome abre no Conexo do dia.
3. Não mexa no mouse nem troque de janela — os cliques são feitos na tela real.

Para parar, use o Stop do Studio (o `While` é um `Interruptible While` e encerra
de forma limpa no fim da iteração).

## Limitações conhecidas

- **É força bruta de verdade.** Os quatro loops não têm filtro para evitar
  repetição, então combinações com a mesma peça mais de uma vez também são
  testadas — e clicar duas vezes na mesma peça só desmarca ela. Muitas
  iterações são desperdiçadas.
- **Não conta erros.** O Conexo tem limite de tentativas erradas; a automação
  ignora isso completamente e vai até o fim das combinações de qualquer jeito.
- Não há condição de parada por sucesso: o `While (True)` roda até você
  interromper manualmente, mesmo com o puzzle já resolvido.
- Os seletores dependem da estrutura atual do site (`.board-wrapper` sob
  `#root`). Se o layout mudar, a varredura para de encontrar as peças.
- Só funciona em primeiro plano, no Chrome.

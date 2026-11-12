## Autores

- RUAN MARCELO RAMACIOTI LUZ

## Objetivo

Reorganizar a loja de meias de L8-end em componentes e,
na segunda etapa, atualizar o carrinho por eventos.

O projeto utiliza Vue.js 3 com Options API por CDN.

## Como executar

1. Abra a pasta que contém index.html e main.js no VS Code.
2. Abra index.html com a extensão Live Server.
3. Mantenha acesso à internet para carregar o Vue pela CDN.
4. Abra o console do navegador para acompanhar mensagens e erros.

Não é necessário executar npm install.

## Organização dos arquivos

- index.html: contém a raiz, o contador do carrinho, a utilização
  de product-display e o carregamento dos scripts.
- main.js: cria a aplicação e mantém cart e premium.
- components/ProductDisplay.js: apresenta o produto e mantém
  seus dados, seleção de variante, métodos e cálculos.
- components/ProductDetails.js: recebe details por prop
  e apresenta a lista de composição.
- assets/styles.css: mantém os estilos da loja.
- assets/images/: contém as imagens das variantes.

## Etapa 1 — Componentização

### 1. O que retornam Vue.createApp e app.mount?

Em main.js, Vue.createApp retorna o objeto da aplicação,
armazenado em app. Ele permite registrar os componentes
com app.component.

Em index.html, app.mount('#app') monta a aplicação no elemento
de id app e retorna a instância pública do componente raiz,
armazenada em mountedApp.

Podemos acessar o estado da raiz pelo console, por exemplo,
com mountedApp.premium = false.

### 2. Quais dados foram para product-display?

Foram transferidos product, brand, selectedVariant, details,
variants e onSale.

cart permaneceu na raiz porque representa o carrinho da
aplicação. O produto conhece a variante selecionada, enquanto
a raiz é responsável por manter o carrinho.

premium também pertence à raiz e é enviado ao produto por prop.

### 3. Como selectedVariant determina imagem e disponibilidade?

O método updateVariant(index) altera this.selectedVariant.

A propriedade computada image retorna:
this.variants[this.selectedVariant].image

A propriedade computada inStock retorna:
this.variants[this.selectedVariant].quantity

Quando o índice muda, o Vue atualiza os valores derivados.
Não é necessário atribuir valores diretamente a image ou inStock.

Nesta versão, inStock retorna uma quantidade: 50 para a verde
e 0 para a azul. Zero é tratado como falso nas condições.

### 4. Como premium chega ao produto e altera shipping?

Em index.html, :premium="premium" envia o dado da raiz à prop
premium declarada em ProductDisplay.js.

A prop é declarada com type: Boolean e required: true.

A propriedade computada shipping retorna 'Free' quando premium
é verdadeiro e 2.99 quando é falso.

Alterar mountedApp.premium atualiza a prop recebida pelo produto
e, consequentemente, o frete apresentado.

### 5. Como verificar a independência das instâncias?

O teste utiliza temporariamente duas tags product-display:
uma com :premium="true" e outra com :premium="false".

Ao selecionar azul no primeiro produto, o segundo deve continuar
verde. Cada instância possui seu próprio selectedVariant.

Os fretes devem ser Free no primeiro e 2.99 no segundo.

Depois do teste, a página deve voltar a apresentar uma única
instância com :premium="premium".

Os resultados realizados estão registrados na tabela de testes.

### 6. Por que this.cart += 1 no filho não altera o pai?

Dentro de ProductDisplay, this referencia a instância do produto.
Ele não referencia o componente raiz, onde cart foi declarado.

Na etapa 1, addToCart obtém o ID da variante selecionada e
executa console.log. Isso permite verificar a seleção sem
tentar alterar o estado do pai.

O carrinho permanece em zero nessa etapa intermediária.
A atualização por eventos será implementada na etapa 2.

## Diferenças em relação à referência L9-end

- A promoção foi preservada com onSale, sale e seu parágrafo.
- O método provisório addToCart registra o ID no console.
- A lista de detalhes utiliza :key="detail".
- A imagem possui :alt="title" como alternativa textual.

## Testes da etapa 1

| Teste | Resultado esperado | Resultado obtido |
|---|---|---|
| Abrir a página | Verde disponível e carrinho em zero | Pendente |
| Selecionar azul | Imagem azul, indisponível e botão desativado | Pendente |
| Adicionar verde | ID 2234 no console e carrinho em zero | Pendente |
| Alterar premium para false | Frete muda para 2.99 | Pendente |
| Conferir promoção | Mensagem de promoção visível | Pendente |
| Conferir detalhes | Três itens de composição visíveis | Pendente |
| Testar duas instâncias | Seleções independentes e fretes Free e 2.99 | Pendente |
| Conferir console | Sem erros ou avisos do Vue | Pendente |

## Etapa 2 — Comunicação por eventos

O carrinho da raiz passou de um número para um array de IDs.

Em ProductDisplay.js, a opção emits declara o evento add-to-cart.
O método addToCart verifica a disponibilidade e emite o evento
com o ID da variante selecionada.

Em index.html, @add-to-cart="updateCart" conecta o evento ao
método da raiz. O contador utiliza cart.length.

Em main.js, updateCart(id) executa this.cart.push(id).
Dois cliques na variante verde produzem [2234, 2234].

O filho comunica a intenção de adicionar. A raiz recebe o ID
e altera seu próprio carrinho, sem acesso direto por $parent.

Adicionar ao carrinho não reduz o estoque neste exercício.
O carrinho permanece apenas na memória e reinicia ao recarregar.

## Testes da etapa 2

| Teste | Resultado esperado | Resultado obtido |
|---|---|---|
| Recarregar | Carrinho vazio e Cart(0) | Pendente |
| Adicionar verde duas vezes | Cart(2) e array [2234, 2234] | Pendente |
| Selecionar azul | Imagem azul, botão desativado e carrinho inalterado | Pendente |
| Voltar à verde e adicionar | Cart(3) e array [2234, 2234, 2234] | Pendente |
| Alterar premium para false | Frete 2.99, preservando carrinho e seleção | Pendente |
| Conferir detalhes e promoção | Ambos preservados | Pendente |
| Conferir console | Sem erros ou avisos do Vue | Pendente |
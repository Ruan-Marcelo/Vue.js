# Loja de meias — Aula 04

**Autor:** Ruan Marcelo Ramacioti Luz

## Objetivo

A atividade foi feita a partir da loja de meias de L8-end, usando Vue.js 3 com Options API. Primeiro, o produto foi separado em componentes. Depois, o carrinho passou a receber os itens por eventos. A pasta original foi mantida para consulta.

## Como executar

Abra a pasta `loja-meias-componentes` no VS Code e execute o `index.html` com o Live Server. É preciso ter internet para carregar o Vue pela CDN. Não precisa instalar pacotes com npm. Para acompanhar erros e testar os dados, use o console do navegador.

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

Em `main.js`, `Vue.createApp` retorna a aplicação, guardada em `app`. Esse objeto é usado para registrar os componentes com `app.component`.

Já `app.mount('#app')`, no `index.html`, monta a aplicação na div de id `app` e retorna a instância pública da raiz. Ela fica em `mountedApp`, que permite acessar os dados pelo console, por exemplo com `mountedApp.premium = false`.

### 2. Quais dados foram para product-display?

`product`, `brand`, `selectedVariant`, `details`, `variants` e `onSale` foram para o `data()` de `ProductDisplay.js`, pois são dados do produto.

O `cart` ficou na raiz porque o pai controla o carrinho da página. O produto só precisa informar qual variante foi escolhida. O dado `premium` também fica no pai e é passado ao produto por uma prop.

### 3. Como selectedVariant determina imagem e disponibilidade?

O método `updateVariant(index)` guarda o índice escolhido em `selectedVariant`. As propriedades computadas consultam a variante nessa posição:

```javascript
// image
return this.variants[this.selectedVariant].image;

// inStock
return this.variants[this.selectedVariant].quantity;
```

Assim, basta mudar o índice para atualizar a imagem e a disponibilidade. `inStock` retorna a quantidade: 50 para a verde e 0 para a azul. Como zero é falso na condição, a azul aparece sem estoque e seu botão fica desativado.

### 4. Como premium chega ao produto e altera shipping?

No `index.html`, `:premium="premium"` passa o valor do pai para a prop declarada em `ProductDisplay.js`, com `type: Boolean` e `required: true`.

O cálculo de `shipping` retorna `Free` quando esse valor é verdadeiro e `2.99` quando é falso. Ao mudar `mountedApp.premium` no console, o frete acompanha a alteração.

### 5. Como verificar a independência das instâncias?

O teste consiste em colocar duas tags `product-display` na página, uma com `:premium="true"` e outra com `:premium="false"`. Ao selecionar azul na primeira, a segunda deve continuar verde. O frete deve ser `Free` na primeira e `2.99` na segunda.

Isso ocorre porque cada chamada de `data()` cria os dados daquela instância, incluindo seu próprio `selectedVariant`. A versão final usa apenas uma tag, com `:premium="premium"`. No teste, selecionar azul no primeiro produto manteve o segundo verde. Os fretes ficaram em Free e 2.99, respectivamente.

### 6. Por que this.cart += 1 no filho não altera o pai?

Dentro do produto, `this` aponta para a instância do produto, não para a raiz. Portanto, `this.cart` não acessa o carrinho do pai.

Na etapa intermediária, `addToCart` apenas obtinha o ID da variante e o mostrava com `console.log`. O contador continuava em zero. Na etapa 2, esse método foi substituído pela emissão do evento.

## Diferenças em relação à referência L9-end

- A promoção foi preservada com onSale, sale e seu parágrafo.
- Na etapa 1, o método provisório addToCart registrava o ID no console.
- A lista de detalhes utiliza :key="detail".
- A imagem possui :alt="title" como alternativa textual.

## Testes da etapa 1

Todos os testes desta etapa passaram. O comportamento do botão nesta tabela corresponde à versão intermediária, anterior aos eventos.

| Teste | Resultado esperado | Resultado obtido |
|---|---|---|
| Abrir a página | Verde disponível e carrinho em zero | Passou |
| Selecionar azul | Imagem azul, indisponível e botão desativado | Passou |
| Adicionar verde | ID 2234 no console e carrinho em zero | Passou |
| Alterar premium para false | Frete muda para 2.99 | Passou |
| Conferir promoção | Mensagem de promoção visível | Passou |
| Conferir detalhes | Três itens de composição visíveis | Passou |
| Testar duas instâncias | Seleções independentes e fretes Free e 2.99 | Passou |
| Conferir console | Sem erros ou avisos do Vue | Passou |

## Etapa 2 — Comunicação por eventos

O `cart` passou de um número para um array de IDs. Em `ProductDisplay.js`, `emits` declara o evento `add-to-cart`. Ao clicar no botão, `addToCart` verifica se há estoque e envia o ID:

```javascript
addToCart() {
  if (!this.inStock) return;

  this.$emit("add-to-cart", this.variants[this.selectedVariant].id);
}
```

No `index.html`, `@add-to-cart="updateCart"` recebe esse evento e chama o método do pai. Em `main.js`, `updateCart(id)` adiciona o ID com `this.cart.push(id)`.

Dois cliques na verde deixam o array como `[2234, 2234]`. O contador mostra `cart.length`, ou seja, dois itens. Dessa forma, quem altera o carrinho é a própria raiz, sem usar `$parent` no produto.

Nesta atividade, adicionar itens não desconta o estoque. O carrinho também não é salvo: ao recarregar a página, ele volta a ficar vazio.

## Testes da etapa 2

| Teste | Resultado esperado | Resultado obtido |
|---|---|---|
| Recarregar | Carrinho vazio e Cart(0) | Passou |
| Adicionar verde duas vezes | Cart(2) e array [2234, 2234] | Passou |
| Selecionar azul | Imagem azul, botão desativado e carrinho inalterado | Passou |
| Voltar à verde e adicionar | Cart(3) e array [2234, 2234, 2234] | Passou |
| Alterar premium para false | Frete 2.99, preservando carrinho e seleção | Passou |
| Conferir detalhes e promoção | Ambos preservados | Passou |
| Conferir console | Sem erros ou avisos do Vue | Passou |

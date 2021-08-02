# Bemarke Docs.

Bem-vindo a documentação do bemarke, esse documento descreverá como as principais
entidades do bemarke funcionam entre si, e como você pode realizar as chamadas para a nossa API pare realizar a integração com o seu serviço.

A API do bemarke é uma API GraphQL. O que significa que temos um PlayGroud
ao qual você pode navegar nos nossos endpoints e testar as suas implementações
usando a extensão para Chrome do Apollo.

Você encontratá a extensão aqui
(https://chrome.google.com/webstore/detail/apollo-client-devtools/jdkknkkbebbapilgoeccciglkfbmbnfm).

Depois que a extensão for adicionada ao Chrome, voce poderá acessa-la pelo menu `Apollo`.
![img.png](https://raw.githubusercontent.com/bemarke/bemarke.github.io/main/images/img.png)
Pronto, voce devera ver o play ground com as principais queries dessa tela
![img_1.png](https://raw.githubusercontent.com/bemarke/bemarke.github.io/main/images/img_1.png).

# Autenticação

O processo de autenticação da nossa API é bem simples. Todos os `requests` realizados na API do bemarke precisam conter uma `apiKey`. Essa apiKey estará vinculada a uma loja já existente no bemarke.

Para obter essa chave de autenticação, basta acessar o sistema do bemarke, no menu lateral esquerdo acessar, Configurações > Integrações > ID.

Podemos testar se o nosso request funciona. Use o comando abaixo no terminal. Lembre de trocar apiKey pela sua respectiva chave.

```
curl -H "Authorization: bearer API_KEY" -H "Content-Type: application/json" -X POST -d '{ "query": "{ loggedUser { name }}"}' staging-api.bemarke.com/graphql
```

Se tudo deu certo, você deverá ver algo parecido como:\
`{"data":{"loggedUser":{"name":"Minha loja de teste"}}}`

### Adendo

Antes de continuar o desenvolvimento da documentação, faremos alguns adendos importantes para facilitar a comunicação da integração. E o primeiro deles é que nossa API é escrita 100% em inglês. Então, lembre-se disso.

Nossa API é baseada na linguagem de consulta GraphQL, ou seja, você sempre realizará uma query ou uma mutation para interagir com os dados.

Sempre que você quiser buscar um ou mais dados, irá realizar uma query, e sempre que quiser alterar algum dado, vai usar uma mutation.

Exemplos:

- Query: `products(variables)`. Se essa query for chamada passando as variables corretas, ela retornara um array de produtos.

- Query: `product(variables)`. Se essa query for chamada passando as variables corretas, ela retornara um único produto.

- Mutation: `saveProduct(variables)`. Se essa mutation for chamada passando as variables corretas, um produto será salvo, o retorno dessa mutation é o próprio produto salvo.

Antes de continuar, vamos também analisar o esqueleto de uma consulta ou mutação, elas sempre possuem o mesmo padrão e sempre tem alguma coisa a nos dizer.

`products(storeId: ID, storeName: String): [Product]`

Essa é a declaração do schema da query que busca um produto. Vamos analisar. Temos o nome da query, products, ou seja, busca vários produtos. Para que possamos buscar uma lista de produtos, precisamos passar o storeId que é o ID da loja do nosso lado, ou o nome da loja. Simples assim. Logo após dos dois pontos, “:”. Temos o tipo de retorno, note que temos um array e dentro desse array o tipo do retorno. Isso indica para gente que essa consulta retorna um array de produtos.

As nossas queries e mutations seguem esse mesmo padrão, sempre assim.

Vamos ver uma mutation agora.

`saveProduct(product: ProductInput!): Product`

Nome da mutation, saveProducts, ele espera como parâmetro um `input` product, o que é isso?
É simples, um objetivo chamado product, esse objeto é do tipo ProductInput, e no final
desse ProductInput possui o ponto de exclamação “!” que indica que esse campo é obrigatório. O retorno dessa mutation é um Product, o que você acabou de criar.


_Vamos agora dar uma passada pelas principais entidades do bemarke. Se você ler atentamente cada uma delas, terá uma visão muito mais clara que como as coisas são
conectadas do nosso lado._

# Categoria

São as categorias dos produtos, também podem ser chamados de grupo e são uma das principais entidades do bemarke, as categorias são importantes porque nelas conseguimos
vincular complementos, tamanhos, fracionarios, frações incompletas e valor produto fracionario cobrado pelo mais caro. Além disse, é vinculado a categoria um tipo de categoria, que é a categoria padronizada do bemarke (market place). Esses tipos possíveis estão definidos no tipe categoryTypes.

```javascript
const request = require('request');

// Link de suporte para mutations = https://graphql.org/graphql-js/graphql-clients/;
// Link de suporte para queries = https://graphql.org/graphql-js/mutations-and-input-types/

// DECLARAÇÃO = saveCategory(category: CategoryInput!): Category

const queryString = `
  mutation saveCategory($category: CategoryInput!) {
   saveCategory(category: $category) {
    _id
    name
   }
  }
`;

const query = {
  query: queryString,
  variables: {
    category: {
      name: 'Pizzas',
      description: 'A melhor pizza da região.',
      isVisible: true,
      categoryType: {
        _id: '23',
      },
      image:
        'https://blog.praticabr.com/wp-content/uploads/2019/06/Pizza-Pizzaria-Forno-Forza-Express.jpg', // Pego da internet.
      complementsIds: ['HviiX3fWRdu3ZR3LW', 'xceExiw9EfT3r7xTC'], // Ids gerados dinamicamente na criação de complementos
      sizesIds: ['puNZHg35Ldh2wCqHK', 'o7MGKm4Hv73eyTZxx', 'wXDZiAE8E758QH9PL'],
      fractionariesIds: ['X6PnPLFoMC5nyJ477', '49bEakrWKtukxapmY'],
      acceptIncompleteFraction: true,
      isPriceByMoreExpensive: false,
      isWithFraction: true,
      isWithSize: true,
      orderType: 'BY_PRODUCT',
    },
  },
};

const options = {
  url: 'https://staging-api.bemarke.com.br/graphql',
  headers: {
    Authorization: 'bearer XXX',
    'Content-Type': 'application/json',
  },
  json: query,
};

request.post(options, (err, httpResponse, body) => {
  console.log('body: ', body);
});
```

Vamos conversar um pouco sobre essa mutation. Bom, primeiro definidos a mutationString. A declaração sempre é assim, se estiver com dúvidas de como declarar uma query ou mutation, vá até o playground.

Uma vez definido, precisamos saber quais parâmetros passar para a propriedade variables, mais uma vez, precisamos ir até o playground, o playground vai nos dizer que quer um objeto chamado category do tipo CategoryInput.

Sempre que criamos uma categoria, devemos passar as propriedades name, description e categoryTtype.
Para saber qual categoryTtype devemos usar, basta acessarmos o playground e rodar essa query:

```
{
  storeTypes {
    _id
    name
  }
}
```

#### Queries de categorias:

```
  categories(storeId: ID, storeName: String): [Category]
  category(_id: ID!): Category
```

#### Mutations de categorias

```
  saveCategory(category: CategoryInput!): Category
  changeOrderCategory(_id: ID!, order: Int!): Category
```

Se não sabe fazer isso ainda, recomendamos fortemente que assista o vídeo descrito no início deste documento.

Os campos `isWithSize,isWithFraction, isPriceByMoreExpensive, acceptIncompleteFraction, fractionariesIds, sizesIds e complementsIds` serão tratados mais a frente.

O campo `isVisible` indica se a categoria está visível na loja.
Se a categoria estiver invisível, todos os produtos a ela também estará, pois ela não será listada nas categorias na tela de compra.

Todas as entidades possuem um campo chamado `integrationId`, este campo é para você tratar assuntos de integração, ele é indiferente a nós. Pode ser, por exemplo, o identificador do produto em outro sistema.

Lembre-se que o retorno da mutation ou query nunca vem por completo a não ser que especifique, note que eu pedi no retorno dessa mutation apenas `_id, name e description`. Caso queria todos os campos da entidade, você precisa pedir.

# Produtos

São os produtos, que podem ter complementos (`complementsIds`) e
fracionários (`isWithFraction e fractionariesIds`), devem ter uma categoria
(`category._id`) e quando tiverem vários tamanhos o preço deve ser informado por tamanho
(`prices` do tipo `ProductSizePriceInput`).

#### Queries:

```
  products(storeId: ID, storeName: String): [Product]
  productsPaginated(storeId: ID, search: String, categoryId: ID, onlyInvisible: Boolean, loyaltyOnly: Boolean, paginationAction: PaginationActionInput): PaginatedList
  product(_id: ID!, isMarketPlace: Boolean): Product
  promotions(city: String, state: String, storeId: ID, slug: String): [Product]
  loyalties(city: String, state: String, storeId: ID, slug: String): [Product]
  allPromotions(city: String, state: String, storeId: ID, slug: String, includeDisabled: Boolean): [Product]
  promotionBanner(city: String, state: String, storeId: ID): PromotionBanner
```

#### Mutations:

```
  saveProduct(product: ProductInput!): Product
  saveProductsOrder(productsOrder: [ProductsOrderInput]!): [Product]
  changeProductVisibility(productId: ID!, isVisible: Boolean!): Product
  removeProductOrProductsFromStore(productId: ID, allProducts: Boolean): [Product]
```

Chamada com a criação de um produto

```javascript
const request = require('request');

// DECLARAÇÃO = saveCategory(category: CategoryInput!): Category

// Link de suporte para mutations = https://graphql.org/graphql-js/graphql-clients/;
// Link de suporte para queries = https://graphql.org/graphql-js/mutations-and-input-types/

const queryString = `
  mutation saveProduct($product: ProductInput!) {
   saveProduct(product: $product) {
    _id
    name
   }
  }
`;

const query = {
  query: queryString,
  variables: {
    product: {
      name: 'Pizza de queijo',
      image:
        'https://www.google.com.br/url?sa=i&url=https%3A%2F%2Fwww.receitas-sem-fronteiras.com%2Freceita-69175-receita-de-pizza-de-mussarela.htm&psig=AOvVaw2-jxwwhSvWbe5lnLDaoab-&ust=1627672903484000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCOCd6aiAifICFQAAAAAdAAAAABAJ',
      description: 'Pizza de queijo muito boa',
      price: 4790, // R$ 47,90
      discountPrice: 500, // R$ 5,00 reais de desconto
      isVisible: true,
      isWithFraction: true,
      category: {
        _id: '8FHBdmouJhTHNDYSp', // Id gerado dinamicamente na criação da categoria.
      },
      order: 0,
    },
  },
};

const options = {
  url: 'https://staging-api.bemarke.com.br/graphql',
  headers: {
    Authorization: 'bearer XXX',
    'Content-Type': 'application/json',
  },
  json: query,
};

request.post(options, (err, httpResponse, body) => {
  console.log('body: ', body);
});

// RETORNO =
// body:  {
//   data: {
//     saveProduct: { _id: 'aqRRyGiyW4DNfevp9', name: 'Pizza de queijo' }
//   }
// }
```

# Tamanhos
São os tamanhos que os produtos de uma categoria estão disponíveis.
Você pode informar que um tamanho possui frações (`isWithFraction e fractionariesIds`).
Também é possível informar se aceita vender frações incompletas, ou seja, meia porção
(`acceptIncompleteFraction`).
E ainda é possível informar se cobra pela fração mais cara ou proporcionalmente (`isPriceByMoreExpensive`).
Ambos os campos apenas quando existem fracionários é que serão utilizados.

#### Queries:

```
  sizes(storeId: ID, storeName: String): [Size]
  size(_id: ID!): Size
```

#### Mutations:

```
  saveSize(size: SizeInput!): Size
  changeOrderSize(_id: ID!, order: Int!): Size
```

Chamada com a criação de um tamanho

```javascript
const request = require('request');

// DECLARAÇÃO = saveCategory(category: CategoryInput!): Category

// Link de suporte para mutations = https://graphql.org/graphql-js/graphql-clients/;
// Link de suporte para queries = https://graphql.org/graphql-js/mutations-and-input-types/

const queryString = `
  mutation saveSize($size: SizeInput!) {
    saveSize(size: $size) {
      # [COMENTÁRIO] Retorno da mutation.
      _id
      name
    }
  }
`;

const query = {
  query: queryString,
  variables: {
    size: {
      name: 'Tamanho pequeno',
      fractionariesIds: ['X6PnPLFoMC5nyJ477', '49bEakrWKtukxapmY'], // Ids gerados dinamicamente na criação de fracionário.
      acceptIncompleteFraction: true,
      isPriceByMoreExpensive: false,
      isWithFraction: true,
      isVisible: true,
    },
  },
};

const query2 = {
  query: queryString,
  variables: {
    size: {
      name: 'Tamanho medio',
      fractionariesIds: ['X6PnPLFoMC5nyJ477', '49bEakrWKtukxapmY'], // Ids gerados dinamicamente na criação de fracionário.
      isPriceByMoreExpensive: true, // Preço definido pelo mais caro
      isWithFraction: true,
      isVisible: true,
    },
  },
};

const query3 = {
  query: queryString,
  variables: {
    size: {
      name: 'Tamanho grande',
      isVisible: true,
    },
  },
};

const options1 = {
  url: 'https://staging-api.bemarke.com.br/graphql',
  headers: {
    Authorization: 'bearer 043FC6B3-44A3-4701-A498-DB3EA0205DF0',
    'Content-Type': 'application/json',
  },
  json: query,
};

const options2 = {
  url: 'https://staging-api.bemarke.com.br/graphql',
  headers: {
    Authorization: 'bearer XXX',
    'Content-Type': 'application/json',
  },
  json: query2,
};

const options3 = {
  url: 'https://staging-api.bemarke.com.br/graphql',
  headers: {
    Authorization: 'bearer XXX',
    'Content-Type': 'application/json',
  },
  json: query3,
};

request.post(options1, (err, httpResponse, body) => {
  console.log('body1: ', body);
});

request.post(options2, (err, httpResponse, body) => {
  console.log('body2: ', body);
});

request.post(options3, (err, httpResponse, body) => {
  console.log('body3: ', body);
});

// RETORNO =
// body1:  {
//   data: { saveSize: { _id: 'o7MGKm4Hv73eyTZxx', name: 'Tamanho pequeno' } }
// }
// body2:  {
//   data: { saveSize: { _id: 'puNZHg35Ldh2wCqHK', name: 'Tamanho medio' } }
// }
// body3:  {
//   data: { saveSize: { _id: 'wXDZiAE8E758QH9PL', name: 'Tamanho medio' } }
// }
```

# Fracionários
São as frações que podem ser vendidas de um tamanho, categoria ou produto,
se ele não for informado significa que o produto é sempre vendido por inteiro.
Uma fração consiste em duas propriedades importantes: o multiplicador (`multiplier`)
e a fração (`fraction`). O multiplicador afeta o preço e a fração afeta a quantidade.
Ou seja, se você quiser vincular sua categoria pizza a um fracionário de
3 sabores você deveria informar que o multiplicador é `0.333`,
ou seja ⅓ do preço e a fração também.
Já para uma porção muitas vezes os estabelecimentos cobram 70% por meia porção
e então a fração ficaria `0.5` e o multiplicador `0.7`.

#### Queries:

```
  fractionaries(storeId: ID, storeName: String): [Fractionary]
  fractionary(_id: ID!): Fractionary
```

#### Mutations

```
  saveFractionary(fractionary: FractionaryInput!): Fractionary
```

Chamada com a criação de um fracionario

```javascript
const request = require('request');

// DECLARAÇÃO = saveCategory(category: CategoryInput!): Category

// Link de suporte para mutations = https://graphql.org/graphql-js/graphql-clients/;
// Link de suporte para queries = https://graphql.org/graphql-js/mutations-and-input-types/

const queryString = `
  mutation saveFractionary($fractionary: FractionaryInput!) {
    saveFractionary(fractionary: $fractionary) {
      # [COMENTÁRIO] Retorno da mutation.
      _id
      name
    }
  }
`;

const query = {
  query: queryString,
  variables: {
    fractionary: {
      name: 'Meia porção',
      fraction: 0.5,
      multiplier: 0.5,
      isVisible: true,
    },
  },
};

const query2 = {
  query: queryString,
  variables: {
    fractionary: {
      name: 'Porção completa',
      fraction: 1,
      multiplier: 1,
      isVisible: true,
    },
  },
};

const options1 = {
  url: 'https://staging-api.bemarke.com.br/graphql',
  headers: {
    Authorization: 'bearer XXX',
    'Content-Type': 'application/json',
  },
  json: query,
};

const options2 = {
  url: 'https://staging-api.bemarke.com.br/graphql',
  headers: {
    Authorization: 'bearer XXX',
    'Content-Type': 'application/json',
  },
  json: query2,
};

request.post(options1, (err, httpResponse, body) => {
  console.log('body1: ', body);
});

request.post(options2, (err, httpResponse, body) => {
  console.log('body2: ', body);
});

// RETORNO =
// body1:  {
//   data: {
//     saveFractionary: { _id: 'X6PnPLFoMC5nyJ477', name: 'Meia porção' }
//   }
// }
// body2:  {
//   data: {
//     saveFractionary: { _id: '49bEakrWKtukxapmY', name: 'Porção completa' }
//   }
// }
```

# Complementos
São complementos de uma categoria ou produto, muitas vezes também chamados de adicionais.
Caso um complemento faça parte de um grupo você deve informar o `groupId`.
Complementos também podem ser do tipo "com opções" (`isWithOptions`) o que significa
que ele terá uma lista de opções disponíveis, por ex, tipo do pão,
ponto da carne, etc. As opções (`complementOptions`)
também possuem seu Input, seguindo o padrão.

#### Queries:

```
  complements(storeId: ID, storeName: String): [Complement]
  complement(_id: ID!): Complement
```

#### Mutations

```
  saveComplement(complement: ComplementInput!): Complement
  changeOrderComplement(_id: ID!, order: Int!): Complement
```

Chamada com a criação de um complemento

```javascript
const request = require('request');

// Link de suporte para mutations = https://graphql.org/graphql-js/graphql-clients/;
// Link de suporte para queries = https://graphql.org/graphql-js/mutations-and-input-types/

// DECLARAÇÃO = saveCategory(category: CategoryInput!): Category

const queryString = `
  mutation saveComplement($complement: ComplementInput!) {
    saveComplement(complement: $complement) {
      # [COMENTÁRIO] Retorno da mutation.
      _id
      name
    }
  }
`;

const query = {
  query: queryString,
  variables: {
    complement: {
      name: 'Adicional de bacon',
      isVisible: true,
      price: 390,
      groupId: 'eRD7CxrzSXAA4MLFd', // Esse ID foi gerado dinamicamente na criação de um grupo de complemento.
    },
  },
};

const query2 = {
  query: queryString,
  variables: {
    complement: {
      name: 'Adicional de calabresa',
      isVisible: true,
      price: 520,
      groupId: 'eRD7CxrzSXAA4MLFd', // Esse ID foi gerado dinamicamente na criação de um grupo de complemento.
    },
  },
};

const options1 = {
  url: 'https://staging-api.bemarke.com.br/graphql',
  headers: {
    Authorization: 'bearer XXX',
    'Content-Type': 'application/json',
  },
  json: query,
};

const options2 = {
  url: 'https://staging-api.bemarke.com.br/graphql',
  headers: {
    Authorization: 'bearer XXX',
    'Content-Type': 'application/json',
  },
  json: query2,
};

request.post(options1, (err, httpResponse, body) => {
  console.log('body1: ', body);
});

request.post(options2, (err, httpResponse, body) => {
  console.log('body2: ', body);
});

// RETORNO =
// body1:  {
//   data: {
//     saveComplement: { _id: 'HviiX3fWRdu3ZR3LW', name: 'Adicional de bacon' }
//   }
// }
// body2:  {
//   data: {
//     saveComplement: { _id: 'xceExiw9EfT3r7xTC', name: 'Adicional de calabresa' }
//   }
// }
```

# Grupos de Complemento
Agrupam complementos para facilitar a escolha do cliente na tela.
Também pode ser definido um limite (`selectionsLimit`) de complementos
que podem ser escolhidas dentro desse grupo.

#### Queries:

```
  complementGroups(storeId: ID, storeName: String): [ComplementGroup]
  complementGroup(_id: ID!): ComplementGroup
```

#### Mutations

```
  saveComplementGroup(complementGroup: ComplementGroupInput!): ComplementGroup
```

Chamada com a criação de um grupos de Complemento

```javascript
const request = require('request');

// Link de suporte para mutations = https://graphql.org/graphql-js/graphql-clients/;
// Link de suporte para queries = https://graphql.org/graphql-js/mutations-and-input-types/

const mutationString = `
  mutation saveComplementGroup($complementGroup: ComplementGroupInput!) {
    saveComplementGroup(complementGroup: $complementGroup) {
      # [COMENTÁRIO] Retorno da mutation.
      _id
      name
      selectionsLimit
    }
  }
`;

const query = {
  query: mutationString,
  variables: {
    complementGroup: {
      name: 'Molhos',
      selectionsLimit: 2,
      integrationId: '1', // Não é obrigatório, esse campo é para controle do lado do serviço que está integrando.
    },
  },
};

const options = {
  url: 'https://staging-api.bemarke.com.br/graphql',
  headers: {
    Authorization: 'bearer XXX',
    'Content-Type': 'application/json',
  },
  json: query,
};

request.post(options, (err, httpResponse, body) => {
  console.log('body: ', body);
});

// Retorno =
// body:  {
//   data: {
//     saveComplementGroup: { _id: 'eRD7CxrzSXAA4MLFd', name: 'Molhos', selectionsLimit: 2 }
//   }
// }

// [IMPORTANT] USAREMOS O _ID PARA CRIAR UM COMPLEMENTO DE GRUPO.
```

# Orders

Como buscar uma order pendente?

```javascript
const request = require('request');

// Link de suporte para mutations = https://graphql.org/graphql-js/graphql-clients/;
// Link de suporte para queries = https://graphql.org/graphql-js/mutations-and-input-types/

const queryString = `
  query orders($storeId: ID, $statuses: [OrderStatus], $statusesSearch: [OrderStatus], $deliveryStatusesSearch: [PlayDeliveryStatus!], $period: Period, $specificOrderDate: DateTime, $orderLogisticType: OrderLogisticType, $logisticMethod: LogisticMethod, $whiteLabelStoreId: ID, $ordersLimit: Int, $search: String){
    orders(storeId: $storeId, statuses: $statuses, dateStart: $dateStart, dateEnd: $dateEnd, statusesSearch: $statusesSearch, deliveryStatusesSearch: $deliveryStatusesSearch, period: $period, specificOrderDate: $specificOrderDate, whiteLabelStoreId: $whiteLabelStoreId, ordersLimit: $ordersLimit, orderLogisticType: $orderLogisticType, logisticMethod: $logisticMethod, search: $search) {
      _id
      sequence
      status
      deliveryStatus
      deliveryAddress {
        _id
        zipcode
        address
        number
        complement
        neighborhood
        city
        state
        state
      }
      isPaymentByPix
      store {
        _id
        name
        
      }
      customer {
        _id
        name
        user {
          email
        }
      }
    }
  }
`;

const query = {
  query: queryString,
  variables: {
    storeId: 'ID_DA_LOJA',
    statuses: ['PENDING'],
    /*
     * PARA OBTER UMA ORDER DIFERENTE DE PENDING, BASTA ALTERAR O STATUSES.
     * OS POSSÍVEIS VALORES SÃO: DRAFT, PENDING, SCHEDULE_ACCEPTED, ACCEPTED, DISPATCHED, CANCELED, EXPIRED, REJECTED E FINISHED.
     * PARA OBTER AS ORDERS COM MAIS DE UM STATUS, BASTA COLOC-LOS NO ARRAY DE STATUSES, [PENDING, ACCPET, EXPIRED, REJECT], POR EXEMPLO.
     * */
  },
};

const options = {
  url: 'https://staging-api.bemarke.com.br/graphql',
  json: query,
  headers: {
    Authorization: 'bearer XXX',
    'Content-Type': 'application/json',
  },
};

request.post(options, (err, httpResponse, body) => {
  console.log('body: ', body);
});
```

Embora na definição da query, existam muitos campos, eles são campos específicos para que seja possível fazer
consultas mais complexas, esses campos não são necessariamente usados em uma query. Na maioria dos casos, as nossas
queries terão diversos campos que provavelmente não serão usados em uma query simples. Quando houver necessidade
de fazer uma consulta mais complexa, esses campos serão utils.

# Order

Como buscar uma order específica? Simples, o retorno funciona da mesma maneira, só precisamos variar a query 
seguindo o playground do GraphQl.

```javascript
const request = require("request");

// Link de suporte para mutations = https://graphql.org/graphql-js/graphql-clients/;
// Link de suporte para queries = https://graphql.org/graphql-js/mutations-and-input-types/

const queryString = `
  query Order(
    $orderId: ID!
    $statuses: [OrderStatus]
    $anonymousId: String
    $whiteLabelStoreId: ID
  ) {
    order(
      _id: $orderId
      statuses: $statuses
      anonymousId: $anonymousId
      whiteLabelStoreId: $whiteLabelStoreId
    ) {
      _id
      sequence
      status
      deliveryStatus
      statusHistory {
        _id
        previousStatus
        currentStatus
        dateTime
      }
      combos {
        _id
        total
        items {
          _id
          product {
            _id
            name
            price
            createdAt
          }
          complements {
            _id
            quantity
          }
          fractionary {
            _id
          }
        }
        quantity
        comment
      }
      calculatedDistanceInMeters
      isPayingBill
      amountInfluencerDiscountApplied
      applyFirstPurchaseDiscount

      deliveryAddress {
        lastUsedAt
        _id
        zipcode
        number
        address
        complement
        neighborhood
        city
        state
        reference
      }
      paymentAddress {
        lastUsedAt
        _id
        zipcode
        number
        address
        complement
        neighborhood
        city
        state
        reference
      }
      payment {
        _id
        status
      }
      isPaymentOnlineEnable
      logisticMethod
      originalScheduledTo
      isPaymentByPix
      customer {
        _id
        name
        user {
          email
        }
      }

      discounts {
        _id
        amount
        description
        storePromotionId
        storePromotion {
          _id
          name
        }
      }
      orderLogisticType
      orderLogisticType
      paymentChoices
      paymentMethodGroupFee
      subtotal
      total
      store {
        _id
        name
        isOpen
      }
      amountBill
      createdAt
      updatedAt
    }
  }
`;

const query = {
  query: queryString,
  variables: {
    _id: "ID_DA_ORDER",
    statuses: ["DISPATCHED"],
    /*
     * PARA OBTER UMA ORDER DIFERENTE DE PENDING, BASTA ALTERAR O STATUSES.
     * OS POSSÍVEIS VALORES SÃO: DRAFT, PENDING, SCHEDULE_ACCEPTED, ACCEPTED, DISPATCHED, CANCELED, EXPIRED, REJECTED E FINISHED.
     * */
  },
};

const options = {
  url: "https://staging-api.bemarke.com.br/graphql",
  json: query,
  headers: {
    Authorization: "bearer XXX",
    "Content-Type": "application/json",
  },
};

request.post(options, (err, httpResponse, body) => {
  console.log("body: ", body);
});

```


# Order Status


E como podemos alterar o status de uma order? Basta chamarmos uma mutation chamada `changeOrderStatus`.
Nós precisamos de duas coisas, o _id da order que será alterada, e também o novo status que a order terá.

Vamos assumir a order `n597hmTXfTuscQn8t` no status `ACCPETED`, agora precisamos terminar a order,
mandando-o para o status de `FINISHED`. Basta chamar a mutation abaixo:
```javascript
const request = require("request");

// Link de suporte para mutations = https://graphql.org/graphql-js/graphql-clients/;
// Link de suporte para queries = https://graphql.org/graphql-js/mutations-and-input-types/

const queryString = `
  mutation ChangeOrderStatus(
    $orderId: ID!
    $status: OrderStatus
    $cancellationReason: String
    $canceledBy: CanceledBy
  ) {
    changeOrderStatus(
      orderId: $orderId
      status: $status
      canceledBy: $canceledBy
      cancellationReason: $cancellationReason
    ) {
      _id
      status
    }
  }
`;

const query = {
  query: queryString,
  variables: {
    orderId: "n597hmTXfTuscQn8t",
    status: "FINISHED",
  },
};

const options = {
  url: "https://staging-api.bemarke.com.br/graphql",
  json: query,
  headers: {
    Authorization: "bearer XXX",
    "Content-Type": "application/json",
  },
};

request.post(options, (err, httpResponse, body) => {
  console.log("body: ", body);
});

```
Observe que a mutation tem dois campos a mais, `canceledBy` e `cancellationReason`, esses parâmetros
devem ser passado apenas se o novo status da order for `CANCELED`, `REJECT`, ou `EXPIRED`, ou seja, a order
por algum motivo esta sendo abortada. 

O retorno da mutation é do tipo Order, mas pegamos apenas o status.


#### Como funciona os status
 * As order no status de `DRAFT` estão apenas no carrinho, não ouve
transação. 
 * As order no status de `PENDING` estão em processo de aceite pelo lojista no bemarke, ou seja, a order foi paga, caso
o pagamento tenha sido feito via aplicativo, via pix ou cartão.
 * As order no status de `ACCEPTED`, foram, aceitas, e logo entraram em processo de entrega.
 * As orders no status de `DISPATCHED`,  são order para entregarem, e esse status indica que a entrega saiu para ser entregada.
 * As order no status de `FINISHED`, encerram o seu ciclo com sucesso, seja retirada, entrega ou consumo.
 * As order no status de `CANCELED`, foram canceladas por algum motivo, seja falta de entregador, cliente muito longe, etc.
 * As order no status de `EXPIRED`, não foram aceitas pelo lojista em tempo hábil.

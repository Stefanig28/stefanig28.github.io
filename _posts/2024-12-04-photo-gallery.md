---
layout: post
title: How I Implemented a Shopping Cart Step by Step
date: 2025-10-10 11:59:00
description: In this post, I explain step by step how I developed and integrated a fully functional shopping cart using the Shopify API
tags: formatting code images
categories: sample-posts
thumbnail: assets/img/1.jpg
featured: true
---

Next, I’ll explain how I implemented a shopping cart in a project using the Shopify API.

Step 1:

I created a cart with a predefined variant and logged it in the console. 

I started by calling the Shopify API using my `shopifyRequest()` function: para eso necesite configurar unos datos

```js
const shopDomain = "bu1fib-rq.myshopify.com";
const storefrontToken = "83e93e5462232411b85e227dad586a69";
const apiUrl = `https://${shopDomain}/api/2024-10/graphql.json`;

async function shopifyRequest(query, variables = {}) {
  const response = await fetch(apiUrl, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-Shopify-Storefront-Access-Token": storefrontToken,
    },
    body: JSON.stringify({ query, variables }),
  });

  const data = await response.json();
  return data;
}
```

defini la variante que voy a usar


```js
const variantId = `gid://shopify/ProductVariant/46805481881826`;
```

Then, I created my `createCart()` function using GraphQL

```js
async function createCart() {
  const mutation = `
    mutation CreateCart($input: CartInput!) {
      cartCreate(input: $input) {
        cart {
          id
          createdAt
          lines(first: 10) {
            edges {
              node {
                id
                quantity
                merchandise {
                  ... on ProductVariant {
                    id
                    title
                    price {
                      amount
                      currencyCode
                    }
                  }
                }
              }
            }
          }
        }
        userErrors {
          field
          message
        }
      }
    }
  `;

  const variables = {
    input: {
      lines: [
        {
          merchandiseId: variantId,
          quantity: 1
        },
      ]
    }
  };

  const result = await shopifyRequest(mutation, variables);

  if (result.errors || result.data.cartCreate.userErrors.length > 0) {
    console.error("GraphQL Errors:", result.errors || result.data.cartCreate.userErrors);
    throw new Error("Failed to create cart");
  }

  const cart = result.data.cartCreate.cart;
  console.log("Cart Created", cart);
  console.log("Cart ID:", cart.id);
}
```

At the moment, my function prints two things to the console:
   1. This logs the cart along with the message “Cart Created”.

      ```js
      console.log("Cart Created", cart);
      ```

   2. This logs the Cart ID, which we’ll need later.
      ```js
      console.log("Cart ID:", cart.id);
      ```

Finally, I call the function:

```js
await createCart();
```

Así quedaría el código completo:

{% highlight js linenos %}

const shopDomain = "bu1fib-rq.myshopify.com";
const storefrontToken = "83e93e5462232411b85e227dad586a69";
const apiUrl = `https://${shopDomain}/api/2024-10/graphql.json`;

const variantId = `gid://shopify/ProductVariant/46805481881826`;

async function shopifyRequest(query, variables = {}) {
  const response = await fetch(apiUrl, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-Shopify-Storefront-Access-Token": storefrontToken,
    },
    body: JSON.stringify({ query, variables }),
  });

  const data = await response.json();
  return data;
}

async function createCart() {
  const mutation = `
    mutation CreateCart($input: CartInput!) {
      cartCreate(input: $input) {
        cart {
          id
          createdAt
          lines(first: 10) {
            edges {
              node {
                id
                quantity
                merchandise {
                  ... on ProductVariant {
                    id
                    title
                    price {
                      amount
                      currencyCode
                    }
                  }
                }
              }
            }
          }
        }
        userErrors {
          field
          message
        }
      }
    }
  `;

  const variables = {
    input: {
      lines: [
        {
          merchandiseId: variantId,
          quantity: 1
        },
      ]
    }
  };

  const result = await shopifyRequest(mutation, variables);

  if (result.errors || result.data.cartCreate.userErrors.length > 0) {
    console.error("GraphQL Errors:", result.errors || result.data.cartCreate.userErrors);
    throw new Error("Failed to create cart");
  }

  const cart = result.data.cartCreate.cart;
  console.log("Cart Created", cart);
  console.log("Cart ID:", cart.id);
}

await createCart();

{% endhighlight %}

Y así se ve en mi cónsolaa

<div style="text-align: center;">
  <img width="800" alt="Shopify cart preview" src="https://github.com/user-attachments/assets/42088928-e9c1-4c5e-9020-96b309bf383b" />
</div>

Step 2:

Añade items al carrito

Primero verificamos si el producto que vamos a añadir existe, para eso usamos una función que revise si hay en stock

```js
async function checkVariantStock(variantId) {
  const query = `
    query VariantStockCheck($id: ID!) {
      node(id: $id) {
        ... on ProductVariant {
          id
          title
          availableForSale
          quantityAvailable
        }
      }
    }
  `;

  const variables = {
    id: variantId
  };

  const response = await shopifyRequest(query, variables);

  if (response.errors) {
    console.error("GraphQL Errors:", response.errors);
    throw new Error("Failed to fetch variant stock");
  }

  const variant = response.data.node;

  if (!variant) {
    console.warn("Variant not found");
    return null;
  }

  console.log(`Variant: ${variant.title}`);
  console.log(`Available for sale: ${variant.availableForSale}`);
  console.log(`Quantity available: ${variant.quantityAvailable}`);

  return {
    id: variant.id,
    title: variant.title,
    availableForSale: variant.availableForSale,
    quantityAvailable: variant.quantityAvailable
  };
}
```

Creamos la función para añadir un item al carrito 

```js
async function addItemToCart(cartId, variantId, quantity = 1) {
  const mutation = `
    mutation CartLinesAdd($cartId: ID!, $lines: [CartLineInput!]!) {
      cartLinesAdd(cartId: $cartId, lines: $lines) {
        cart {
          id
          totalQuantity
          lines(first: 10) {
            edges {
              node {
                id
                quantity
                merchandise {
                  ... on ProductVariant {
                    id
                    title
                  }
                }
              }
            }
          }
        }
        userErrors {
          field
          message
        }
      }
    }
  `;

  const variables = {
    cartId,
    lines: [
      {
        merchandiseId: variantId,
        quantity
      }
    ]
  };

  const result = await shopifyRequest(mutation, variables);

  if (result.errors || result.data.cartLinesAdd.userErrors.length > 0) {
    console.error("GraphQL Errors:", result.errors || result.data.cartLinesAdd.userErrors);
    throw new Error("Failed to add item to cart");
  }
  
  const updatedCart = result.data.cartLinesAdd.cart;
  console.log("Item added to cart", updatedCart);
  return updatedCart;
}
```

Ahora no vamos simplemente a llamar a la función, vamos a hacer algo un poco diferente, vamos a decir que cuando se cargue el dom creamos un carrito vacío, verificamos el stock dle producto que vamos a añadir llamando a nuestra función `checkVariantStock(variantId)` y por ultimo agregamos el producto al carrito. El código se vería de esta forma:

```js
document.addEventListener("DOMContentLoaded", async () => {
  const cartId = await createCart(); 
  await checkVariantStock(variantId)
  await addItemToCart(cartId, variantId, 1);
});
```

Así se vería el step 2

```js
const shopDomain = "bu1fib-rq.myshopify.com";
const storefrontToken = "83e93e5462232411b85e227dad586a69";
const apiUrl = `https://${shopDomain}/api/2024-10/graphql.json`;

const variantId = `gid://shopify/ProductVariant/46880650297570`;

async function shopifyRequest(query, variables = {}) {
  const response = await fetch(apiUrl, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-Shopify-Storefront-Access-Token": storefrontToken,
    },
    body: JSON.stringify({ query, variables }),
  });

  const data = await response.json();
  return data;
}

async function createCart() {
  const mutation = `
    mutation CreateCart($input: CartInput!) {
      cartCreate(input: $input) {
        cart {
          id
          createdAt
          lines(first: 10) {
            edges {
              node {
                id
                quantity
                merchandise {
                  ... on ProductVariant {
                    id
                    title
                    price {
                      amount
                      currencyCode
                    }
                  }
                }
              }
            }
          }
        }
        userErrors {
          field
          message
        }
      }
    }
  `;
  
  const variables = {
    input: {} 
  };

  const result = await shopifyRequest(mutation, variables);

  if (result.errors || result.data.cartCreate.userErrors.length > 0) {
    console.error("GraphQL Errors:", result.errors || result.data.cartCreate.userErrors);
    throw new Error("Failed to create cart");
  }

  const cart = result.data.cartCreate.cart;
  console.log("Cart Created", cart);
  console.log("Cart ID:", cart.id);
  
  return cart.id
}

async function addItemToCart(cartId, variantId, quantity = 1) {
  const mutation = `
    mutation CartLinesAdd($cartId: ID!, $lines: [CartLineInput!]!) {
      cartLinesAdd(cartId: $cartId, lines: $lines) {
        cart {
          id
          totalQuantity
          lines(first: 10) {
            edges {
              node {
                id
                quantity
                merchandise {
                  ... on ProductVariant {
                    id
                    title
                  }
                }
              }
            }
          }
        }
        userErrors {
          field
          message
        }
      }
    }
  `;

  const variables = {
    cartId,
    lines: [
      {
        merchandiseId: variantId,
        quantity
      }
    ]
  };

  const result = await shopifyRequest(mutation, variables);

  if (result.errors || result.data.cartLinesAdd.userErrors.length > 0) {
    console.error("GraphQL Errors:", result.errors || result.data.cartLinesAdd.userErrors);
    throw new Error("Failed to add item to cart");
  }
  
  const updatedCart = result.data.cartLinesAdd.cart;
  console.log("Item added to cart", updatedCart);
  return updatedCart;
}

async function checkVariantStock(variantId) {
  const query = `
    query VariantStockCheck($id: ID!) {
      node(id: $id) {
        ... on ProductVariant {
          id
          title
          availableForSale
          quantityAvailable
        }
      }
    }
  `;

  const variables = {
    id: variantId
  };

  const response = await shopifyRequest(query, variables);

  if (response.errors) {
    console.error("GraphQL Errors:", response.errors);
    throw new Error("Failed to fetch variant stock");
  }

  const variant = response.data.node;

  if (!variant) {
    console.warn("Variant not found");
    return null;
  }

  console.log(`Variant: ${variant.title}`);
  console.log(`Available for sale: ${variant.availableForSale}`);
  console.log(`Quantity available: ${variant.quantityAvailable}`);

  return {
    id: variant.id,
    title: variant.title,
    availableForSale: variant.availableForSale,
    quantityAvailable: variant.quantityAvailable
  };
}


document.addEventListener("DOMContentLoaded", async () => {
  const cartId = await createCart(); 
  await checkVariantStock(variantId)
  await addItemToCart(cartId, variantId, 1);
});
```

<div style="text-align: center;">
  <img width="800" alt="Screenshot 2025-09-26 at 3 07 20 PM" src="https://github.com/user-attachments/assets/0f11b6aa-f15c-48aa-a369-66477ce5ede6" />
</div>

Step 3:

Este paso consiste en lograr que cuando le doy click al boton cart se despliegue un modal con el item agregado 




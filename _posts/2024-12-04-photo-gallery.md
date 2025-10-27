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

Step 1: Creating the Cart

I started by creating a helper function called `shopifyRequest()` to handle all Shopify API calls. For that, I needed to configure some data such as the Shopify store domain, the storefront access token, and the API endpoint URL. 

```js
const shopDomain = "your-shop-name.myshopify.com";
const storefrontToken = "your-storefront-access-token";
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

Once the request function was ready, I defined a product `variantId` that I wanted to use for testing.


```js
const variantId = "gid://shopify/ProductVariant/your-variant-id";
```

Then, I created my `createCart()` function using GraphQL, which sends a mutation to create a new cart with that predefined variant.

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

At this stage, my function logs two things to the console:
   1. This logs the cart along with the message “Cart Created”.

      ```js
      console.log("Cart Created", cart);
      ```

   2. This prints the Cart ID, which will be useful later.
      ```js
      console.log("Cart ID:", cart.id);
      ```

Finally, I call the function:

```js
await createCart();
```

Here’s how the complete code would look:

{% highlight js linenos %}

const shopDomain = "your-shop-name.myshopify.com";
const storefrontToken = "your-storefront-access-token";
const apiUrl = `https://${shopDomain}/api/2024-10/graphql.json`;

const variantId = "gid://shopify/ProductVariant/your-variant-id";

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

And this is how it looks in my console:

<div style="text-align: center;">
  <img width="800" alt="Shopify cart preview" src="https://github.com/user-attachments/assets/42088928-e9c1-4c5e-9020-96b309bf383b" />
</div>

Step 2: Adding Items to the Cart

Before adding a product to the cart, I first check whether the variant exists and if it’s available for sale.
To do this, I created a function called `checkVariantStock()`, which performs a GraphQL query to fetch information about a specific product variant.

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

We create a function to add an item to the cart.

The function `addItemToCart()` performs a GraphQL mutation called `cartLinesAdd`, which allows us to add one or more product variants to an existing cart.
This mutation requires the cart ID, the product variant ID, and the quantity to add.

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

Now we’re not just going to call the function directly we’ll do something a little different.
When the DOM finishes loading, we’ll create an empty cart, check the stock availability of the product we want to add by calling our `checkVariantStock(variantId)` function, and finally, we’ll add the product to the cart.

The code would look like this:

```js
document.addEventListener("DOMContentLoaded", async () => {
  const cartId = await createCart(); 
  await checkVariantStock(variantId)
  await addItemToCart(cartId, variantId, 1);
});
```

This is how Step 2 would look:

```js
const shopDomain = "your-shop-name.myshopify.com";
const storefrontToken = "your-storefront-access-token";
const apiUrl = `https://${shopDomain}/api/2024-10/graphql.json`;

const variantId = "gid://shopify/ProductVariant/your-variant-id";

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

Step 3: In this step, we want the cart button to display a modal showing the item that was added.

At this point, we need to actually display the cart. Since `createCart()` only creates an empty cart, we have to fetch the current state of that cart. To do this, we’ll use a query that tells Shopify: “Show me the contents of the cart with this ID.”

Here’s how I implemented it:

```js
async function getCart(cartId) {
    const query = `
      query GetCart($cartId: ID!) {
        cart(id: $cartId) {
          id
          createdAt
          updatedAt
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
                    price {
                      amount
                      currencyCode
                    }
                    product {
                      title
                    }
                  }
                }
              }
            }
          }
          cost {
            totalAmount {
              amount
              currencyCode
            }
          }
        }
      }
    `;
    
  const variables = { cartId };
    
  const result = await shopifyRequest(query, variables);
  
  if (result.errors) {
    console.error("GraphQL errors:", result.errors);
    throw new Error("Failed to fetch cart.");
  }
  
  console.log("Get Cart", result.data.cart)
  return result.data.cart;
}
```

As mentioned earlier, we need to display the cart. My page is built on Webflow, so I created the modal by adding divs and assigning them an ID through the Settings panel. To display the modal, I created a function called `showCart()`.

Here’s a step by step explanation of what I did with this function:
   1. First, I retrieved the current state of my cart using the function I created earlier.
   I also selected the cart modal, which will contain all the information, the cart items container where the items will be added, and an empty message element.
   It’s important to make sure all of these elements are properly defined either in your HTML or in the Webflow settings if you’re using it.

      ```js
      async function showCart(cartId) {
        const cart = await getCart(cartId);
        const cartModal = document.getElementById('cart-modal');
        const cartItemsContainer = document.getElementById('cart-items');
        const emptyCartMessage = document.getElementById('empty-cart-message');
      ```

   2. Next, we clear the cart container to make sure it’s empty before rendering the updated items. This doesn’t remove products from the Shopify cart itself it only clears the cart’s visual content on the page so we can display the new data cleanly.
      ```js
        cartItemsContainer.innerHTML = '';
      ```

   3. If the cart is empty, we display our “No items found” message; if there are items, this message is hidden.
      ```js
        if (!cart || !cart.lines || cart.lines.edges.length === 0) {
          emptyCartMessage.style.display = 'block';
          return;
        } else {
          emptyCartMessage.style.display = 'none';
        }
      ``` 

   4. Here, I used a forEach loop to iterate through all the items in the cart and display them. For each item, I extracted details like the product title, variant title, quantity, price, and currency.
      ```js
        cart.lines.edges.forEach(edge => {
          const item = edge.node;
          const variant = item.merchandise;

          const productTitle = variant.product?.title || 'Producto sin nombre';
          const variantTitle = variant.title || '';
          const quantity = item.quantity;
          const price = parseFloat(variant.price.amount);
          const currency = variant.price.currencyCode;
          const total = (price * quantity).toFixed(2);

          const listItem = document.createElement('div');
          listItem.classList.add('cart-item');

          listItem.textContent = `${productTitle} - ${variantTitle} | Cantidad: ${quantity} | Precio unitario: $${price} ${currency} | Total: $${total}`;

          cartItemsContainer.appendChild(listItem);
        });
      ```

   5. Then, I calculated the total price, created a new div element for the item, and added it to the cart container.
      ```js
        const totalAmount = cart.cost.totalAmount.amount;
        const currency = cart.cost.totalAmount.currencyCode;

        const totalDiv = document.createElement('div');
        totalDiv.classList.add('cart-total');
        totalDiv.textContent = `Total del carrito: $${totalAmount} ${currency}`;
        cartItemsContainer.appendChild(totalDiv);
      ```

   6. Finally, we display the cart. If the modal element exists, we set its display style to "flex" so it becomes visible. Otherwise, we log a warning message in case the element wasn’t found in the DOM.
      ```js
        if (cartModal) {
          cartModal.style.display = "flex";
        } else {
          console.warn("cartList no encontrado en el DOM");
        }
      }
      ```

Now, in addition to what we had before, we add an addEventListener that triggers when the cart button is clicked.

```js
document.addEventListener("DOMContentLoaded", async () => {
  const cartBtn = document.getElementById('cart-button');
  const cartId = await createCart();
  await checkVariantStock(variantId)
  await addItemToCart(cartId, variantId, 1);
  cartBtn.addEventListener('click', async () => {
    showCart(cartId);
    })
});
```

And this is how Step 3 looks completed, it’s still a bit rough, but we’ll keep improving it step by step.

<div style="text-align: center;">
  <img width="800" alt="Screenshot 2025-09-27 at 9 28 36 AM" src="https://github.com/user-attachments/assets/14ca3a4d-2abc-49a3-9254-044c0896f421" />
</div>

Step 4: In this step, we’re going to add products to the cart using the “Add to Cart” button.

To do this, we’ll add another addEventListener that triggers when the button is clicked, and we’ll place our function inside it.

```js
document.addEventListener("DOMContentLoaded", async () => {
  const cartBtn = document.getElementById('cart-button');
  const addToCartBtn = document.getElementById('add-to-cart-button');

  const cartId = await createCart(); 
  await checkVariantStock(variantId)

  addToCartBtn.addEventListener('click', async () => {
    await addItemToCart(cartId, variantId, 1);
    console.log("Producto añadido");
  });

  cartBtn.addEventListener('click', async () => {
    showCart(cartId);
  });
});
```

I added a console.log to see that the item was successfully added, and I also gave it a title from Webflow to add a bit more style.

<div style="text-align: center;">
  <img width="800" alt="Screenshot 2025-09-27 at 2 44 10 PM" src="https://github.com/user-attachments/assets/945946f5-0a6e-4be8-a8b3-384c7106d3a0" />
</div>

Step 5: In this step, we’ll add the product to the cart based on the variant associated with the button.

First, we need to remove this line:

```js
const variantId = `gid://shopify/ProductVariant/46880650297570`;
```

We then add this forEach loop that goes through all buttons with a data-variant-id attribute. When a button is clicked, it retrieves the variant ID from that button and adds the product to the cart:

```js
const addToCartButtons = document.querySelectorAll('[data-variant-id]');

addToCartButtons.forEach(button => {
  button.addEventListener('click', async () => {
    const variantId = button.getAttribute('data-variant-id');
    if (!variantId) {
      console.error("Variant ID no encontrado en el botón");
      return;
    }

    try {
      await addItemToCart(cartId, variantId, 1);
      console.log("Producto añadido al carrito");
    } catch (e) {
      console.error("Error al añadir producto:", e);
    }
  });
});
```

We also replace addToCartBtn with addToCartButtons to handle all buttons instead of just one.

Step 6: We add a toast notification to show a success message when a product is successfully added to the cart.

First, I created my `showToast()` function.

```js
function showToast(message, type = 'error') {
  const existingToast = document.getElementById('toast-notification');
  if (existingToast) {
    existingToast.remove();
  }

  const toast = document.createElement('div');
  toast.id = 'toast-notification';
  toast.textContent = message;

  toast.style.position = 'fixed';
  toast.style.top = '30px';
  toast.style.right = '30px';
  toast.style.padding = '14px 20px';
  toast.style.backgroundColor = type === 'error' ? '#ff4d4f' : '#4caf50';
  toast.style.color = '#fff';
  toast.style.borderRadius = '8px';
  toast.style.boxShadow = '0 4px 8px rgba(0,0,0,0.2)';
  toast.style.fontSize = '16px';
  toast.style.zIndex = '9999';
  toast.style.opacity = '0';
  toast.style.transition = 'opacity 0.4s ease, transform 0.4s ease';
  toast.style.transform = 'translateY(20px)';

  document.body.appendChild(toast);

  requestAnimationFrame(() => {
    toast.style.opacity = '1';
    toast.style.transform = 'translateY(0)';
  });

  setTimeout(() => {
    toast.style.opacity = '0';
    toast.style.transform = 'translateY(-20px)';
    setTimeout(() => {
      toast.remove();
    }, 400);
  }, 3000);
}
```

Then, I modified my forEach loop slightly. First, it checks if the product is in stock. If not, it shows a toast message saying “No stock available”. If the product is successfully added, it displays a toast saying “Product added successfully.”

```js
addToCartButtons.forEach(button => {
  button.addEventListener('click', async () => {
    const variantId = button.getAttribute('data-variant-id');
    if (!variantId) {
      console.error("Variant ID no encontrado en el botón");
      return;
    }

    try {
      const stock = await checkVariantStock(variantId);

      if (!stock || stock.quantityAvailable <= 0 || !stock.availableForSale) {
        showToast("No stock available");
        return;
      }

      await addItemToCart(cartId, variantId, 1);
      showToast("Product added successfully", "success");
    } catch (e) {
      console.error("Error al añadir producto:", e);
      showToast("Error checking stock.");
    }
  });
});
```

<div style="text-align: center;">
    <div class="col-sm mt-3 mt-md-0">
      <img width="800" alt="Screenshot 2025-09-27 at 4 12 21 PM" src="https://github.com/user-attachments/assets/9a6b7720-c626-40fc-b76f-a93550494ba2" />
    </div>
    <div class="col-sm mt-3 mt-md-0">
      <img width="800" alt="Screenshot 2025-09-27 at 3 54 47 PM" src="https://github.com/user-attachments/assets/f076fa65-50bb-4dcd-946f-e00faf5f14cc" />
    </div>
</div>

Here’s how our code looks so far:

{% highlight js linenos %}
const shopDomain = "your-shop-name.myshopify.com";
const storefrontToken = "your-storefront-access-token";
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
  console.log("Cart ID:", cart.id);
  
  return cart.id
}

async function getCart(cartId) {
    const query = `
      query GetCart($cartId: ID!) {
        cart(id: $cartId) {
          id
          createdAt
          updatedAt
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
                    price {
                      amount
                      currencyCode
                    }
                    product {
                      title
                      featuredImage {
                        url
                        altText
                      }
                    }
                  }
                }
              }
            }
          }
          cost {
            totalAmount {
              amount
              currencyCode
            }
          }
        }
      }
    `;
    
  const variables = { cartId };
    
  const result = await shopifyRequest(query, variables);
  
  if (result.errors) {
    console.error("GraphQL errors:", result.errors);
    throw new Error("Failed to fetch cart.");
  }
  
  console.log("Get Cart", result.data.cart)
  return result.data.cart;
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

function showToast(message, type = 'error') {
  const existingToast = document.getElementById('toast-notification');
  if (existingToast) {
    existingToast.remove();
  }

  const toast = document.createElement('div');
  toast.id = 'toast-notification';
  toast.textContent = message;

  toast.style.position = 'fixed';
  toast.style.top = '30px';
  toast.style.right = '30px';
  toast.style.padding = '14px 20px';
  toast.style.backgroundColor = type === 'error' ? '#ff4d4f' : '#4caf50';
  toast.style.color = '#fff';
  toast.style.borderRadius = '8px';
  toast.style.boxShadow = '0 4px 8px rgba(0,0,0,0.2)';
  toast.style.fontSize = '16px';
  toast.style.zIndex = '9999';
  toast.style.opacity = '0';
  toast.style.transition = 'opacity 0.4s ease, transform 0.4s ease';
  toast.style.transform = 'translateY(20px)';

  document.body.appendChild(toast);

  requestAnimationFrame(() => {
    toast.style.opacity = '1';
    toast.style.transform = 'translateY(0)';
  });

  setTimeout(() => {
    toast.style.opacity = '0';
    toast.style.transform = 'translateY(-20px)';
    setTimeout(() => {
      toast.remove();
    }, 400);
  }, 3000);
}

async function showCart(cartId) {
  const cart = await getCart(cartId);
  const cartModal = document.getElementById('cart-modal');
  const cartItemsContainer = document.getElementById('cart-items');
  const emptyCartMessage = document.getElementById('empty-cart-message');

  cartItemsContainer.innerHTML = '';

  if (!cart || !cart.lines || cart.lines.edges.length === 0) {
    emptyCartMessage.style.display = 'block';
    return;
  } else {
    emptyCartMessage.style.display = 'none';
  }

  cart.lines.edges.forEach(edge => {
    const item = edge.node;
    const variant = item.merchandise;
    const product = variant.product;

    const productTitle = product?.title || 'Producto sin nombre';
    const variantTitle = variant.title || '';
    const quantity = item.quantity;
    const price = parseFloat(variant.price.amount);
    const currency = variant.price.currencyCode;
    const total = (price * quantity).toFixed(2);
    const imageUrl = product?.featuredImage?.url || '';
    const altText = product?.featuredImage?.altText || productTitle;

    const listItem = document.createElement('div');
    listItem.style.display = 'flex';
    listItem.style.gap = '15px';
    listItem.style.padding = '12px';
    listItem.style.borderBottom = '1px solid #eee';
    listItem.style.alignItems = 'center';

    if (imageUrl) {
      const imageDiv = document.createElement('div');
      const img = document.createElement('img');
      img.src = imageUrl;
      img.alt = altText;
      img.style.width = '80px';
      img.style.height = '80px';
      img.style.objectFit = 'cover';
      img.style.borderRadius = '6px';
      imageDiv.appendChild(img);
      listItem.appendChild(imageDiv);
    }

    const detailsDiv = document.createElement('div');
    detailsDiv.style.display = 'flex';
    detailsDiv.style.flexDirection = 'column';
    detailsDiv.style.gap = '4px';

    const titleEl = document.createElement('div');
    titleEl.textContent = productTitle;
    titleEl.style.fontWeight = 'bold';
    titleEl.style.fontSize = '16px';

    const variantEl = document.createElement('div');
    variantEl.textContent = variantTitle;
    variantEl.style.fontSize = '14px';
    variantEl.style.color = '#555';

    const quantityEl = document.createElement('div');
    quantityEl.textContent = `Cantidad: ${quantity}`;
    quantityEl.style.fontSize = '14px';

    const priceEl = document.createElement('div');
    priceEl.textContent = `Precio unitario: $${price} ${currency}`;
    priceEl.style.fontSize = '14px';

    const totalEl = document.createElement('div');
    totalEl.textContent = `Total: $${total} ${currency}`;
    totalEl.style.fontSize = '14px';

    detailsDiv.appendChild(titleEl);
    detailsDiv.appendChild(variantEl);
    detailsDiv.appendChild(quantityEl);
    detailsDiv.appendChild(priceEl);
    detailsDiv.appendChild(totalEl);

    listItem.appendChild(detailsDiv);
    cartItemsContainer.appendChild(listItem);
  });

  const totalAmount = cart.cost.totalAmount.amount;
  const currency = cart.cost.totalAmount.currencyCode;

  const totalDiv = document.createElement('div');
  totalDiv.textContent = `Total del carrito: $${totalAmount} ${currency}`;
  totalDiv.style.fontWeight = 'bold';
  totalDiv.style.fontSize = '18px';
  totalDiv.style.marginTop = '24px';
  totalDiv.style.paddingTop = '12px';
  totalDiv.style.borderTop = '1px solid #ddd';
  totalDiv.style.textAlign = 'center';

  cartItemsContainer.appendChild(totalDiv);

  if (cartModal) {
    cartModal.style.display = "flex";
  } else {
    console.warn("cart-modal no encontrado en el DOM");
  }
}


document.addEventListener("DOMContentLoaded", async () => {
  const cartBtn = document.getElementById('cart-button');
  const addToCartButtons = document.querySelectorAll('[data-variant-id]');

  const cartId = await createCart(); 

  addToCartButtons.forEach(button => {
    button.addEventListener('click', async () => {
      const variantId = button.getAttribute('data-variant-id');
      if (!variantId) {
        console.error("Variant ID no encontrado en el botón");
        return;
      }

      try {
        const stock = await checkVariantStock(variantId);

        if (!stock || stock.quantityAvailable <= 0 || !stock.availableForSale) {
          showToast("No stock available");
          return;
        }

        await addItemToCart(cartId, variantId, 1);
        showToast("Product added successfully", "success");
      } catch (e) {
        console.error("Error al añadir producto:", e);
        showToast("Error checking stock.");
      }
    });
  });


  cartBtn.addEventListener('click', async () => {
    showCart(cartId);
  });
});

{% endhighlight %}

I also added some styling to my `showCart()` function.

<div style="text-align: center;">
  <img width="800" alt="Screenshot 2025-09-27 at 5 16 02 PM" src="https://github.com/user-attachments/assets/7345d406-02ed-4652-bdd6-3c4c942f7610" />
</div>

For the final part, I had to split the code into two sections because of Webflow. One part was placed in a code embed nested inside the cart button, and the other part was added in a recursive code embed within a collection for the “Add to Cart” buttons in the catalog on the page.

This is the final code for the cart button. Here, we create an empty cart and define it globally using window so it can be accessed and modified by the other code section. We also include the logic for the checkout button and the counter displayed on the cart button.

{% highlight js linenos %}

async function shopifyRequest(query, variables = {}) {
  const response = await fetch("https://${your-shop-name.myshopify.com}/api/2024-10/graphql.json", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-Shopify-Storefront-Access-Token": "your-storefront-access-token",
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
          checkoutUrl
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
  const checkoutUrl = cart.checkoutUrl;
  
  return {id: cart.id, checkoutUrl : checkoutUrl}
}

async function getCart(cartId) {
    const query = `
      query GetCart($cartId: ID!) {
        cart(id: $cartId) {
          id
          createdAt
          updatedAt
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
                    price {
                      amount
                      currencyCode
                    }
                    product {
                      title
                      featuredImage {
                        url
                        altText
                      }
                    }
                  }
                }
              }
            }
          }
          cost {
            totalAmount {
              amount
              currencyCode
            }
          }
        }
      }
    `;
    
  const variables = { cartId };
    
  const result = await shopifyRequest(query, variables);
  
  if (result.errors) {
    console.error("GraphQL errors:", result.errors);
    throw new Error("Failed to fetch cart.");
  }
  
  console.log("Get Cart", result.data.cart)
  return result.data.cart;
}

async function showCart(cartId) {
  const cart = await getCart(cartId);
  const cartModal = document.getElementById('cart-modal');
  
  const cartItemsContainer = document.getElementById('cart-items');
  cartItemsContainer.style.display = 'flex';
  cartItemsContainer.style.flexDirection = 'column';
  cartItemsContainer.style.maxHeight = '400px';
  cartItemsContainer.style.overflowY = 'auto';
  cartItemsContainer.style.overflowX = 'hidden';
  
  const emptyCartMessage = document.getElementById('empty-cart-message');

  cartItemsContainer.innerHTML = '';
  
  if (!cart || !cart.lines || cart.lines.edges.length === 0) {
    cartModal.style.display = "flex";

    const cartTotalDiv = document.getElementById('cart-total');
    if (cartTotalDiv) {
      cartTotalDiv.textContent = '';
      cartTotalDiv.style.display = 'none';
    }

    emptyCartMessage.style.display = 'block';
    return;
  } else {
    emptyCartMessage.style.display = 'none';
  }

  cart.lines.edges.forEach(edge => {
    const item = edge.node;
    const variant = item.merchandise;
    const product = variant.product;

    const productTitle = product?.title || 'Producto sin nombre';
    const variantTitle = variant.title || '';
    const quantity = item.quantity;
    const price = parseFloat(variant.price.amount);
    const currency = variant.price.currencyCode;
    const total = (price * quantity).toFixed(2);
    const imageUrl = product?.featuredImage?.url || '';
    const altText = product?.featuredImage?.altText || productTitle;

    const listItem = document.createElement('div');
    listItem.style.display = 'flex';
    listItem.style.gap = '15px';
    listItem.style.padding = '12px';
    listItem.style.borderBottom = '1px solid #eee';
    listItem.style.alignItems = 'center';

    if (imageUrl) {
      const imageDiv = document.createElement('div');
      const img = document.createElement('img');
      img.src = imageUrl;
      img.alt = altText;
      img.style.width = '80px';
      img.style.height = '80px';
      img.style.objectFit = 'cover';
      img.style.borderRadius = '6px';
      imageDiv.appendChild(img);
      listItem.appendChild(imageDiv);
    }

    const detailsDiv = document.createElement('div');
    detailsDiv.style.display = 'flex';
    detailsDiv.style.flexDirection = 'column';
    detailsDiv.style.gap = '4px';

    const titleEl = document.createElement('div');
    titleEl.textContent = productTitle;
    titleEl.style.fontWeight = 'bold';
    titleEl.style.fontSize = '16px';

    const variantEl = document.createElement('div');
    variantEl.textContent = variantTitle;
    variantEl.style.fontSize = '14px';
    variantEl.style.color = '#555';

    const quantityWrapper = document.createElement('div');
    quantityWrapper.style.display = 'flex';
    quantityWrapper.style.alignItems = 'center';
    quantityWrapper.style.gap = '8px';

    const minusBtn = document.createElement('button');
    minusBtn.textContent = '-';
    minusBtn.style.width = '24px';
    minusBtn.style.height = '24px';
    minusBtn.style.cursor = 'pointer';
    minusBtn.style.fontSize = '16px';

    const quantityDisplay = document.createElement('span');
    quantityDisplay.textContent = quantity;
    quantityDisplay.style.fontSize = '14px';
    quantityDisplay.style.minWidth = '20px';
    quantityDisplay.style.textAlign = 'center';

    const plusBtn = document.createElement('button');
    plusBtn.textContent = '+';
    plusBtn.style.width = '24px';
    plusBtn.style.height = '24px';
    plusBtn.style.cursor = 'pointer';
    plusBtn.style.fontSize = '16px';

    quantityWrapper.appendChild(minusBtn);
    quantityWrapper.appendChild(quantityDisplay);
    quantityWrapper.appendChild(plusBtn);

    minusBtn.addEventListener('click', async () => {
      const newQuantity = item.quantity - 1;

      try {
        if (newQuantity === 0) {
          const updatedCart = await removeCartLine(cart.id, item.id);
          updateCartCounter(updatedCart.totalQuantity);
          showCart(cart.id);
        } else {
          const updatedCart = await updateCartLineQuantity(cart.id, item.id, newQuantity);
          updateCartCounter(updatedCart.totalQuantity);
          showCart(cart.id);
        }
      } catch (err) {
        console.error(err);
      }
    });

    plusBtn.addEventListener('click', async () => {
      try {
        const variantStock = await checkVariantStock(variant.id);
        const available = variantStock?.quantityAvailable || 0;
        const newQuantity = item.quantity + 1;

        if (newQuantity > available) {
          alert(`Only ${available} unit${available === 1 ? '' : 's'} available in stock`);
          return;
        }

        const updatedCart = await updateCartLineQuantity(cart.id, item.id, newQuantity);
        updateCartCounter(updatedCart.totalQuantity);
        showCart(cart.id); 
      } catch (err) {
        console.error(err);
      }
    });
    
    const priceEl = document.createElement('div');
    priceEl.textContent = `Precio unitario: $${price} ${currency}`;
    priceEl.style.fontSize = '14px';

    detailsDiv.appendChild(titleEl);
    detailsDiv.appendChild(variantEl);
    detailsDiv.appendChild(priceEl);
    detailsDiv.appendChild(quantityWrapper);

    listItem.appendChild(detailsDiv);
    cartItemsContainer.appendChild(listItem);
  });

  const totalAmount = cart.cost.totalAmount.amount;
  const currency = cart.cost.totalAmount.currencyCode;
  
  const cartTotalDiv = document.getElementById('cart-total');

  if (cartTotalDiv) {
    cartTotalDiv.textContent = `Total del carrito: $${totalAmount} ${currency}`;
    cartTotalDiv.style.display = 'block';
    cartTotalDiv.style.fontWeight = 'bold';
    cartTotalDiv.style.fontSize = '18px';
    cartTotalDiv.style.marginTop = '24px';
    cartTotalDiv.style.paddingTop = '12px';
    cartTotalDiv.style.borderTop = '1px solid #ddd';
    cartTotalDiv.style.textAlign = 'center';
  } else {
    console.warn("No se encontró el div con id 'cart-total'");
  }

  if (cartModal) {
    cartModal.style.display = "flex";
  } else {
    console.warn("cart-modal no encontrado en el DOM");
  }
}

function updateCartCounter(count) {
  const counterEl = document.getElementById('cart-counter');
  counterEl.textContent = count;
}

async function updateCartLineQuantity(cartId, lineId, newQuantity) {
  const mutation = `
    mutation CartLinesUpdate($cartId: ID!, $lines: [CartLineUpdateInput!]!) {
      cartLinesUpdate(cartId: $cartId, lines: $lines) {
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
        id: lineId,
        quantity: newQuantity
      }
    ]
  };

  const result = await shopifyRequest(mutation, variables);

  if (result.errors || result.data.cartLinesUpdate.userErrors.length > 0) {
    console.error("Error al actualizar línea:", result.errors || result.data.cartLinesUpdate.userErrors);
    throw new Error("Error al actualizar cantidad en carrito");
  }

  return result.data.cartLinesUpdate.cart;
}

async function removeCartLine(cartId, lineId) {
  const mutation = `
    mutation CartLinesRemove($cartId: ID!, $lineIds: [ID!]!) {
      cartLinesRemove(cartId: $cartId, lineIds: $lineIds) {
        cart {
          id
          totalQuantity
          lines(first: 10) {
            edges {
              node {
                id
                quantity
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
    lineIds: [lineId]
  };

  const result = await shopifyRequest(mutation, variables);

  if (result.errors || result.data.cartLinesRemove.userErrors.length > 0) {
    console.error("Error al eliminar línea:", result.errors || result.data.cartLinesRemove.userErrors);
    throw new Error("Error al eliminar producto del carrito");
  }

  return result.data.cartLinesRemove.cart;
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

  const variables = { id: variantId };

  const result = await shopifyRequest(query, variables);

  if (result.errors) {
    console.error("Error al obtener stock:", result.errors);
    return null;
  }

  return result.data.node;
}


document.addEventListener("DOMContentLoaded", async () => {
  const cartInfo = await createCart();
  const cartId = cartInfo.id
  const checkoutUrl = cartInfo.checkoutUrl
  window.cartId = cartId;
  console.log("cartId", cartId);
  updateCartCounter(0)
  
  const cartBtn = document.getElementById('cart-button');
  const checkoutBtn = document.getElementById('checkout-button');

  cartBtn.addEventListener('click', async () => {
    showCart(cartId);
  });
  
  checkoutBtn.addEventListener('click', () => {
    window.location.href = checkoutUrl; 
  });
  
});

{% endhighlight %}

And in the code embed for the “Add to Cart” button, all the logic for adding a product to the cart is included. I also added a setTimeout() to ensure that the cart is created first before we add items to it.

{% highlight js linenos %}

<div  
class="buy-button"
id="buy-button-{{ page.variant_id }}"
><p>Buy from {{ page.roaster_name }}</p>
</div>

async function shopifyRequest(query, variables = {}) {
  const response = await fetch("https://bu1fib-rq.myshopify.com/api/2024-10/graphql.json", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-Shopify-Storefront-Access-Token": "83e93e5462232411b85e227dad586a69",
    },
    body: JSON.stringify({ query, variables }),
  });

  const data = await response.json();
  return data;
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
        merchandiseId: `gid://shopify/ProductVariant/${variantId}`,
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
  return updatedCart.totalQuantity;
}

async function getTotalCount(cartId) {
    const query = `
      query GetCart($cartId: ID!) {
        cart(id: $cartId) {
          id
          createdAt
          updatedAt
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
                    price {
                      amount
                      currencyCode
                    }
                    product {
                      title
                      featuredImage {
                        url
                        altText
                      }
                    }
                  }
                }
              }
            }
          }
          cost {
            totalAmount {
              amount
              currencyCode
            }
          }
        }
      }
    `;
    
  const variables = { cartId };
    
  const result = await shopifyRequest(query, variables);
  
  if (result.errors) {
    console.error("GraphQL errors:", result.errors);
    throw new Error("Failed to fetch cart.");
  }
  
  console.log("Get Cart", result.data.cart)
  return result.data.cart.totalQuantity;
}

function showToast(message, type = 'error') {
  const existingToast = document.getElementById('toast-notification');
  if (existingToast) {
    existingToast.remove();
  }

  const toast = document.createElement('div');
  toast.id = 'toast-notification';
  toast.textContent = message;

  toast.style.position = 'fixed';
  toast.style.top = '30px';
  toast.style.right = '30px';
  toast.style.padding = '14px 20px';
  toast.style.backgroundColor = type === 'error' ? '#ff4d4f' : '#4caf50';
  toast.style.color = '#fff';
  toast.style.borderRadius = '8px';
  toast.style.boxShadow = '0 4px 8px rgba(0,0,0,0.2)';
  toast.style.fontSize = '16px';
  toast.style.zIndex = '9999';
  toast.style.opacity = '0';
  toast.style.transition = 'opacity 0.4s ease, transform 0.4s ease';
  toast.style.transform = 'translateY(20px)';

  document.body.appendChild(toast);

  requestAnimationFrame(() => {
    toast.style.opacity = '1';
    toast.style.transform = 'translateY(0)';
  });

  setTimeout(() => {
    toast.style.opacity = '0';
    toast.style.transform = 'translateY(-20px)';
    setTimeout(() => {
      toast.remove();
    }, 400);
  }, 3000);
}

function updateCartCounter(count) {
  const counterEl = document.getElementById('cart-counter');
  counterEl.textContent = count;
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
  setTimeout( async () => {
    updateCartCounter(0)
    console.log("Hey this is cart id", window.cartId);

    const buyButton = document.getElementById(`buy-button-{{ page.variant_id }}`);

    buyButton.addEventListener('click', async () => {
      const stock = await checkVariantStock(`gid://shopify/ProductVariant/{{ page.variant_id }}`);
        if (!stock || stock.quantityAvailable === 0 || stock.availableForSale === false) {
          showToast("Not enough stock for product");
          return;
        } 
      showToast("Product added successfully", "success");
      const totalCount = await addItemToCart(window.cartId, `{{ page.variant_id }}`, 1);
      updateCartCounter(totalCount);

    });

  }, 2000); 
});

{% endhighlight %}

And after a few design tweaks, this is the final result.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/1.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

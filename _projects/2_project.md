---
layout: page
title: Unblended
description: Coffee page
img: assets/img/3.jpg
importance: 2
category: work
giscus_comments: true
---

This is a sales page for a coffee company called <a href="">Unblended</a>. The implementation I worked on includes adding a shopping cart that allows users to add, remove, and view the total value of the items. The cart also has a checkout button that redirects you to the page where you can complete your payment. 

It works like this:

On the page, there’s a catalog displaying all the products. Each product has an “Add to Cart” button, and when I click it, the item is added to the database (I implemented this by calling the Shopify API). The cart button counter is also updated automatically.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

When I click the cart button, a cart modal opens showing the items that have been added. If there are no items, a message saying “no items found” is displayed. When there are items, the modal shows the product images along with plus and minus buttons to add or remove items from the cart. It also displays the total amount to pay and, finally, a checkout button.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/1.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

When you click the checkout button in the cart, it redirects you to the payment page. You fill out the required payment information, and the purchase is complete! This is the flow of how the functionality I implemented works.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

You can see the full page <a href="">here</a>

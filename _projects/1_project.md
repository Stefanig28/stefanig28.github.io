---
layout: page
title: Flety
description: Load Board Page
img: assets/img/12.jpg
importance: 1
category: work
related_publications: false
---

The Load Board is a platform that connects transportation companies with freight owners, enabling them to post, search, and bid on available loads.

On the "Mis Fletes" page, a table displays load information such as origin, destination, delivery and pickup dates, vehicle type, status, and a detail section. Clicking on “ver detalle” redirects the user to the full load view, where all available information is shown.

At the top of the page, above the table, there are some statistics. Below them, a filter button allows users to refine the loads shown in the table. Lastly, there are “Crear flete” buttons that lead to a form where new freights can be created. 

<div class="row mt-5 mb-5">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/12.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The following view shows how the filters and detailed load information are displayed on the page.

<div class="row mt-5 mb-5">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/6.jpg" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/10.jpg" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

When clicking on “Crear flete” the user is redirected to a page containing a form to create a new load. The form includes dropdown menus and other input fields. We also implemented maps with autocomplete functionality using the Google Places API. After submitting all the required information, a confirmation modal appears, indicating that the load has been successfully created.

<div class="row mt-5 mb-5">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/4.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/7.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

This is the final result of the maps implemented with the Google Places API.

<div class="row mt-5 mb-5">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/8.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

We can also see two very important pages: “Mis Aliados y Conductores”, which shows information about the drivers who have transported our loads at some point, and “Mis Lugares”, which displays information about the places from or to which we’ve transported loads.

<div class="row mt-5 mb-5">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/9.jpg" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/13.jpg" class="img-fluid rounded z-depth-1" %}
    </div>
</div>




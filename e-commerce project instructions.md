# E-Commmerce Lab

# Getting Started

## Set up libraries

1. Create or reuse a Django equipped virtual environment
2. Use pip to install Pillow version 8.3.2 or later for image handling

## Set up project

3. Start a new Django project called myshop
4. Start a new Django app called shop


# Create the models and views for your product catalogue
## Create models for myshop

1. Create Category model

   ```python
   from django.db import models
   from django.urls import reverse


   class Category(models.Model):
       name = models.CharField(max_length=200, db_index=True)
       slug = models.SlugField(max_length=200, unique=True)

       class Meta:
           ordering = ('name',)
           verbose_name = 'category'
           verbose_name_plural = 'categories'

       def __str__(self):
           #TODO

       def get_absolute_url(self):
           return reverse('shop:product_list_by_category',
                         args=[self.slug])

   ```

2. Create Product model with the following fields

   - category: A ForeignKey to the Category model. This is a one-to-many relationship: a product belongs to one category and a category contains multiple products.
   - name: The name of the product.
   - slug: The slug for this product to build beautiful URLs.
   - image: An optional product image.
   - description: An optional description of the product.
   - price: This field uses Python's decimal.Decimal type to store a fixedprecision decimal number. The maximum number of digits (including
     the decimal places) is set using the max_digits attribute and decimal
     places with the decimal_places attribute.
   - available: A Boolean value that indicates whether the product is available
     or not. It will be used to enable/disable the product in the catalog.
   - created: This field stores when the object was created.
   - updated: This field stores when the object was last updated.

    ```python
        class Product(#TODO):
            category = #TODO
            slug = models.SlugField(#TODO)
            image = models.ImageField(upload_to='products/%Y/%m/%d', blank=True)
            description = models.TextField(blank=True)
            updated = models.DateTimeField(auto_now=True)
            #TODO

            class Meta:
                ordering = ('name',)
                index_together = (('id', 'slug'),)

            def __str__(self):
                #TODO

            def get_absolute_url(self):
                return reverse('shop:product_detail',
                            args=[self.id, self.slug])
    ```

3. Make and apply migrations

## Set up the Admin

1. Create the CategoryAdmin class
   - List the name and slug fields
   - Prepopulate the slug from the name
   ```python
   class CategoryAdmin(admin.ModelAdmin):
       list_display = ['name', 'slug']
       prepopulated_fields = {'slug': ('name',)}
   ```
2. Create the ProductAdmin class
 - List everything except the image and description
 - Filter for the appropriate fields
 - Make the **price** and **available** editable (Hint: **list_editable**)
 - Prepopulate the slug
3. Register the classes
4. Don't forget to set up a superuser

## Build Views for the models in the shop application

1. Create the product_list view
   This view should display all available products or, if passed a category_slug, then it should display the products filtered by it
2. Create the product_detail view
   This view should display a single product by id. Additionally, it should accept and use a
   slug (this helps build SEO-friendly URLs) and return only available products

## Build the urls

Each path should have an internal name that matches the view name unless otherwise specified

1. The default path should resolve to product_list
2. The path category_slug should resolve to product_list. Call this product_list_by_category internally
3. The path id/slug should resolve to product_detail
4. Don't forget to reference the shop urls in the main url file

## Add the get_absolute_url methods to each model

Django has a convention that the method get_absolute_url() will return the URL for a given object

1. Modify the model classes using the following code

   ```python
   from django.urls import reverse
   # ...

   class Category(models.Model):
      # ...
   def get_absolute_url(self):
      return reverse('shop:product_list_by_category',
      args=[self.slug])

   class Product(models.Model):
      # ...
      def get_absolute_url(self):
      return reverse('shop:product_detail',
      args=[self.id, self.slug])
   ```


## Create templates for your views

Be sure your templates are in the product template directory for the shop application indicated below.

1. Create the base.html template in `shop\templates\shop\` as follows:
   ```html
   {% load static %}
    <!DOCTYPE html>
    <html>

    <head>
    <meta charset="utf-8" />
    <title>{% block title %}My shop{% endblock %}</title>
    <link href="{% static " css/base.css" %}" rel="stylesheet">
    </head>

    <body>
    <div id="header">
        <a href="/" class="logo">My shop</a>
    </div>
    <div id="subheader">
        <div class="cart">
        Your cart is empty.
        </div>
    </div>
    <div id="content">
        {% block content %}
        {% endblock %}
    </div>
    </body>

    </html>
   ```
2. Create the list.html product in the  `shop\templates\shop\product` as follows:
   ```html
   #TBD1
    {% block content %}
    <div id="sidebar">
    <h3>Categories</h3>
    <ul>
        <li {% if not category %}class="selected" {% endif %}>
        <a href="{% url " shop:product_list" %}">All</a>
        </li>
        #TBD2
    </ul>
    </div>
    <div id="main" class="product-list">
    <h1>{% if category %}{{ category.name }}{% else %}Products
        {% endif %}</h1>
    {% for product in products %}
    <div class="item">
        <a href="{{ product.get_absolute_url }}">
        <img src="{% if product.image %}{{ product.image.url }}{%
    else %}{% static " img/no_image.png" %}{% endif %}">
        </a>
        <a href="{{ product.get_absolute_url }}">{{ product.name }}</ a>
        <br>
        ${{ product.price }}
    </div>
    {% endfor %}
    </div>
    {% endblock %}
   ```
3. Include the requisite Django instructions to include static files and define a block called title at #TBD1
4. Provide a default title using the category.name if one exists or "Products" if it does not
5. Complete the code for the categories at #TBD2. If the category.slug matches the slug for the current list item, then set the HTML attribute to select that item
6. Copy the resource files provided to the `static` directory of the shop application
7. Add the following code to settings.py
    ```python
    MEDIA_URL = '/media/'
    MEDIA_ROOT = os.path.join(BASE_DIR, 'media/')
    ```
8. Add the following code to the main urls.py file to support media:
   ```python
    from django.conf import settings
    from django.conf.urls.static import static
    urlpatterns = [
        # ...
    ]
    if settings.DEBUG:
        urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
   ```
9. If you want to, go ahead and test the product view now (see below)
10. Add the detail_view.html template in the same folder as list.html as follows:
    ```html
    {% extends "shop/base.html" %}
    {% load static %}
    {% block title %}
    {{ product.name }}
    {% endblock %}
    {% block content %}
    <div class="product-detail">Chapter 7
    <img src="{% if product.image %}{{ product.image.url }}{% else %}
    {% static " img/no_image.png" %}{% endif %}">
    <h1>{{ product.name }}</h1>
    <h2>
        <a href="{{ product.category.get_absolute_url }}">
        {{ product.category }}
        </a>
    </h2>
    <p class="price">${{ product.price }}</p>
    {{ product.description|linebreaks }}
    </div>
    {% endblock %}
    ```
## Test the views
Run your server and add a couple of products using the admin site. Be sure to include some pictures. If you do not include a picture then the page will display a "no image available" image. 

### Your list page should look like
![Product page](lab_images/products1.jpg)

### Your detail page should look like
![Detail page](lab_images/detail1.jpg)
 
# Build a shopping cart
## How to use sessions
The shopping cart will utilize [Django sessions](https://docs.djangoproject.com/en/3.2/topics/http/sessions/), so be sure that the MIDDLEWARE setting of your project contains `django.contrib.sessions.middleware.SessionMiddleware`. Session is accessed like a dictionary as follows:
```python
request.session['foo'] = 'bar'      # write
foo = request.session['foo']        # read 
foo = request.session.get('foo')    # read (gets None if nonexistant) 
del request.session['foo']          # delete
```
By using sessions, we get rapid access to temporary data that will delete itself automatically either when the browser closes (`SESSION_EXPIRE_AT_BROWSER_CLOSE`) or at the end of the period set in `SESSION_COOKIE_AGE`. 
## Shopping cart design
- Create a cart when needed if none exists; otherwise use existing cart
- Store price in cart to insulate against price changes
## Implement the shopping cart
1. Create a shopping cart app called `cart`
2. Store the session id for the shopping cart in `settings.py` in a `CART_SESSION_ID` variable. Pick an appropriate id, i.e. `'cart'`.
3. Create a `cart.py` file in the cart application with a constructor as follows:
```python
from decimal import Decimal
from django.conf import settings
from shop.models import Product

class Cart(object):
    def __init__(self, request):
    """
    Initialize the cart.
    """
    self.session = request.session
    cart = self.session.get(settings.CART_SESSION_ID)
    if not cart:
        # save an empty cart in the session
        cart = self.session[settings.CART_SESSION_ID] = {}
     self.cart = cart
```
4. Create an add method with the the following signature
```python
    def add(self, product, quantity=1, override_quantity=False):
    """
    Add a product to the cart or update its quantity
        product: The product to add
            Use the string representation of the product.id to store it into session
        quantity: The quantity of product to add
        override_quantity: True to replace the existing quantity, false to add to it
    """
    #TBD
```
5. Create a save method as follows:
   ```python
   def save(self):
       # mark the session as "modified" to make sure it gets saved
       self.session.modified = True
   ```
6. Create a remove method
7. To enable iteration over the cart items in client code, you should create an [iterator](https://www.programiz.com/python-programming/iterator) for it as follows:
   ```python
   class Cart(object):
   # ...
   def __iter__(self):
       """
       Iterate over the items in the cart and get the products
       from the database.
       """
       product_ids = self.cart.keys()
       # get the product objects and add them to the cart
       products = Product.objects.filter(id__in=product_ids)
       cart = self.cart.copy()
       for product in products:
           cart[str(product.id)]['product'] = product
       for item in cart.values():
           item['price'] = Decimal(item['price'])
           item['total_price'] = item['price'] * item['quantity']
           yield item
   ```
8. Implement a `__len__` method for the cart that returns the total quantity of items in the cart (sum of quantity for all products)
9. Implement a `get_total_price` method that returns the total price for the cart (sum of price * quantity)
10. Implement a `clear` method which removes the entire shopping cart from the session


## Create views for the shopping cart
1. Create a view to add an item to the cart. The view will receive the **product_id** from the url but the **quantity** and **override_quantity** will only be available as a form field via the POST. The code is provided below:
    ```python
    from django.shortcuts import render, redirect, get_object_or_404
    from django.views.decorators.http import require_POST
    from shop.models import Product
    from .cart import Cart
    from .forms import CartAddProductForm
    @require_POST
    def cart_add(request, product_id):
        cart = Cart(request)
        product = get_object_or_404(Product, id=product_id)
        form = CartAddProductForm(request.POST)
        if form.is_valid():
            cd = form.cleaned_data
            cart.add(product=product, quantity=cd['quantity'], override_quantity=cd['override'])
            return redirect('cart:cart_detail')
    ```
2. Implement a `cart_remove` view to remove an item from the cart
3. Implement a `cart_detail` view to display a cart
4. Implement urls for the cart with the following paths:
    ```python
    ''                  # Display the cart
    'add/product_id'    # Add an item to the cart
    'remove/product_id` # Remove an item from the cart
    ```
## Implement a template for the cart
1. The template code is provided below:
   ```python
    {% extends "shop/base.html" %}
    {% load static %}
    {% load i18n %}

    {% block title %}
    {% trans "Your shopping cart" %}
    {% endblock %}

    {% block content %}
    <h1>{% trans "Your shopping cart" %}</h1>
    <table class="cart">
    <thead>
        <tr>
        <th>{% trans "Image" %}</th>
        <th>{% trans "Product" %}</th>
        <th>{% trans "Quantity" %}</th>
        <th>{% trans "Remove" %}</th>
        <th>{% trans "Unit price" %}</th>
        <th>{% trans "Price" %}</th>
        </tr>
    </thead>
    <tbody>
        {% for item in cart %}
        {% with product=item.product %}
        <tr>
        <td>
            <a href="{{ product.get_absolute_url }}">
            <img src="{% if product.image %}{{ product.image.url }}
                    {% else %}{% static "img/no_image.png" %}{% endif %}">
            </a>
        </td>
        <td>{{ product.name }}</td>
        <td>
            <form action="{% url "cart:cart_add" product.id %}" method="post">
            {{ item.update_quantity_form.quantity }}
            {{ item.update_quantity_form.override }}
            <input type="submit" value="Update">
            {% csrf_token %}
            </form>

        </td>
        <td>
            <form action="{% url "cart:cart_remove" product.id %}" method="post">
            <input type="submit" value="{% trans "Remove" %}">
            {% csrf_token %}
            </form>
        </td>
        <td class="num">${{ item.price }}</td>
        <td class="num">${{ item.total_price }}</td>
        </tr>
        {% endwith %}
        {% endfor %}
        {% if cart.coupon %}
        <tr class="subtotal">
        <td>{% trans "Subtotal" %}</td>
        <td colspan="4"></td>
        <td class="num">${{ cart.get_total_price|floatformat:2 }}</td>
        </tr>
        <tr>
        {% blocktrans with code=cart.coupon.code discount=cart.coupon.discount %}
            <td>"{{ code }}" coupon ({{ discount }}% off)</td>
        {% endblocktrans %}
        <td colspan="4"></td>
        <td class="num neg">
            - ${{ cart.get_discount|floatformat:2 }}
        </td>
        </tr>
    {% endif %}
    <tr class="total">
        <td>{% trans "Total" %}</td>
        <td colspan="4"></td>
        <td class="num">
        ${{ cart.get_total_price_after_discount|floatformat:2 }}
        </td>
    </tr>
    </tbody>
    </table>
    {% if recommended_products %}
    <div class="recommendations cart">
    <h3>{% trans "People who bought this also bought" %}</h3>
    {% for p in recommended_products %}
    <div class="item">
        <a href="{{ p.get_absolute_url }}">
        <img src="{% if p.image %}{{ p.image.url }}{% else %}
        {% static " img/no_image.png" %}{% endif %}">
        </a>
        <p><a href="{{ p.get_absolute_url }}">{{ p.name }}</a></p>
    </div>
    {% endfor %}
    </div>
    {% endif %}
    <p>{% trans "Apply a coupon" %}</p>
    <form action="{% url "coupons:apply" %}" method="post"
    {{ coupon_apply_form }}
    <input type="submit" value="Apply">
    {% csrf_token %}
    </form>
    <p class="text-right">
    <a href="{% url "shop:product_list" %}" class="button light">{% trans "Continue shopping" %}</a>
    <a href="{% url "orders:order_create" %}" class="button">
        {% trans "Checkout" %}
    </a>
    </p>
    {% endblock %}
    ```
## Utilize the cart in the product detail view
1. Add code to create the the product form and pass it to the shop/product/detail template for rendering
2. Add a form to the detail template to render the form
3. Also add a button to add the product to the cart
## Test your shopping cart
1. When viewing product detail, the Add to cart button should be visible along with a quantity selector. Your result should be simlar to this example:
![detail with cart button](lab_images/detail2.jpg)
2. Adding the product to the cart should redirect to the cart view which should be similar to the following:
![cart view](lab_images/cart2.jpg)## Improve the cart to allow the user to change quantities

## Improve the cart to allow the user to change quantities
1. Change the `cart_detail` view to create a `CartAddProductForm` for each item in the cart as follows:
    ```python
    def cart_detail(request):
    cart = Cart(request)
    for item in cart:
        item['update_quantity_form'] = CartAddProductForm(initial={ 
            'quantity': item['quantity'], 
            'override': True 
            }) 
        return render(request, 'cart/detail.html', {'cart': cart})
    ```
2. Replace the line that displays the `item.quantity` in the cart detail template with a form that permits the user to change the quantity or remove it by posting it to the cart_add view
## Test your updated shopping cart
1. The cart should now support changing the product quantity as shown below:
![cart3](lab_images/cart3.jpg)

## Display cart total on all pages
The cart total needs to be displayed on all pages, so we can't use a single view to display it. Instead, we'll add it to the *request* by using a [context processor](https://betterprogramming.pub/django-quick-tips-context-processors-da74f887f1fc). 
1. Create a new file in the **cart** application named **context_processors.py**.
2. Create a function in it named cart that accepts a `request` parameter and returns a dictionary with a single element having a key of `'cart'` and a value of `Cart(request)`. The **Cart** class is the model you created.
3. Add your context processor to the `'context_processors'` list in settings.
4. Replace the text `Your cart is empty` in your base template with the following code:
```html
<div class="cart">
  {% with total_items=cart|length %}
    {% if total_items > 0 %}
      Your cart:
      <a href="{% url "cart:cart_detail" %}">
        {{ total_items }} item{{ total_items|pluralize }},
        ${{ cart.get_total_price }}
      </a>
  {% else %}
    Your cart is empty.
  {% endif %}
{% endwith %}
</div>
```
5. Test the cart total feature

## Display cart total on all pages
The cart total needs to be displayed on all pages, so we can't use a single view to display it. Instead, we'll add it to the *request* by using a [context processor](https://betterprogramming.pub/django-quick-tips-context-processors-da74f887f1fc). 
1. Create a new file in the **cart** application named **context_processors.py**.
2. Create a function in it named cart that accepts a `request` parameter and returns a dictionary with a single element having a key of `'cart'` and a value of `Cart(request)`. The **Cart** class is the model you created.
3. Add your context processor to the `'context_processors'` list in settings.
4. Replace the text `Your cart is empty` in your base template with the following code:
```html
<div class="cart">
  {% with total_items=cart|length %}
    {% if total_items > 0 %}
      Your cart:
      <a href="{% url "cart:cart_detail" %}">
        {{ total_items }} item{{ total_items|pluralize }},
        ${{ cart.get_total_price }}
      </a>
  {% else %}
    Your cart is empty.
  {% endif %}
{% endwith %}
</div>
```
5. Test the cart total feature
# Place orders based on the shopping cart contents
When a customer is finished shopping, the shopping cart data needs to be saved to the database (up to now it's been cached).
## Create the order models
1. Create a new app called **orders** (remember to register it)
2. Create an **Order** model with the following characteristics (note that for string fields the max length is specified in parenthesis)
   Fields:
      - first_name (50)
      - last_name (50)
      - email
      - address (250)
      - postal_code (20)
      - city (100)
      - created (default to current time on creation)
      - updated (default to current time on update)
      - paid (default to False)
    Other characteristics:
       - Sort by most recent creation
       - Standard string representatino should be the id
       - Provide a method called **get_total_cost** which returns to total price of all of the items in the order (see below)
3. Create an **OrderItem** model with the following characteristics:
    Fields
        - order (foreign key to above with `related_name='items'`)
        - product (foreign key with `related_name='order_items'`)
        - price
        - quantity
4. Migrate the new models
## Add the order models to the admin site
1. Add and register the **OrderAdmin** class which sets an appropriate list_display and list_filter
2. To handle the OrderItem, add a class to your admin as follows:
    ```python
    class OrderItemInline(admin.TabularInline):
    model = OrderItem
    raw_id_fields = ['product']
    ```
3. Also add the following line to the **OrderAdmin** class:
    ```inlines = [OrderItemInline]```
4. Test your page at **http://127.0.0.1:8000/admin/orders/order/add/**. It should resemble the following: [AddOrder1](lab_images/AddOrder1.jpg)
## Create the order forms
We need forms to present to the user to allow them to place an order.
1. Create an order form in the **orders** app (in **forms.py**) using the class-based view approach (complete code provided below):
   ```python
    from django import forms
    from .models import Order
    class OrderCreateForm(forms.ModelForm):
    class Meta:
    model = Order
    fields = ['first_name', 'last_name', 'email', 'address',
    'postal_code', 'city']
   ```
2. Create an **order_create** view that does the following:
   - Create a new **Cart** object
   - If the request is a GET, create a new empty **OrderCreateForm**
   - If the request is a POST, create a new **OrderCreateForm** using the POST data. If the form is valid then:
     - Save the order to the **Order** model in the database
     - Save each of the items in the cart to the **OrderItem** model in the database being sure to include the **product**, **price**, and **quantity** and linking it to the **Order**
3. Render the **order/create** template below. Note this should be in `templates/orders/order/create.html` in the **orders** app.
   ```html
    {% extends "shop/base.html" %}
    {% load i18n %}

    {% block title %}
    {% trans "Checkout" %}
    {% endblock %}

    {% block content %}
    <h1>{% trans "Checkout" %}</h1>

    <div class="order-info">
    <h3>{% trans "Your order" %}</h3>
    <ul>
        {% for item in cart %}
        <li>
        {{ item.quantity }}x {{ item.product.name }}
        <span>${{ item.total_price|floatformat:2 }}</span>
        </li>
        {% endfor %}
        {% if cart.coupon %}
        <li>
        {% blocktrans with code=cart.coupon.code discount=cart.coupon.discount %}
            "{{ code }}" ({{ discount }}% off)
        {% endblocktrans %}
        <span class="neg">- ${{ cart.get_discount|floatformat:2 }}</span>
        </li> 
        {% endif %}
    </ul>
    <p>{% trans "Total" %}: ${{ cart.get_total_price_after_discount|floatformat:2 }}</p>
    </div>

    <form method="post" class="order-form">
    {{ form.as_p }}
    <p><input type="submit" value="{% trans "Place order" %}"></p>
    {% csrf_token %}
    </form>
    {% endblock %}
   ```
4. Create the **order/created** template below:
    ```html
    {% extends "shop/base.html" %}
    {% load i18n %}

    {% block title %}
    {% trans "Thank you" %}
    {% endblock %}

    {% block content %}
    <h1>{% trans "Thank you" %}</h1>
    {% blocktrans with order_id=order.id %}
        <p>Your order has been successfully completed. Your order number is <strong>{{ order_id }}</strong>.</p>
    {% endblocktrans %}
    {% endblock %}
    ```
5. Create a URL path in the **orders** to route **create/** to the **order_create** view
6. Add a URL path in the **myshop** project to route **orders/** to the **orders** app
7. Test the app by populating a shopping cart and checking out. Your checkout form should resemble the following:
   [CheckoutForm](lab_images/CheckoutForm.jpg)
When the checkout is complete, you should see a checkout confirmation similar to
    [CheckoutConfirmation](lab_images/CheckoutConfirmation.jpg)

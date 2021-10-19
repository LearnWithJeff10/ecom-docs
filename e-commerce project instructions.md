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
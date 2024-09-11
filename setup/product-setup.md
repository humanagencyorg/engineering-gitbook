---
description: A product represents a single white-labeled instance of the application.
---

# Product Setup

## Product

A product must be defined in order to use the app.  A product defines the hosting domains, branding, and billing settings used within the application.  Multiple products can be defined per server.

### Setup

Run `rake db:load_from_seeds` to configure a product with the following:

* Billing Plans
* Admin Users
* Branding
* Default Workspace
* Application Host

For more details on setup, view [db/seeds/review\_app\_seeds.rb](https://github.com/humanagencyorg/avala/blob/master/db/seeds/review\_app\_seeds.rb).

### Manual Setup

If you would like to manually configure a new product

### Customization

#### Hosting

After a product is setup, additional product domains can be added at `{{your_product_domain}}/admin/whitelabel_domains`.

#### Branding

After a product is setup, brand colors and brand specific links can be modified at  `{{your_product_domain}}/admin/products`.

#### Users

After a product is setup,  additional admin users can be added at

`{{your_product_domain}}/admins`


---
description: What are they and what are they not?
---

# Services

## Overview

Services are a controversial point in Rails development.  It is a pattern outside of the "Rails Way" and borrows concepts from Domain Driven Design.  Here's our take:

{% hint style="info" %}
Before you use a Service, consider if the logic you need to write would be better suited in a helper or in a domain object.
{% endhint %}

### What Services Are

**Easily Unit Testable** - Services should allow you to abstract logic into a Service that is more easily unit testable.

**Immutable** - Services can be initialized with data.  Once initialized, this object cannot be changed; only a new service can be created.

**Stateless **- The Service object should not contain any state across any request.  Calling the service should always yield the same answer given the same input.

{% hint style="info" %}
**Best Practice**: The class `ApplicationService` should be extended so that all params passed to `call` initialize a new Service object.
{% endhint %}

**Easily Stubbable** - Service objects should make it easy to stub out logic in a Controller, FormObject, Worker, or Workflow.

**Easily Mockable** - Service objects should make it easy to mock logic in a Controller, FormObject, Worker, or Workflow..

### What Services Are NOT

**Place to persist data** - No database transactions are committed inside of Services.  This lets Services be completely re-usable throughout the codebase.  It avoids complexity around nested transactions and multiple commits per request.

**Generate JSON** - Serializers should be preferred when it is required to generate JSON for an endpoint.

**Reconstructing Active Record Objects from JSON** - It is preferred that JSON passed from the client should be passed in a format that matches our ActiveRecord objects or a corresponding FormObject.  We recommend avoiding using Service objects to build ActiveRecord objects based on client side JSON.

### Use Cases

**Perform a complex calculation** - This is a great place for a Service.  It provides a single file where logic can be completely unit tested.  A given input will always generate a specific output.

**Wrap an Adapter** - A service can wrap an adapter that makes an external API call.  For example:

1. call an Adapter of an external API
2. build and return an unsaved ActiveRecord object&#x20;

**Build ActiveRecord Objects **- A service can wrap the logic around building a series of ActiveRecord objects that will be persisted.  For example:

1. Receive a webhook for a new Stripe invoice
2. Generate and return ActiveRecord objects for persisting the Invoice in the database.

## Example <a href="example" id="example"></a>

### Build an ActiveRecord Object

```ruby
# frozen_string_literal: true

module ResponseServices
  class BuildRecurringTransaction < ApplicationService
    def initialize(invoice)
      @invoice = invoice
    end

    def call
      response = Response.find_by!(
        stripe_subscription_id: @invoice.subscription,
      )

      response_transaction =
        response.
          response_transactions.
          find_or_initialize_by(stripe_invoice_id: @invoice.id)

      response_transaction.assign_attributes(
        stripe_invoice_id: @invoice.id,
        amount_cents: @invoice.amount_paid,
        currency: @invoice.currency,
        status: @invoice.status,
        payment_type: :recurring,
        payment_service: :stripe,
      )

      response_transaction
    end
  end
end
```

## Testing <a href="testing" id="testing"></a>

Services are unit tested.  Since there is nothing persisted to the database, you should verify:

1. The output of the Service was correct
2. The appropriate stubs were called from the service.

```
Provide a code example here
```

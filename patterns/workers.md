# Workers

## Overview

Workers allow us to run code asynchronously.  We use workers to move code out of the out of the web thread as well as perform long running tasks and migrations.

This app uses Sidekiq, specifically Sidekiq Enterprise, as our worker implementation.

A few points about workers:

**Always Idempotent** - The goal is that workers can be ran multiple times and always produce the same result.  The default assumption should be that the worker will fail and will need to be re-run.

**Primitive Arguments** - When arguments are passed to Sidekiq, they are serialized and used as a key in Redis when looking up the job.  Passing an object that must be serialized can blow out redis memory or cause unpredictable results.  Because of this only primitive objects (numbers, strings) should be passed as arguments to a worker. &#x20;

**Raise Meaningful Exceptions** - When a job fails, we will see the job in the retry column with a print out of the exception that caused the job to fail.  Make sure that the exception raised helps the engineer understand why it failed.

**Avoid silent failures** - A Sidekiq job is considered complete when the job completes without an exception.  Ensure that any API requests or interactions with the database raise an error if the action does not complete.

**Avoid calling workers from other workers** - Things can get very complex when one worker calls another worker.  It can be difficult to trace the logic flow through the codebase.  In these cases, consider using a Workflow to execute a series of workers in sequential steps.

## Example

Here is a simple example of a worker that performs an API call to Stripe.

```ruby
# frozen_string_literal: true

module Asks
  class StripePlanCreateWorker
    include Sidekiq::Worker

    def perform(product_id, ask_id)
      ask = Ask.find(ask_id)
      account_id = ask.asker.stripe_user_id
      params = {
        product: product_id,
        amount: 1,
        currency: ask.currency,
        interval: "month",
        interval_count: 1,
      }
      plan = Stripe::Plan.create(params, stripe_account: account_id)

      ask.update!(plan_id: plan.id)
    end
  end
end
```

## Testing

Sidekiq offers [several testing options for workers](https://github.com/mperham/sidekiq/wiki/Testing#setup).  The strategy you use depends on the what type of test you are writing.

### Unit Testing

Unit testing a worker is done by instantiating the Worker and directly calling the `perform`. &#x20;

```ruby
  it "calls Stripe::Plan#create" do
    account_id = "fake_stripe_account"
    asker = create(:asker, stripe_user_id: account_id)
    ask = create(:ask, asker: asker)
    product = double(id: "prod_123")
    plan = double(id: "plan_123")
    allow(Stripe::Plan).to receive(:create).
      and_return(plan)

    described_class.new.perform(product.id, ask.id)

    expect(Stripe::Plan).to have_received(:create).with(
      {
        product: product.id,
        amount: 1,
        currency: "usd",
        interval: "month",
        interval_count: 1,
      },
      stripe_account: account_id,
    )
  end
```

### System / Request Testing

The app uses `Sidekiq::Testing.fake!` as the default test mode in our apps.  This allows a system spec to check that a particular test was enqueued.

```ruby
expect(SomeClass).to have_enqueued_sidekiq_worker(some_id)
```

Some specs may wish to test an entire workflow through the workers.  In this case they may wrap the test in `Sidekiq::Testing.inline!` .  This will execute each worker as a normal Ruby object. &#x20;

{% hint style="info" %}
`Sidekiq::Testing.inline!` should be used with caution. &#x20;

* When the Sidekiq job is run in using `inline`! it will be executed in the same transaction.  It production it will be in a different transaction.
* Jobs that you would expect to execute asynchronously will execute synchronously. &#x20;
{% endhint %}

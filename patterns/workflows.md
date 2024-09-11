# Workflows

## Overview

Workflows allow us to write multi-step background processes. Instead of having one worker call another worker, we can use workflows to declaratively show how each worker flows into the next.

Workflows are an abstraction on top of [Sidekiq Batches](https://github.com/mperham/sidekiq/wiki/Batches). We use the [simple-sidekiq-batches gem](https://github.com/mpmenne/sidekiq-simple-workflow) to simplify the Sidekiq Batches interface. This abstraction also helps protect us from some "sharp edges" mentioned in the [Sidekiq Batch documentation](https://github.com/mperham/sidekiq/wiki/Batches#notes) (...like empty steps).

A few points about workflows:

**All of the steps are executed linearly** - Each step is executed only after all of the jobs of the previous step complete. So if you 6 workers in `step_1`, all 6 workers will need to complete before `step_2` begins executing.

**Everything is a worker** - the workflow itself implements the Sidekiq `perform` method and is executed in the background. Additionally, each step is executed as a worker.

**Avoid uniqueness on the batch** - Mike Perham makes a specific point about this in the [Sidekiq Batch documentation](https://github.com/mperham/sidekiq/wiki/Batches#notes). It can cause bad things.

**Your batch may never finish...** - Batches are not guaranteed to finish. The entire batch only completes if all of the jobs in the preceding steps are completed. This is obvious, but should be noted.

**Arguments to workflows** - Arguments may be passed directly into the workflow. Arguments may also be added to the `*args` param within `perform`. At the moment, more refactoring of workflows is required to pass new arguments from one step to the next.

## Example

Here is an example of a workflow that is used to provision a whitelabeled domains and emails.

```ruby
module Whitelabels
  class SetupDomainWorkflow
    include Sidekiq::SimpleWorkflow
    include Sidekiq::Worker

    sidekiq_options unique_for: 15.minutes

    def perform(*args)
      start_workflow(*args)
    end

    def step_1(_, options)
      type_id = options["type_id"]
      type = WhitelabelType.find(type_id)
      asker = type.whitelabel_domain.asker

      if !asker.has_sendgrid_subaccount?
        Whitelabels::CreateSendgridAccountWorker.perform_async(asker.id)
      end
    end

    def step_2(_, options)
      type_id = options["type_id"]
      type = WhitelabelType.find(type_id)
      asker = type.whitelabel_domain.asker

      Whitelabels::CreateSendgridDomainWorker.perform_async(type_id)
      Whitelabels::RetrieveSendgridTokenWorker.perform_async(asker.id)
    end

    def step_3(_, options)
      type_id = options["type_id"]

      Whitelabels::CreateDnsimpleRecordsWorker.perform_async(type_id)
    end
  end
end
```

## Testing

Workflows can be integration tested by calling `start_workflow`.

{% hint style="info" %}
Mike Perham (author of Sidekiq) has a strong opinion that batches should not be integration tested. His opinion is that each piece should be unit tested.
{% endhint %}

The method `batch.status.join` is called on each step when in a test environment. This will drain all of the jobs in the previous step.

```ruby
  describe "#step_2" do
    it "should enqueue a worker for create domains" do
      asker = create(:asker, :with_sendgrid)
      domain = create(:whitelabel_domain, :with_email, asker: asker)
      type = domain.email_type

      options = { "type_id" => type.id }
      workflow = described_class.new
      step1_batch = workflow.start_workflow(options)

      expect(Whitelabels::CreateSendgridDomainWorker).
        to have_received(:perform_async).with(type.id)
    end
end
```

# Migrations

## Overview

Migrations are run to change the schema of the database.  Additionally migrations may also need to be run to change the data in the database.&#x20;

For changing the schema, the migration must pass all recommendations of [lol\_dba](https://github.com/plentz/lol\_dba).

For changing data, we run these migrations as separate rake tasks.  These rake tasks are run on staging prior to the deployment of production.  The migrations are typically stored in `lib/data_migration.rb`

The rake task should be **idempotent so that the rake task can be run multiple times.**  For example, we'll often need to rerun the rake task after deploying to production to "catch up" any data that has come in since deployment.

The rake task should also be run within a **Sidekiq Batch** such that a single Sidekiq job is queued for each record that needs to be changed.  Using a Sidekiq Batch provides a couple of benefits:

1. The rake task is able to be run asynchronously.  The rake task will continue to run if the connection is closed.
2. The total completion of the rake task can be seen on the Sidekiq dashboard.
3. Any records that fail migration are highlighted in the Sidekiq Dashboard

The class `BatchMigrator` provides a convenient wrapper around Sidekiq Batch.&#x20;

## Testing

The rake task should be unit tested.  The worker that the rake task calls should be tested to ensure it creates / updates the correct data.

## Example

```
task update_responder_details: :environment do
    product_uuid = ENV.fetch("PRODUCT_UUID")
    responder_uuid = ENV.fetch("RESPONDER_UUID")
    email = ENV.fetch("EMAIL")
    name = ENV.fetch("NAME")
    first_name = ENV.fetch("FIRST_NAME")
    last_name = ENV.fetch("LAST_NAME")

    product = Product.find_by!(uuid: product_uuid)
    ActsAsTenant.with_tenant(product) do
      responder = Responder.find_by!(uuid: responder_uuid)
      BatchMigrator.migrate(
        "Updating responder details for #{responder.uuid}",
        DataMigration::UpdateResponderDetailsWorker.to_s,
        [responder]
      ) do |batch|
        batch.map { |responder| [responder.uuid, email, name, first_name, last_name] }
      end
    end
  end

```

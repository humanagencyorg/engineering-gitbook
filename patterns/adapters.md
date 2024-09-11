# Adapters

## Overview

An Adapter is used as a buffer between the application and an external system.  Adapters are typically used when interacting with APIs.

The Adapter will do the work of translating the application domain into objects that can be used in the domain of an external system. The benefit of Adapters is that they prevent the logic of external systems from becoming too intertwined in the application.

## Testing

Adapters are typically unit tested and use stubs and mocks to ensure that the correct API call is made.

## Example

Here is an effective use of an Adapter for interfacing with Fly.io.  Fly.io provides DNS whitelabeling for users fo the the app.  The Fly.io API accepts GraphQL requests to provision certificates and provide updates about those certificates.

Here you can see that the complexity of the GraphQL requests are hidden behind simple Ruby methods.

```
require "graphql/client"
require "graphql/client/http"

class FlyIoAdapter
  ENDPOINT = "https://api.fly.io/graphql".freeze
  HTTP = GraphQL::Client::HTTP.new(ENDPOINT) do
    def headers(_context)
      fly_api_token = ENV.fetch("FLY_API_TOKEN")
      {"Authorization" => "Bearer #{fly_api_token}"}
    end
  end

  Client = GraphQL::Client.new(
    schema: Rails.root.join("lib/flyio/schema.json").to_s,
    execute: HTTP
  )

  AppQuery = Client.parse <<-GRAPHQL
    query {
      apps {
        nodes {
          id
          name
          hostname
        }
      }
    }
  GRAPHQL

  CreateCertificateQuery = Client.parse <<-GRAPHQL
    mutation($appId: ID!, $hostname: String!) {
      addCertificate(appId: $appId, hostname: $hostname) {
        certificate {
          configured
          acmeDnsConfigured
          acmeAlpnConfigured
          certificateAuthority
          certificateRequestedAt
          dnsProvider
          dnsValidationInstructions
          dnsValidationHostname
          dnsValidationTarget
          hostname
          id
          source
        }
      }
    }
  GRAPHQL

  DeleteCertificateQuery = Client.parse <<-GRAPHQL
    mutation($appId: ID!, $hostname: String!) {
      deleteCertificate(appId: $appId, hostname: $hostname) {
        certificate {
          dnsValidationHostname
          dnsValidationTarget
        }
      }
    }
  GRAPHQL

  CheckCertificateQuery = Client.parse <<-GRAPHQL
    mutation($appId: ID!, $hostname: String!) {
      checkCertificate(input: { appId: $appId, hostname: $hostname }) {
        check {
          acmeDnsConfigured
          dnsConfigured
        }
      }
    }
  GRAPHQL

  def get_app_by_name(app_name)
    result = Client.query(AppQuery)
    HaLogger.info("Fly.io get_app_by_name for #{app_name}", result.to_h)
    node = result.data.apps.nodes.find do |cnode|
      cnode.name == app_name
    end

    return unless node

    {id: node.id, hostname: node.hostname}
  end

  def create_certificate(app_id:, domain:)
    result = Client.query(
      CreateCertificateQuery,
      variables: {
        appId: app_id,
        hostname: domain
      }
    )

    HaLogger.info(
      "Fly.io create_certificate for app_id:#{app_id} domain:#{domain}",
      result.to_h
    )

    return unless result.data.add_certificate?

    cert = result.data.add_certificate.certificate

    {
      dns_validation_hostname: cert.dns_validation_hostname,
      dns_validation_target: cert.dns_validation_target
    }
  end

  def check_certificate(app_id:, domain:)
    result = Client.query(
      CheckCertificateQuery,
      variables: {
        appId: app_id,
        hostname: domain
      }
    )
    HaLogger.info(
      "Fly.io check_certificate for app_id:#{app_id} domain:#{domain}",
      result.to_h
    )
    return unless result.data.check_certificate

    check = result.data.check_certificate.check

    {
      dns_configured: check.dns_configured,
      acme_configured: check.acme_dns_configured
    }
  end

  def delete_certificate(app_id:, domain:)
    result = Client.query(
      DeleteCertificateQuery,
      variables: {
        appId: app_id,
        hostname: domain
      }
    )
    HaLogger.info(
      "Fly.io delete_certificate for app_id:#{app_id} domain:#{domain}",
      result.to_h
    )
  end
end
```

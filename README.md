---
description: Running Experience
---

# Getting Started

## Local Development

Running Experience is simple to get running locally.  this is a local change request

```
$ bin/setup
```

{% hint style="info" %}
This assumes that you have a machine capable of running Ruby, Postgresql (10+), and Redis.  If that is not configured on your machine a few `brew` commands should do the trick 😉 &#x20;
{% endhint %}

This will do a couple of things:

* Install and configure [Lefthook](https://github.com/evilmartians/lefthook)
* Run yarn
* Setup a basic `.env`

### Ngrok

We also use [ngrok](https://ngrok.com) quite a bit in our local development for remote pairing and integrations with 3rd party APIs.  Here is an example ngrok config that lets you stand up:

* Rails reverse proxy to `3000`
* SSH reverse proxy to `22`
* SimpleHTTPServer reverse proxy to `8000`

{% code title="ngrok.yml" %}
```bash
authtoken: your_token
tunnels:
  ssh:
    proto: tcp
    addr: 22
    hostname: hamike.ngrok.io
  rails:
    proto: http
    addr: 3000
    hostname: hamike.ngrok.io
  simple_http:
      proto: http
      addr: 8000
      hostname: hamike.ngrok.io
```
{% endcode %}

{% hint style="info" %}
Store your `ngrok.yml` outside of the repository to avoid committing it to version control.  You can reference an external `ngrok.yml` with the following command

```
$ ngrok start -config ~/.ngrok2/ngrok.yml rails ssh simple_http
```
{% endhint %}

### Servers

There are several servers that you will need to run have a functioning local environment.

#### Rails

```bash
$ rails s
```

#### Webpacker

```bash
$ bin/webpack-dev-server
```

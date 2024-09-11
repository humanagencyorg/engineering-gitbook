# Local Environment

## Local Development

Running The OS is simple to get running locally.

```
$ bin/setup
```

{% hint style="info" %}
This assumes that you have a machine capable of running Ruby, Postgresql (10+), and Redis. If that is not configured on your machine a few `brew` commands should do the trick 😉
{% endhint %}

This will do a couple of things:

* Install and configure [Lefthook](https://github.com/evilmartians/lefthook)
* Run yarn
* Setup a basic `.env`

### Ngrok

We also use [ngrok](https://ngrok.com) quite a bit in our local development for remote pairing and integrations with 3rd party APIs. Here is an example ngrok config that lets you stand up:

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
Store your `ngrok.yml` outside of the repository to avoid committing it to version control. You can reference an external `ngrok.yml` with the following command

```
$ ngrok start -config ~/.ngrok2/ngrok.yml rails ssh simple_http
```
{% endhint %}

### Servers

There are several servers that you will need to run have a functioning local environment.

#### Rails

A standard rails server on any port that you like.

```bash
$ rails s
```

#### Sidekiq

Sidekiq Enterprise for background jobs.

```bash
$ bundle exec sidekiq
```

{% hint style="info" %}
The OS uses a private gem repository for Sidekiq Enterprise. You will need to add the following keys to Bundler.

```
$ BUNDLE_ENTERPRISE__CONTRIBSYS__COM=sidekiq_key
$ bundle config --local enterprise.contribsys.com sidekiq_key
```
{% endhint %}

#### Webpacker

A local Webpacker dev server to hot reload JS and React changes.

```bash
$ bin/webpack-dev-server
```

#### React on Rails Renderer

React on Rails for server side rendering our react components.

```bash
$ yarn run renderer
```

{% hint style="info" %}
The OS uses a private Gem repository for React on Rails. You will need to add the following keys to Bundler.

```
$ bundle config set --local 'rubygems.pkg.github.com' '<username>:<token>'

// Also since we have private npm package in our app too we need to export NPM_TOKEN env variable to be able to install that like so:
export NPM_TOKEN=<token>

// for convinience you can do that in .zshrc or some other file which your termincal calls before start.
```
{% endhint %}

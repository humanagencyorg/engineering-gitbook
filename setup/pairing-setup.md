# Pairing Setup

## Joining a Pairing Session

### Generate a key

1. &#x20;Open your terminal and run the following command.  Please provide your email address so that team members can identify each key.

```bash
$ ssh-keygen -t ed25519 -C "your_email@example.com"
```

2\.  Save your key in the default location

```
> Enter a file in which to save the key (/Users/you/.ssh/id_ed25519): [Press enter]
```

3\.  Enter a security password to your key in case your computer is compromised.

```
> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]
```

4\.  Share your **public key** with your pairing partner so that your machine can authenticate into their machine

```bash
$ cat ~/.ssh/id_rsa.pub
ssh-rsa asdfasdfasdfasdh4isaIfyFamQKudT8xYzBw== yourname@humanagency.com
```

5\.  SSH into your partner's machine by running.  Be sure to substitute your partner's port, computer username, and ngrok host.

```bash
$ ssh -p 12419 mmenne@6.tcp.ngrok.io
```

6\. Attach to their TMUX session.

```bash
$ tmux attach
```

## Hosting a Pairing Session

### Configure your Mac

You will need to allow other machines the ability to SSH into your Macbook using private key.

1. &#x20;Open System Preferences and open "Internet Sharing"

![](<../.gitbook/assets/Screen Shot 2021-08-16 at 9.37.50 PM.png>)

2\.  Turn on "Remote Login"

![](<../.gitbook/assets/Screen Shot 2021-08-16 at 9.36.13 PM.png>)

### Configure SSH

You will need to add your partners **public key** to your authorized keys file. &#x20;

```bash
$ cat your_partners_key.pub >> ~/.ssh/authorized_keys
```

{% hint style="info" %}
&#x20;This will allow your partner to log into your Macbook without a password by using the private key stored on their computer.
{% endhint %}

### Setup Ngrok

You'll want to run a reverse proxy forwarding to port 22 of your local computer.&#x20;

{% code title="ngrok.yml" %}
```yaml
# ngrok auth config
# set NGROK_AUTHTOKEN
tunnels:
  ssh:
    proto: tcp
    addr: 22
  rails:
    proto: http
    addr: 3000
```
{% endcode %}

```yaml
$  ngrok start ssh rails
```

After Ngrok spins up, you will be able to see the host and the port number of your reverse proxy.

```yaml
ngrok by @inconshreveable                                                                                                                                                                                                     (Ctrl+C to quit)

Session Status                online
Account                       Mike Menne (Plan: Basic)
Update                        update available (version 2.3.40, Ctrl-U to update)
Version                       2.3.35
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    tcp://6.tcp.ngrok.io:11669 -> localhost:22
Forwarding                    http://hamike.ngrok.io -> http://localhost:3000
Forwarding                    https://hamike.ngrok.io -> http://localhost:3000

Connections                   ttl     opn     rt1     rt5     p50     p90
                              0       0       0.00    0.00    0.00    0.00
```

Construct the following ssh command and send it to your pairing partner.

```yaml
$ ssh -p 11669 your_username@6.tcp.ngrok.io
```

{% hint style="info" %}
You can get your username by looking up the name of your home directory. Run `pwd`
{% endhint %}

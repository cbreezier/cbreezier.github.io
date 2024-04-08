---
title: TLS certificates for locally hosted webservices
tags:
  - networking
date: 2024-04-08 21:44:29
---

It's 2024, and there's generally no excuse for any website to be served over 
HTTP instead of HTTPS. While SSL/TLS certificates used to be expensive and 
difficult to manage, ever since Let's Encrypt launched in 2016 it's become 
almost unnervingly simple to generate valid certificates for your own 
websites for the exceedingly low price of Free.

But what about locally hosted websites? That is, websites that are hosted on 
your local machine, or another machine on your LAN (Local Area Network). 
These websites might be accessed via something like [http://192.168.1.123:8093](http://192.168.1.123:8093)

There's nothing terribly wrong about this; if you're following tutorials to 
set up your home lab, you'll likely have a whole bunch of services that look 
like this. But there are also several reasons why you might _not_ want to 
access them via HTTP and the LAN IP:

 - It's kind of ugly and you'd like to be able to use a nicer name like 
   [https://subdomain.mydomain.com](https://subdomain.mydomain.com)
 - The "Not Secure" badge in the address bar upsets you
 - Sometimes your browser shows a big scary warning screen and you have to 
   click lots of buttons to proceed (yes, two buttons is _lots_)
 - You want to use [modern web APIs that are restricted to secure contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts/features_restricted_to_secure_contexts)

![Your connection is not private](your_connection_is_not_private.png)
<div style="text-align: center"><em>Did you know that in Chrome, you 
can type thisisunsafe to get past this screen?</em></div>

## DNS

Just a quick note here about how to make something like [https://subdomain.mydomain.com](https://subdomain.mydomain.com)
point to your non-web-accessible server at 192.168.1.123.

It's actually very simple - remember that DNS just resolves a domain name to an IP address. 
While it may be better to run your own [local DNS server](https://www.isc.org/bind/),
there's really nothing stopping you from using something like [Cloudflare DNS](https://www.cloudflare.com/learning/dns/what-is-1.1.1.1/)
to let the whole world know that some domain should resolve to a local IP.

It doesn't make your local LAN any more accessible to the outside internet.

### HSTS preload list

By the way, there's _another_ reason why you might _need_ a TLS certificate. 
Originally, I was happy to just set up DNS and access my sites via HTTP.

But there's this thing called the [HSTS preload list](https://hstspreload.org/)
which is hard-coded into modern browsers. Any domain that appears on this 
list is forced to upgrade to HTTPS, even if it was originally accessed via 
HTTP and the webserver didn't request an upgrade.

The surprising part is that Google [added all of the TLDs that it owns
to the HSTS preload list](https://endtimes.dev/dev-and-hsts-preload/#other-domains). 
This means that if you have a `.dev` or a `.you` domain or any of the other 
50+ TLDs that Google owns, you literally _must_ have a valid TLS certificate 
for your websites.

![All .dev domains are preloaded](hsts_preload_active.png)
<div style="text-align: center"><em>Thanks Google. I mean, it's a 
good thing, but still...</em></div>

## Standard Let's Encrypt setup

The problem is that the most stock-standard method of deploying Let's Encrypt 
certificates requires that your website is accessible to the public internet,
so that Let's Encrypt's servers can access a file on your webserver and prove
that you have control over that webserver. The process looks something like this:

1. You install `certbot` on your webserver, which automates the whole process
2. `certbot` puts a file on your webserver which contains a special token, 
   making it accessible at `http://<YOUR_DOMAIN>/.well-known/acme-challenge/<TOKEN>`
3. Let's Encrypt's servers access that file, verifies its authenticity, and 
   issues you a certificate
4. `certbot` can even update your webserver configuration to use this 
   certificate to properly handle TLS termination

This is called the [HTTP-01 challenge](https://letsencrypt.org/docs/challenge-types/), and you can issue a certificate for a 
particular domain using this method. The main benefit is that the entire 
process is _entirely automated_, since `certbot` can modify your webserver 
to pass the challenge. It can even handle renewals, which is _amazing_.

## Some sub-optimal workarounds

### Temporarily make your local webserver public accessible

After letting `certbot` and Let's Encrypt complete the HTTP-01 challenge 
successfully, you could turn off your port forwarding and you would have a 
valid certificate...for 90 days. Have fun doing that every 90 days for 
certificate renewal, not to mention how icky is feels to have your private 
webservers open to attack, even temporarily.

### Be your own Certificate Authority

Instead of letting Let's Encrypt or some other recognised CA sign your 
certificates, you could become your own CA and sign certificates for your 
services. But this means that every device that you want to use with your 
services has to recognise your new CA by installing the CA root certificate 
on it.

Good luck doing that for your computer(s), mobile devices, your spouse's 
devices, any guests that come over...

## Confusing advice out there

The main reason I wrote this post is because it's surprisingly difficult to 
find the (really quite simple) solution to this problem. There's even a 
page from 
the official Let's Encrypt website that talks about [certificates for 
localhost](https://letsencrypt.org/docs/certificates-for-localhost/) that 
seems relevant and is often linked to, but really isn't.

That page talks about the very specific case about a native app deployed 
alongside a webserver to be run on `localhost`, and the difficulties of 
getting a valid certificate issued in this case.

## DNS-01 challenge

The [DNS-01 challenge](https://letsencrypt.org/docs/challenge-types/) is 
another method of proving ownership of a domain to issue a certificate. In 
fact, it's actually a _stronger_ proof than the HTTP-01 challenge because it 
proves that you own the entire domain (including all sub-domains), instead 
of just a single domain name. This means you can issue wildcard certificates 
(eg, for `*.mydomain.com`).

The beauty of this challenge type is that it doesn't require your webserver 
to be publicly accessible; instead you need to add a special record to a TXT 
entry on your DNS server to prove ownership.

It's no secret that this challenge type exists. It's just rarer - perhaps 
because it comes with the somewhat-significant downside of needing to store 
API keys for your DNS provider on your webserver to allow `certbot` to 
automate the challenge. Additionally, your DNS provider actually 
needs to have API support for creating DNS records. Cloudflare is, once 
again, fantastic on this front (and free for simple use cases like ours).

Of course, because our webserver is local and not accessible to the internet,
the attack surface is much reduced and keeping API credentials on the webserver 
probably isn't a big deal.

## Putting it all together

Here's the setup I ended up with:
 - DNS records that all resolve to the same local IP address on Cloudflare 
   (because I haven't gotten around to setting up a local DNS server yet)
 - An [nginx](https://www.nginx.com/) reverse proxy at that IP, which looks 
   at the `Host` header and forwards traffic to other local webservices
 - Certbot set up alongside nginx to automate TLS certificate renewals using 
   the DNS-01 challenge

I'm running nginx in a [Docker](https://www.docker.com/) container because 
it makes everything _so easy_. I even found an off-the-shelf container that 
already has nginx and certbot set up. Here's the `docker compose` file:

```dockercompose
services:
  nginx:
    # https://github.com/JonasAlfredsson/docker-nginx-certbot
    image: jonasal/nginx-certbot:5.0.1
    container_name: nginx
    restart: 'unless-stopped'
    volumes:
      # The final certificates are stored here
      # You must also provide your DNS provider's API key as a file here
      # See https://github.com/JonasAlfredsson/docker-nginx-certbot/blob/master/docs/certbot_authenticators.md
      - ./letsencrypt:/etc/letsencrypt
      # Makes it easy to configure nginx
      - ./conf.d:/etc/nginx/conf.d
    ports:
      - "80:80"
      - "443:443"
    environment:
      - CERTBOT_EMAIL=<your email>
      # Tells cerbot to use DNS-01, specifically with Cloudflare
      - CERTBOT_AUTHENTICATOR=dns-cloudflare
```

Here's the `nginx` config:

```nginx
server { 
    # Listen to port 443 on both IPv4 and IPv6. 
    listen 443 ssl; 
    listen [::]:443 ssl; 
 
    # Domain names this server should respond to 
    server_name subdomain.mydomain.com; # certbot_domain:*.mydomain.com 
 
    # Load the certificate files 
    ssl_certificate         /etc/letsencrypt/live/wildcard.mydomain.com.dns-cloudflare/fullchain.pem; 
    ssl_certificate_key     /etc/letsencrypt/live/wildcard.mydomain.com.dns-cloudflare/privkey.pem; 
    ssl_trusted_certificate /etc/letsencrypt/live/wildcard.mydomain.com.dns-cloudflare/chain.pem; 
 
    ssl_dhparam /etc/letsencrypt/dhparams/dhparam.pem; 
 
    proxy_http_version 1.1; 
    proxy_set_header   Upgrade $http_upgrade; 
    proxy_set_header   Connection $http_connection; 
    proxy_set_header   Host $http_host; 
    proxy_set_header   X-Real-IP $remote_addr; 
    proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for; 
    proxy_set_header   X-Forwarded-Proto $scheme; 
 
    location / { 
        proxy_pass http://192.168.1.123:9876; 
    } 
}
```

Note the special `# certbot_domain:*.mydomain.com` comment. This is a hint 
that the scripts in this particular Docker image understand, so that they'll 
ask certbot to use this domain and issue a wildcard certificate instead of 
using the `server_name` and generating a certificate for each subdomain. See 
the [issue](https://github.com/JonasAlfredsson/docker-nginx-certbot/issues/130)
and the [docs](https://github.com/JonasAlfredsson/docker-nginx-certbot/blob/master/docs/advanced_usage.md#override-server_name)
and the [example](https://github.com/JonasAlfredsson/docker-nginx-certbot/blob/master/examples/example_server_overrides.conf).

The scripts actually automatically add those `ssl_*` directives. It's pretty 
neat.

After this simple config, you can now access [https://subdomain.mydomain.com](https://subdomain.mydomain.com) 
and everything works perfectly.
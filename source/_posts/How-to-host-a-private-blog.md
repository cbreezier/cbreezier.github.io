---
title: How to host a private blog
date: 2024-06-01 21:06:22
tags:
---


I wanted to write a private blog, primarily chronicling my journey and 
thoughts as a parent, as well as serving as a diary for the growth and 
development of my kids.

This would be a blog that was very personal to me, where my intended 
viewership is mostly myself (or rather, my future self). But I also wanted to be
able to share it with close friends and family.

## Why not make it public?

A normal, publicly accessible website would serve my needs just fine. But I 
had two competing requirements that were completely at odds with each other:

1. I wanted to use the real names of my kids and my partner, as well as 
   upload photos and videos that were completely uncensored.
2. I didn't want all of this to be visible to the World Wide Dangerous Web.

### An uncensored blog

Many people write about their kids and family by using aliases, fake names, 
and censored photos.

If the intention is to write a blog that will be read 
by hundreds or thousands of people that you don't know, then that may be the 
only option you have available to you. That's a completely valid use case, 
and I appreciate some of the blogs from other parents that I've been able to 
glean knowledge from.

However, as a personal blog, the idea just didn't sit right with me. It's a 
bit hard to describe, but I didn't want to feel like I was doing something 
shameful or dangerous. An analogy would be if I constantly used fake names 
when talking to my partner about our children, for fear of the neighbours 
hearing.

It's a kind of pressure that I didn't want to deal with while writing.

## The dangers of the internet

Why do we need to protect the names and photos of our kids anyway?

I've heard a lot of reasons, ranging from the generic "but privacy" to the 
nauseating possibility of someone using AI deepfakes to create pornographic 
material from photos.

However, I found the following argument somewhat more convincing: consent.

My kids are unable to consent to having their personal details published to 
the wide web. And once things are on the internet, you can never take it back.

## Options that I discarded

I considered and discarded the following options:

- a managed service like Blogger with privacy controls
  - I want to host and control the tech stack myself...just because
  - still requires uploading my personal data to a 3rd party
  - finnicky to make people create user accounts and log in
- hosting something that allows logging in
  - much more complex tech stack
  - annoying to use
- throwing basic auth on every page with a shared password
  - very annoying to use
  - can't revoke access to a single user

## IP whitelisting

I ended up on the idea of simply whitelisting IP addresses that could access 
the personal blog. I would self-host the blog on my home server, and 
configure the IP whitelist on my nginx reverse proxy. This has _numerous_ 
disadvantages:

- uptime of my home server is almost certainly worse than Blogger (but who 
  cares)
- updating the whitelist is a bit annoying (but really not that bad)
- people with dynamic IPV4 addresses (read: everyone) could change IPs
- people on CGNAT would mean that their entire cohort sharing the same 
  public IP could technically access my blog (but who cares - the scrapers 
  certainly can't)
- I can't just share the blog out with a single link - I have to actually 
  get their IP address

But ultimately, none of these were deal-breakers to me for my particular use 
case.


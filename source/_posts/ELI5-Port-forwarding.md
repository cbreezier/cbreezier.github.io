---
title: 'ELI5: Port forwarding'
tags:
  - eli5
  - networking
date: 2024-04-09 21:56:46
---


Once upon a time, there was a very, very long street. It was so long 
that you could build over 4 _billion_ houses along this street. The people 
who designed and built this street were very smart, and they thought that 
this would be enough for everyone to have a house on the street.

This street was called **Fourth Street**.

The people who lived on Fourth Street never left their house, but they loved 
sending letters to each other. As long as you knew the **address** of someone 
else living on Fourth Street, you could get a letter delivered to _any_ 
house on the street very very quickly.

Every house on Fourth Street had a very special letterbox. The letterbox had 
many, _many_ little compartments in it. In fact, each letterbox had 65,535 
little compartments. These were called **ports**.

People would use these ports to manage their mail. For example, a person 
might want all of the letters related to Minecraft to arrive in one 
particular port. So, in order to send a letter to someone, you had to 
specify a port in addition to the address.

At first, there weren't many people living on Fourth Street. But it grew in 
popularity very quickly. Everybody wanted a house of their own, with 
their own letterbox that they could use to exchange letters with 
everyone else. Within 10 years, over half of the street was occupied.

In another 5 years, the entire street was completely filled. 

### We need taller buildings

There were still more and more people that wanted to move into Fourth Street.
It was decided that instead of giving a house to every single person, 
multiple people would have to share a house. The houses on Fourth Street 
started to be converted into taller apartments, where many people could live 
at the same address.

Each apartment was still only allowed to have a single special letterbox on 
the street. But the people living on Fourth Street loved the special letterboxes
so much that they would install one right outside of their room door.

Even so, the postmen could only deliver letters to the shared letterbox on 
the street. They didn't know how to navigate the insides of the apartment.

To solve this problem, each apartment would have a **doorman** who would check 
the shared mailbox for letters, and deliver them to the letterboxes _inside_ 
the apartment.

But how did the doorman know where each letter should go? The doorman had a 
little **rulebook** that had instructions like:

> Letters in port 80 should be delivered to port 9000 in the third room's 
> letterbox.

If you wanted to talk to someone about Minecraft, you could just tell them 
to send letters to your street address on port 100, and then tell your 
doorman to forward all letters in port 100 to port 200 in your personal 
letterbox.

And so, everyone was able to share the addresses on Fourth Street and send 
letters to each other again.

### What's next?

Meanwhile, there is a new street being built. This one is even longer than 
Fourth Street. It is incredibly, mind-bogglingly long. This new street 
can fit far more than a measly 4 billion addresses. It can fit 340,282,
366,920,938,463,463,374,607,431,768,211,456 (340 undecillion) addresses. 
Surely, this time, we will never ever run out addresses again.

This new street is called **Sixth Street**.

## So... what's port forwarding?

In real computing terms, Fourth Street is the **IPV4 address space**. There are 
truly over 4 billion IPV4 addresses (although not all are reserved for use 
as public IP addresses), and we have truly run out since 2011.

Computers also communicate via ports. Instead of letters, they send 
**packets**. There are certain standard ports that everyone has agreed to 
use for certain purposes. For example:

- Port 80: default port for HTTP web traffic
- Port 443: default port for HTTPS encrypted web traffic
- Port 25565: default port for hosting Minecraft servers
- etc

But you can really use any port you like - there's no difference.

The street-level address from the analogy would be your **public IP 
address**, and it might look something like `172.217.167.78`. This part of 
the network is called the **Wider Area Network (WAN)**.

Within the apartment we define the **Local Area Network (LAN)**, and each room 
would have a **local IP address** which might look something like `192.168.1.1`.

The doorman described in the Fourth Street analogy would be a **router** in 
computing terms, and the job they are performing is called **Network 
Address Translation (NAT)**. 

The router is the bridge between the WAN and the LAN.

The little rulebook described in the analogy are the **port forwarding 
rules** that you can configure on your router. These rules simply tell your 
router how to route external WAN traffic on a particular port to an internal 
LAN address on a particular port.

It's common for the external port to be the same as the internal one (eg, 
external port `25565` to internal `192.168.1.10:25565`), but there is 
absolutely no requirement for that to be the case.

### Double NAT

Just a note - the most common problem you'll run into if you can't get port 
forwarding to work is when your network is set up so that there are _two_ 
layers of routers performing NAT. This is called **double NAT**.

Unless you have very special requirements, there is almost never a good reason
to have more than one router performing NAT in your home network.

You could probably port forward from router 1 to router 2, then from router 
2 to your computer. But the better solution is to restructure your network 
so that you don't have double NAT.

Hopefully, you have a better understanding of how it all works and are 
better equipped to debug your home network now :)
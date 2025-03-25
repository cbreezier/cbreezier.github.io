---
title: WiFi infrastructure demystified - APs vs WDS vs Mesh
tags:
  - networking
date: 2025-03-25 21:46:50
---


If there's one thing that frustrates me, it's Terminology That Doesn't Really
Mean Anything. And the world of WiFi infrastructure appears to be full of them,
from vague technical specifications to meaningless marketing terms.

Let's cut through the jargon and try and distill all of the confusing
terminology into what they truly mean.

This post assumes some basic understanding of WiFi networking. I'll be
focusing on covering concepts that relate to the infrastructure and topology
of a network with multiple components to it. The only components we're
really interested in are:

- the nodes (eg, WAPs, repeaters, even switches)
- the clients (eg, laptop or smartphone)

I don't think that "node" is standard terminology, but I don't know of a
better catch-all to use. Essentially, I'm referring to all the components of
the WiFi infrastructure that eventually route client packets to the internet.

---

This post will be broken down into two main sections: **back-haul** and
**roaming**.

Back-haul refers to the way the nodes talk to each other.

Roaming refers to the way that clients switch between which node they're
connected to.

Ultimately, regardless of what you call it, almost every decision you make
in terms of configuring your WiFi infrastructure and topology comes down to
features and trade-offs relating to these two concepts.

## Back-haul

You can think of the back-haul as the core of the network, whereas the
clients that connect to the nodes sit on the edge of this infrastructure.

Within the different kinds of back-haul, we're going to split it again into
two main categories: wired and wireless.

### Wired

With a wired back-haul, the pieces of infrastructure are connected via
physical cables. This is simple, performant, and generally the best choice
if possible.

Of course, the problem is that running cabling across a site may be
prohibitively expensive or difficult.

#### Cat6/Cat5e/Cat5 Ethernet

This is the traditional, reliable back-haul with no compromises on performance.
Ethernet is full duplex, low latency, and low packet loss.

If this is a viable option, there's no reason to consider wireless back-haul.

#### Ethernet over Power

If you can't run dedicated Ethernet cabling, you can use Ethernet over Power
adapters to leverage your existing mains power cables.

It's honestly not bad...when it works. There will be significantly more
interference due to the high voltage running through the cabling at the same
time, and therefore packet loss.

You also have to worry about which circuits you're running your ethernet
over; if your destination is on a different circuit and you have to go
through a breaker, it may not work as well.

If you have completely disjoint circuits (eg, 3-phase power), you will obviously
not be able to connect across them.

### Wireless

This is where it gets interesting. Instead of a wired backbone, the nodes
can communicate wirelessly.

#### Repeaters / Extenders

The meaning of this term seems to differ between different sources. However,
the most common definition seems to be cheap, simple devices that simply
rebroadcast any WiFi packets that they pick up.

The repeaters are typically configured to be on the same channel as the node
that they're repeating.

An analogy would be a bunch of people playing Chinese whispers, except
they're actually kind of far apart and they're all shouting. You can hear
the closest person pretty well, but you can also kind of hear the person
further away, and they try and take turns shouting but sometimes they shout at
the same time and you have to ask them to say it again, and you have to wait for
them to take turns shouting each message up the chain.

The bandwidth decreases by 50% for each repeater, because they have to take
turns shouting up the chain.

The interference gets pretty bad because they're all on the same channel.

Why don't repeaters receive on one channel and then shout on another? One
reason I've seen is that these cheap repeaters typically only have one 
hardware radio. Switching the physical radio to transmit/receive on different
channels isn't instantaneous, so it can make things slower or
even miss packets.

Another reason is to avoid hogging all available channels in your area and
deprive your neighbours of usable frequencies.

#### WDS (Wireless Distribution System)

This specification is supposed to enable a multi-hop, wireless back-haul
network to be established. It's an earlier technology and certainly could be
classified as a "mesh" technology.

However, this isn't actually a full specification - it doesn't prescribe any
particular routing algorithm. This causes different manufacturers to
implement different proprietary routing protocols that aren't compatible
with each other.

Specifically, WDS simply defines a 4th address to be added to each packet:

- intended receiver
- transmitting node
- BSSID of the network
- original sender (new 4th field added by WDS)

So WDS describes the packet format that makes a multi-hop, mesh-y back-haul
_possible_, but it's really kind of meaningless and completely misunderstood
and misrepresented in most places.

It's not even really explicitly defined in the IEEE 802.11-1999 standard -
it's kind of just _mentioned_. It's not until 2005 that its meaning is
clarified in [802.11-05/0710r0][4-address-format].

#### IEEE 802.11s - WiFi Mesh?

An extension to the IEEE 802.11 standard for WiFi that was eventually
ratified in 2012, this actually does describe a routing protocol (on top of
the 4-address format defined by WDS).

The nice thing is that if you can find multiple routers that support 802.11s, 
even across different brands, they should be interoperable.

However, this standard is not commonly
implemented among manufacturers. **Sometimes when vendors say "mesh" they mean 
802.11s**. More often than not, they'll be implementing their own proprietary
routing protocol for their products which isn't compatible with any other brand.

#### Single vs dual/multi radio nodes

If a node only has a single physical radio, it cannot simultaneously 
transmit and receive. This means that the available bandwidth is effectively halved.

However, nodes with multiple radios can receive on one radio and transmit on
the other. For example, an access point may talk to the back-haul
infrastructure on 5Ghz while connecting to clients on 2.4Ghz.

## Roaming

The back-haul is all about the infrastructure, but we haven't said anything
about how client devices connect to said infrastructure. In practice, the
decision about what kind of interface to present to client devices really
comes down to what you want to achieve in terms of roaming.

At any point in time, a client device is able to see multiple candidates to
connect to. As a high level summary, each candidate is uniquely identified
by a combination of:

- SSID (the "name" of the WiFi network)
- Channel (the exact frequency being broadcasted on)
- Mac address (the hardware identifier of the AP)
- Authentication (eg, the password)

### All roaming decisions are client-led

When a client device sees multiple candidates that it can connect to, all
decisions about whether to switch from one candidate to another is done by
the device. Historically, there has been very little that an AP can do to
influence that decision.

### Sharing an SSID

Broadcasting the same SSID and authentication on different APs is the
general strategy to make all of the APs appear as one cohesive system. To
the end user, there will only be a single WiFi network.

However, the device is aware that there are multiple APs and therefore
multiple connection candidates, because the channel and mac address may be
different for each AP.

### MCA - multi channel architecture

This is the "standard" setup, where each AP broadcasts on a different
channel. This is important to avoid interference with each other.

In this architecture, all roaming decisions are client-led (eg, by the 
mobile device).

### SCA - single channel architecture

This is a general concept, not an exact specification or technology.
Different manufacturers implement this differently and call it different
names. The idea here is for every AP to broadcast on the exact same channel
and even spoof the same mac address.

This prevents the client device from even realising that there are multiple
APs, and therefore removes the ability for the client device to perform any
roaming decisions.

Instead, the responsibility is shifted completely onto the APs. In such a
system, it's crucial for the APs to be able to communicate with each other
over the back-haul network in order to negotiate which one will be in charge
of talking to the client device.

Such an architecture allows for the greatest control over roaming, and the
theoretical best hand-off between APs. Poor roaming decisions on clients are 
completely bypassed, and it's possible to achieve "zero handoff" where the 
switch in AP is immediate and transparent to the client.

However, there are also some big disadvantages. Sharing the same channel 
makes the entire system more vulnerable to external interference, while also 
reducing the available bandwidth (can't make use of multiple channels). The 
implementation of SCA is also hugely variable and proprietary, so you're 
heavily subject to vendor lock-in on a single brand.

The biggest issue is that there is a huge burden of complexity placed on the 
controller in charge of making all these roaming decisions. While SCA offers 
the greatest _theoretical_ roaming performance, in practice these systems 
don't always work so well.

SCA systems tend to be very enterprise and very expensive.

### Fast Roaming - 802.11/k/v/r
There are a relatively new set of standards that intend to improve roaming 
performance. They all operate in a standard Multi-Channel Architecture, and 
describe a co-operative roaming experience between the client and the APs.

The transition towards these modern standards highlight the biggest change 
in the WiFi roaming landscape in recent years - clients have become much 
more sophisticated (think smartphones) and no longer tend to make incredibly 
bad roaming decisions.

#### 802.11k - Assisted roaming with neighbour report
APs help clients find other nearby APs to roam to by providing a "Neighbour 
Report", which reduces the time a client needs to spend scanning for APs itself.

#### 802.11v - Client steering
Allows APs to actively provide roaming suggestions to clients.

#### 802.11r - Fast transition
Speeds up roaming transitions by greatly reducing the time spent during 
reauthentication with a new AP, which involves a 4-way handshake. This standard 
allows APs to pre-negotiate encryption keys.


## Closing thoughts
Now that we've torn away the marketing facade and exposed the fundamental 
technologies that actually underpin a WiFi network, I hope you're suitably 
confused.

Maybe it's a good thing to just spout "WiFi Mesh" while waggling our hands 
after all.


[4-address-format]: https://www.ieee802.org/1/files/public/802_architecture_group/802-11/4-address-format.doc
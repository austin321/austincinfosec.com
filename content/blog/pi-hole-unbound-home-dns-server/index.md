---
title: "Cutting Out the Middleman: Pi-hole + Unbound at Home"
date: 2026-09-21T09:30:00-04:00
slug: "pi-hole-unbound-home-dns-server"
summary: "It started with my grandfather trying to listen to Taiwanese music on YouTube and getting hit with 90-120 seconds of unskippable ads before every song. That was the moment I decided to stop renting my DNS from someone else and start resolving it myself."
cover: ""
categories: ["Lab", "Networking"]
tags: ["pi-hole", "unbound", "dns", "networking", "privacy", "self-hosted"]
---

It started with my grandfather trying to listen to Taiwanese music on YouTube. He watches these 2-3 hour long compilations of this one artist. Every 20 minutes, he'd get hit with 90 to 120 seconds of unskippable ads. Watching him sit there waiting it out was the final push I needed to stop putting up with it and actually fix DNS at the network level for the house.

## The Problem With "Just Use a Better DNS"

The easy answer is to point your router at a public resolver like Google's `8.8.8.8` or Cloudflare's `1.1.1.1` and call it a day. That's still a middleman, though. Every domain anyone in the house looks up gets funneled through a company whose business model, in more than one case, is built on knowing exactly that kind of thing. Swapping one third party for another doesn't provide us any privacy.

I started looking online... what I really wanted was:

1. DNS-level ad and tracker blocking for every device on the network, automatically, with no per-device setup
2. Full control over what gets resolved and logged
3. No upstream DNS company sitting in the middle of every lookup

That meant running my own **recursive** resolver instead of forwarding queries to anyone else's.


## Pi-hole Handles the Blocking, Unbound Handles the Resolving

Pi-hole is the piece most networking enthusiasts will be aware of. It just sits between your devices and the internet as a DNS sinkhole, checking every query against curated blocklists and returning nothing for known ad and tracker domains. This is already such a great improvement! Out of the box, though, Pi-hole still forwards every query it doesn't block up to an upstream DNS provider. 

That's where **Unbound** comes in. Unbound is a validating, recursive DNS resolver, meaning instead of asking Google or Cloudflare "hey, what's the IP for this domain," it does the full lookup itself: starting at the root servers, walking down through the TLD servers, and finally querying the authoritative name server for the domain directly. No single company ever sees my complete resolution history, because no single company is doing the resolving besides me.

So the chain becomes:

```
Device → Pi-hole (filters known-bad domains) → Unbound (does the actual recursive lookup) → Root/TLD/Authoritative servers
```

To simplify: If my dog, Newton, is sick and I need to call the vet I'll first check my contacts, then my phonebook, then Google it. Google will then know that on this particular date that I am trying to reach a veterinarian and will spam me with ads related to pets and pet health. 

Unbound is a constantly updated phonebook with eveything in it so I don't need to hit the upstream Google part. Therefore, much less trackers.

## Setting Up Pi-hole

Installed on a small Linux box that stays on 24/7 on the home network:

```bash
curl -sSL https://install.pi-hole.net | bash
```

The installer walks through picking an interface, a temporary upstream DNS provider (this gets replaced shortly), and which blocklists to load by default. I layered in a handful of additional lists through Pi-hole's Group Management afterward, focused on ad networks and tracker domains rather than anything overly aggressive that would start breaking sites.

## Configuring Unbound as the Recursive Resolver

Installed Unbound alongside it:

```bash
sudo apt install unbound
```

Then dropped a config in `/etc/unbound/unbound.conf.d/pi-hole.conf` so it listens locally on a non-standard port, does full recursion, and validates DNSSEC:

```
server:
    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-ip6: no
    do-udp: yes
    do-tcp: yes

    harden-glue: yes
    harden-dnssec-stripped: yes
    use-caps-for-id: no
    edns-buffer-size: 1232
    prefetch: yes
    num-threads: 1
    so-rcvbuf: 1m

    private-address: 192.168.0.0/16
    private-address: 10.0.0.0/8
    private-address: 172.16.0.0/12
```

Unbound needs a current root hints file to know where the root servers live:

```bash
sudo curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.root
```

DNSSEC validation comes from letting Unbound manage its own trust anchor:

```bash
sudo unbound-anchor -a /var/lib/unbound/root.key
sudo systemctl restart unbound
```

Then a couple of sanity checks. First that resolution works at all:

```bash
dig @127.0.0.1 -p 5335 google.com
```

And that DNSSEC is actually being enforced, using the standard verteiltesysteme test domains — one that should fail validation and one that should pass:

```bash
dig sigfail.verteiltesysteme.net @127.0.0.1 -p 5335   # should come back SERVFAIL
dig sigok.verteiltesysteme.net @127.0.0.1 -p 5335      # should resolve normally
```

## Pointing Pi-hole at Unbound Instead of the Outside World

Last step of the actual DNS chain: in the Pi-hole admin under **Settings → DNS**, I unchecked every public upstream provider and set a custom upstream pointing back at Unbound:

```
127.0.0.1#5335
```

From that point on, Pi-hole filters the obviously bad domains, and everything that survives that filter gets resolved by Unbound directly against the authoritative source, never touching Google's or Cloudflare's resolvers.

## Rolling It Out to the Whole House

I gave the Pi-hole box a static IP reservation on the router, then changed the router's DHCP options so it hands out that IP as the DNS server to every device that joins the network. That way nothing needed to be configured by hand, my grandfather's TV, phones, laptops, everything just inherits the filtering and the recursive resolution automatically the moment it connects to WiFi.

## Did It Actually Fix the YouTube Ads?

Mostly. The blocklists I loaded in target the ad domains YouTube uses so the frequency and length of what my grandfather sees dropped noticeably. It's not a perfect 100% since some ad delivery is served off the same CDN as the video itself, but the difference going from a guaranteed 90-120 second unskippable block to a fraction of that most of the time was exactly the win I was after. I'll see if I can iterate on this! 

## Takeaways

Getting DNSSEC validation working end to end, watching Unbound walk the root/TLD/authoritative chain instead of just trusting whatever a third party handed back, and pushing all of it out to the whole house through DHCP with zero per-device setup was pretty cool exercise in gaining back some of the control I have over my network.

No more middleman between this house and the DNS root servers, and no more 120 seconds of ads standing between my grandfather and his music.

Austin

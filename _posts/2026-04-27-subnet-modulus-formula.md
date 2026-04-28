---
title: "The Subnet Modulus Formula: A Shortcut for Network Addresses"
date: 2026-04-27 12:00:00 -0500
categories: [Networking, Subnetting]
tags: [networking, subnetting, ipv4, cidr, network-plus, ccna]
---

If you've spent any time studying for Network+ or CCNA, or you've just tried to teach yourself subnetting, you know the part I'm talking about. Someone gives you an IP and a subnet mask and asks what the network address is. Out come the tables. You start counting by 16s. You sketch a few ranges on scratch paper and hope you didn't lose count somewhere.

There's a quicker way I picked up that I wanted to share. It's one line of math, it works on any subnet, and it's a nice thing to have in your back pocket when you're working through a bunch of these. I've been calling it the **Subnet Modulus Formula**, or SMF for short.

It's not a replacement for knowing how subnetting works under the hood. It's just another tool in the toolbag.

Let me walk you through it.

## The Problem

Here's a typical question:

- **IP address:** 10.23.179.73
- **CIDR:** /20 (mask: 255.255.240.0)

What's the network address?

The normal way is to figure out the block size in the third octet (it's 16), then count up: 0, 16, 32, 48, 64, 80, 96, 112, 128, 144, 160, 176, 192… and figure out which block 179 falls into. It works fine. It's just a little tedious, and it's easy to lose your spot when you're under pressure on an exam.

## The Shortcut

Here's the whole formula:

```
Network octet = IP octet − (IP octet mod Block Size)
```

Where **Block Size = 256 − the mask value in that octet**. The block size is the spacing between subnet boundaries in that octet. For example, with a block size of 16, subnets start at 0, 16, 32, 48, and so on.

**Quick refresher on "mod" if you're rusty:** modulus is the remainder you get after dividing. So `179 mod 16` means "divide 179 by 16 and tell me what's left over." 16 goes into 179 eleven times (16 × 11 = 176), and you've got 3 left. So `179 mod 16 = 3`. It's just long division where you throw out the answer and keep the remainder.

Okay, let's run the formula on our example:

1. The third octet of the mask is 240, so the block size is `256 − 240 = 16`.
2. `179 mod 16 = 3` (because 16 × 11 = 176, and there's 3 left over).
3. `179 − 3 = 176`.

Network address: **10.23.176.0/20**.

## Why It Works

It's not magic. It's just using how subnets are built in the first place. Every subnet block size is a power of 2 (1, 2, 4, 8, 16, 32, 64, 128), and every subnet starts at a multiple of its block size.

If your block size is 16, then valid subnets in that octet can only start at 0, 16, 32, 48, and so on. Any IP sitting inside one of those blocks is just "the start of the block plus some leftover." Modulus gives you that leftover. Subtract it and you've snapped back to the start of the block.

That's the whole idea. Modulus tells you how far past the boundary you are. Subtract it, you're back on the boundary.

## The "Interesting Octet"

You only run the formula on one octet, and that's the first one in the mask that isn't 255. That's where the network and host portions split.

- For 255.255.240.0, the interesting octet is the third one (240).
- For 255.240.0.0, it's the second (240).
- For 255.255.255.224, it's the fourth (224).

Anything to the left of the interesting octet stays the same as the IP. Anything to the right becomes 0. The interesting octet is the only one that needs any math.

One edge case worth mentioning: classful CIDRs (/8, /16, /24, /32) don't need the formula at all, since they land right on octet boundaries. The network address is just the IP with the host octets zeroed out. The Subnet Modulus Formula really pays off on classless CIDRs that fall between octets, like /12, /19, /20, /28, and so on. Those are the ones that are a pain to count by hand.

## A Few More Examples

Let's see the formula running in different octets.

**Example 1 — Second octet (/12)**
- IP: 172.19.45.200
- Mask: 255.240.0.0
- Block size: 256 − 240 = 16
- 19 mod 16 = 3 → 19 − 3 = 16
- **Network: 172.16.0.0/12**

**Example 2 — Third octet (/19)**
- IP: 192.168.150.77
- Mask: 255.255.224.0
- Block size: 256 − 224 = 32
- 150 mod 32 = 22 → 150 − 22 = 128
- **Network: 192.168.128.0/19**

**Example 3 — Fourth octet (/28)**
- IP: 10.0.0.137
- Mask: 255.255.255.240
- Block size: 256 − 240 = 16
- 137 mod 16 = 9 → 137 − 9 = 128
- **Network: 10.0.0.128/28**

Same formula every time. The only thing that changes is which octet you're applying it to.

## What This Gets You (and What It Doesn't)

The Subnet Modulus Formula gives you the **network address**, which is the first IP in the subnet and the one that identifies the subnet itself. That's usually step one of any subnetting question, but it's not the whole picture. Once you've got the network address though, the rest is pretty easy.

Let's stick with our original example: 10.23.179.73 /20, network **10.23.176.0**, block size 16.

**Broadcast address.** This is the last IP in the subnet. Take the network octet, add the block size, subtract 1. Any octets to the right of the interesting octet get filled with 255 (since they're all host bits).

```
176 + 16 − 1 = 191  →  10.23.191.255
```

**First usable host.** The IP right after the network address.

```
10.23.176.0 + 1  →  10.23.176.1
```

**Last usable host.** The IP right before the broadcast.

```
10.23.191.255 − 1  →  10.23.191.254
```

**Next subnet.** Add the block size to the network octet.

```
176 + 16 = 192  →  10.23.192.0/20
```

## TL;DR

```
Block Size    = 256 − interesting mask octet
Network octet = IP octet − (IP octet mod Block Size)
```

Apply it to the interesting octet. Zero out everything to the right. Done.

Try it on a few IPs from your own network and see if it sticks. Worst case you've got another way to double-check your work.

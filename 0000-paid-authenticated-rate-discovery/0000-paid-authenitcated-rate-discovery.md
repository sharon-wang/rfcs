---
title: Paid Authenticated Rate Discovery (PARD)
draft: 1
---

# Paid Authenticated Rate Discovery (PARD)

## Preface

This document describes Paid Authenticated Rate Discovery, a protocol for
discovering asset prices without pulling rates from outside the Interledger
network.

## Introduction

### Motivation

Pricing is an important part of any paid application. Pricing is easy when you
have one currency, but it's harder when you have an unbounded number of
currencies. In Interledger, we cannot make any assumptions about the currency
that anyone is using.

This is solved in many use cases by having the receiver set a price. The sender
sees the price in their source units, and then decides whether to make the payment.

The case that PARD solves is when the receiver will accept any amount, and the
sender needs a reasonable default amount to send. Examples of this are [Web
Monetization](../0028-web-monetization/0028-web-monetization.md) extensions
that decide a rate per second, or autonomous programs which need to be deployed
any part of the network and decide what to spend and charge.

Pulling rates from an API introduces a single point of failure. You could set
up many redundant APIs, but this is hard to work with and may not reflect the
actual rates of the Interledger network. PARD works entirely within the
Interledger network.

### Scope

PARD is intended for end-user applications that need to price services and
fetch sensible defaults.

PARD returns a rate from your local units to the units of a remote destination,
but costs a small amount of money to prevent DoS attacks.

### Operation

Any participant of the Interledger network can run a PARD landmark. A PARD
landmark does not need to be connected to the internet. They can then share
this landmark with others by publishing an ILP address, public key, asset code,
and asset scale.

PARD clients fetch rates by sending an Interledger packet to a PARD landmark.
The landmark fulfills the payment and returns the amount that got to them in
the fulfillment data.

### Definitions

- **PARD Packet** - The Interledger prepare packet containing PARD data.
- **PARD Response** - The authenticated response data returned in the fulfill of a PARD packet.
- **PARD Landmark** - The receiver of PARD packets who responds with rate information.
- **PARD Landmark Details** - The ILP address, public key, asset code, and asset scale used to reach a PARD Landmark.
- **PARD Client** - The sender of PARD packets who is paying for rates.

## Overview

### Relation to Other Protocols

### Model of Operation

## Specification

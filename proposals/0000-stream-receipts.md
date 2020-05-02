---
title: STREAM Receipts
type: proposal
draft: 1
---

# STREAM Receipts
> Proof provided by a [STREAM](../0029-stream/0029-stream.md) receiver of the total amount received on a stream.

## Motivation

Interledger payment verification is necessary for multiple use cases including Web Monetization, where ILP payments must be verified to securely serve exclusive content, and API micropayments.

Standardized receipt functionality at the STREAM layer enables the payment recipient or a third party to verify payment without having access to the corresponding ILP packets.

## Conventions and Definitions

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “NOT RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in BCP 14 [RFC2119](https://tools.ietf.org/html/rfc2119) [RFC8174](https://tools.ietf.org/html/rfc8174) when, and only when, they appear in all capitals, as shown here.

Definitions of terms that are used in this document:

- **Receiver** - The party that is receiving funds over STREAM
- **Verifier** - The party that wishes to verify that ILP payments occurred to the Receiver
- **Sender** - The party that is sending funds over STREAM
- **Receipt Nonce** - A unique nonce used to identify a STREAM connection in a Receipt
- **Receipt Secret** - The key used to generate a Receipt's HMAC
- **Receipt** - A binary object containing a version number, the Receipt Nonce, a Stream ID on the STREAM connection, an integer amount of funds received on a stream, and an HMAC over all those fields

## Overview

In a STREAM connection, a Receipt is generated by the Receiver for every fulfilled ILP packet. Each Receipt on a stream contains a progressively higher amount, representing the total amount received on that stream. So the latest Receipt on a stream replaces any previous Receipts on that stream.

The Verifier SHOULD consider a Receipt valid if:

- The Receipt contents produce an HMAC (using the Receipt Secret as the seed) identical to the Receipt's HMAC field.
- The amount is greater than any other Receipts with that Receipt Nonce and Stream ID.
- The Receipt is not stale.

The Verifier SHOULD determine a length of time starting from when a Receipt Nonce is generated that corresponding Receipts will be considered valid. Mechanisms by which the Verifier can check Receipts' staleness include, but are not limited to:
- persistently storing the Receipt Nonce and a timestamp relative to when the Nonce was generated
- encoding a timestamp in the Receipt Nonce

If keeping a balance of how much the Sender has paid, the Verifier SHOULD store the Receipt amount under the Receipt Nonce + Stream ID. If there is already an amount stored for that Stream ID, the Verifier SHOULD only credit the Sender the difference in Receipt amounts.

Alteratively, the Verifier MAY forego checking the Receipt amount if they are only concerned with whether or not the Sender paid at all, as opposed to how much.

### Flow

This flow occurs for each STREAM connection with Receipts enabled.

1. Verifier MUST generate a unique Receipt Nonce.

2. Verifier MUST communicate the Receipt Nonce and Receipt Secret to the Receiver. The Receipt Secret MUST NOT be communicated to the Sender.

3. Once the STREAM connection is open and the Sender is sending, the Receiver MUST add an additional frame containing a Receipt to each ILP Fulfill packet.

4. Sender MUST give the Receipt directly or indirectly to the Verifier.

5. Verifier MUST verify the Receipt before accepting the Receipt amount as paid.

### Stateless Verifier

The Verifier MAY generate the Receipt Secret by producing an HMAC of the Receipt Nonce using a seed (known only to itself) as the HMAC key. This ensures the Receipt Secret will be unique to this STREAM connection and allows the Verifier to reproduce the Receipt Secret from any Receipts containing the Receipt Nonce.

### Stateless Receiver

Before a STREAM connection is established, the Receiver MAY use the Receipt Nonce and Receipt Secret as parameters to generate connection details (ILP Address and shared secret). The Receiver MAY encode the Receipt Nonce and Receipt Secret in the STREAM server ILP Address to allow the STREAM receiver to avoid persisting state. If doing so, the Receipt Secret MUST be encrypted. The Receipt Nonce MUST be unique each time STREAM parameters are generated, since it MUST be unique globally.

### SPSP

The Verifier MAY communicate the Receipt Nonce and Receipt Secret to the Receiver by proxying the Sender's [SPSP](../0009-simple-payment-setup-protocol/0009-simple-payment-setup-protocol.md) query, as follows:

1. Sender queries the Verifier's payment pointer URL.
2. Verifier MUST add the Receipt Nonce and Receipt Secret to the request in `Receipt-Nonce` and `Receipt-Secret` headers.
3. Verifier MUST pass the query to the Receiver.
4. Receiver MUST include `"receipts_enabled": true` in a successful SPSP response.
5. Verifier MAY return a `409` error code if the response does not include `"receipts_enabled": true`. Otherwise, the Verifier MUST return the SPSP response to the Sender.

### Web Monetization

A [Web Monetization](https://webmonetization.org/specification.html) Sender MUST include each ILP Fulfill packet's Receipt in the corresponding `monetizationprogress` event. This allows the webpage to pass Receipts to the Verifier.

## Specification

A Receipt MUST contain the following fields encoded using [CANONICAL-OER](https://github.com/interledger/rfcs/blob/master/0030-notes-on-oer-encoding/0030-notes-on-oer-encoding.md#canonical-oer) (find more details [here](https://github.com/interledger/rfcs/blob/05ab457b9301b031e1ec954632582a325c4907b4/asn1/README.md)):

| Field | Type | Description |
|---|---|---|
| Version | UInt8 | `1` for this version. |
| Receipt Nonce | UInt128 | A unique nonce pre-shared between the Verifier and the Receiver used to identify the STREAM connection. |
| Stream ID | UInt8 | Identifier of the stream this Receipt refers to. |
| Total Received | UInt64 | Total amount, denominated in the units of the Receiver, that the Receiver has received on this stream thus far. |
| HMAC | UInt256 | HMAC-SHA256 using the 32 byte Receipt Secret, which is pre-shared between the Verifier and the Receiver. The HMAC message is the concatenation of all other Receipt fields, in the order listed above. |
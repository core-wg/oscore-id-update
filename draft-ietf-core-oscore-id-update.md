---
v: 3

title: Identifier Update for OSCORE
abbrev: Identifier Update for OSCORE
docname: draft-ietf-core-oscore-id-update-latest

# stand_alone: true

ipr: trust200902
wg: CoRE Working Group
kw: Internet-Draft
cat: std
submissiontype: IETF

venue:
  group: Constrained RESTful Environments (core)
  mail: core@ietf.org
  github: core-wg/oscore-id-update

coding: utf-8

author:
      -
        ins: R. Höglund
        name: Rikard Höglund
        org: RISE AB
        street: Isafjordsgatan 22
        city: Kista
        code: SE-16440 Stockholm
        country: Sweden
        email: rikard.hoglund@ri.se
      -
        ins: M. Tiloca
        name: Marco Tiloca
        org: RISE AB
        street: Isafjordsgatan 22
        city: Kista
        code: SE-16440 Stockholm
        country: Sweden
        email: marco.tiloca@ri.se

normative:
  RFC2119:
  RFC7252:
  RFC7641:
  RFC8174:
  RFC8613:
  RFC8949:
  I-D.ietf-core-oscore-key-update:

informative:
  RFC9528:

entity:
  SELF: "[RFC-XXXX]"

--- abstract

Two peers that communicate with the CoAP protocol can use the Object Security for Constrained RESTful Environments (OSCORE) protocol to protect their message exchanges end-to-end. To this end, the two peers share an OSCORE Security Context and a number of related identifiers. In particular, each of the two peers stores a Sender ID that identifies its own Sender Context within the Security Context, and a Recipient ID that identifies the Recipient Context associated with the other peer within the same Security Context. These identifiers are sent in plaintext within OSCORE-protected messages. Hence, they can be used to correlate messages exchanged between peers and track those peers, with consequent privacy implications. This document defines an OSCORE ID update procedure that two peers can use to update their OSCORE identifiers. This procedure can be run stand-alone or seamlessly integrated in an execution of the Key Update for OSCORE (KUDOS) procedure.

--- middle

# Introduction

When using the CoAP protocol {{RFC7252}}, two peers can use the Object Security for Constrained RESTful Environments (OSCORE) protocol to protect their message exchanges end-to-end. To this end, the two peers share an OSCORE Security Context and a number of related identifiers.

As part of the shared Security Context, each peer stores one Sender Context identified by a Sender ID and used to protect its outgoing messages. Also, it stores a Recipient Context identified by a Recipient ID and used to unprotect the incoming messages from the other peer. That is, one's peer Sender ID (Recipient ID) is equal to the other peer's Recipient ID (Sender ID).

When receiving an OSCORE-protected message, the recipient peer uses the Recipient ID conveyed within the message or otherwise implied, in order to retrieve the correct Security Context and unprotect the message.

These identifiers are sent in plaintext within OSCORE-protected messages and are immutable throughout the lifetime of a Security Context, even in case the two peers migrate to a different network or simply change their addressing information. Therefore, the identifiers can be used to correlate messages that the two peers exchange at different points in time or through different paths, hence allowing to track them with the consequent privacy implications.

In order to address this issue, this document defines an OSCORE ID update procedure that two peers can use to update their OSCORE Sender and Recipient IDs. For instance, two peers may want to use this procedure before switching to a different network, in order to make it more difficult to understand that their communication is continuing in the new network.

The OSCORE ID update procedure can be run stand-alone or seamlessly integrated in an execution of the Key Update for OSCORE (KUDOS) procedure {{I-D.ietf-core-oscore-key-update}}.

## Terminology ## {#terminology}

{::boilerplate bcp14-tagged}

Readers are expected to be familiar with the terms and concepts related to CoAP {{RFC7252}}, Observe {{RFC7641}}, CBOR {{RFC8949}}, OSCORE {{RFC8613}}, and KUDOS {{I-D.ietf-core-oscore-key-update}}.

This document additionally uses the following terminology.

* Initiator: the peer starting the OSCORE ID update procedure, by sending the first message.

* Responder: the peer that receives the first message in an execution of the OSCORE ID update procedure.

* Forward message flow: the execution workflow where the initiator acts as CoAP client (see {{example-client-initiated-id-update}}).

* Reverse message flow: the execution workflow where the initiator acts as CoAP server (see {{example-server-initiated-id-update}}).

# Update of OSCORE Sender/Recipient IDs # {#update-oscore-ids}

This section defines the procedure that two peers can perform, in order to update the OSCORE Sender/Recipient IDs that they use in their shared OSCORE Security Context.

When performing an update of OSCORE Sender/Recipient IDs, a peer provides its new intended OSCORE Recipient ID to the other peer, by means of the Recipient-ID Option defined in {{sec-recipient-id-option}}. Hereafter, this document refers to a message including the Recipient-ID Option as an "ID update (request/response) message".

This procedure can be initiated by either peer, i.e., the CoAP client or the CoAP server may start it by sending the first OSCORE ID update message. The former case is denoted as the "forward message flow" and the latter as the "reverse message flow".

Furthermore, this procedure can be executed stand-alone, or instead seamlessly integrated in an execution of the KUDOS procedure for updating OSCORE keying material used in its FS mode (see {{Section 4 of I-D.ietf-core-oscore-key-update}}) or no-FS mode (see {{Section 4.5 of I-D.ietf-core-oscore-key-update}}).

* In the former stand-alone case, updating the OSCORE Sender/Recipient IDs effectively results in updating part of the current OSCORE Security Context.

   That is, both peers derive a new Sender Key, Recipient Key, and Common IV, as defined in {{Section 3.2 of RFC8613}}. Also, both peers re-initialize the Sender Sequence Number and the Replay Window accordingly, as defined in {{Section 3.2.2 of RFC8613}}. Since the same Master Secret is preserved, forward secrecy is not achieved.

   As defined in {{id-update-additional-actions}}, the two peers must take additional actions to ensure a safe execution of the OSCORE ID update procedure.

   A peer can safely discard the old OSCORE Security Context including the old OSCORE Sender/Recipient IDs after the following two events have occurred, in this order: first, the peer has sent to the other peer a message protected with the new OSCORE Security Context including the new OSCORE Sender/Recipient IDs; then, the peer has received from the other peer and successfully verified a message protected with that new OSCORE Security Context.

* In the latter integrated case, the KUDOS initiator (responder) also acts as initiator (responder) for the OSCORE ID update procedure. That is, both KUDOS and the OSCORE ID update procedure MUST be run either in their forward message flow or in their reverse message flow.

   The new OSCORE Sender/Recipient IDs MUST NOT be used with the OSCORE Security Context CTX\_OLD, and MUST NOT be used with the temporary OSCORE Security Context CTX\_1 used to protect the first KUDOS message of a KUDOS execution.

   The first use of the new OSCORE Sender/Recipient IDs with the new OSCORE Security Context CTX\_NEW occurs: for the KUDOS initiator, after having received from the KUDOS responder and successfully verified the second KUDOS message of the KUDOS execution in question; for the KUDOS responder, after having sent to the KUDOS initiator the second KUDOS message of the KUDOS execution in question.

A peer terminates an ongoing OSCORE ID update procedure with another peer as successful, in any of the following two cases.

* The peer is acting as initiator, and it has received and successfully verified the second ID update message from the other peer.

* The peer is acting as responder, and it has sent the second ID update message to the other peer.

A peer MUST NOT initiate an OSCORE ID update procedure with another peer, if it has another such procedure ongoing with that other peer.

Upon receiving a valid, first ID update message, a responder MUST send the second ID update message, except in the case any of the conditions for failing or aborting the procedure apply (see {{update-failure}}}).

## Failure of the ID Update Procedure {#update-failure}

The following section describes cases where the OSCORE ID update procedure fails, or must to be aborted by one of the peers.

Upon receiving a valid first ID update message, a responder MUST abort the ID update procedure, in the following case:

* The received ID update message is not a KUDOS message (i.e., the OSCORE ID update procedure is being performed stand-alone) and the responder has no eligible Recipient ID to offer to the initiator (see {{id-update-additional-actions}}).

Upon receiving a valid ID update message, a peer MUST abort the ID update procedure, in the following case:

* The received ID update message contains a Recipient-ID option with a length that exceeds the maximum length of OSCORE Sender/Recipient IDs for the AEAD algorithm in use for the OSCORE Security Context shared between the peers. This is the case when the length of the Recipient-ID option exceeds the length of the AEAD nonce minus 6 (see {{Section 3.3 of RFC8613}}).

If, after receiving an ID update message as CoAP request, a peer aborts the ID update procedure, the peer MUST also reply to the received ID update request message with a protected 5.03 (Service Unavailable) error response. The error response MUST NOT include the Recipient-ID Option, and its diagnostic payload MAY provide additional information. When receiving the error response, the initiator terminates the OSCORE IDs procedure as failed.

An initiator terminates an ongoing OSCORE ID update procedure with another peer as failed, in case, after having sent the first ID update message for the procedure in question, a pre-defined amount of time has elapsed without receiving and successfully verifying the second ID update message from the other peer. It is RECOMMENDED that such an amount of time is equal to MAX\_TRANSMIT\_WAIT (see {{Section 4.8.2 of RFC7252}}).

When the OSCORE ID update procedure is integrated into the execution of the KUDOS procedure, it is possible that the KUDOS procedure succeeds while the OSCORE ID update procedure fails. In such case, the peers continue their communications using the newly derived OSCORE Security Context CTX\_NEW obtained from the KUDOS procedure, and still use the old Sender and Recipient IDs. That is, any Recipient IDs conveyed in the exchanged Recipient-ID Options is not considered.

Conversely, the OSCORE ID update procedure may succeed while the KUDOS procedure fails. As long as the peers have exchanged a pair of OSCORE-protected request and response that conveyed their desired new Recipient IDs in the Recipient-ID Option, the peers start using those IDs in their communications.

## The Recipient-ID Option # {#sec-recipient-id-option}

The Recipient ID-Option defined in this section has the properties summarized in {{table-recipient-id-option}}, which extends Table 4 of {{RFC7252}}. That is, the option is elective, safe to forward, part of the cache key, and not repeatable.

| No.   | C | U | N | R | Name         | Format | Length | Default |
| TBD24 |   |   |   |   | Recipient-ID | opaque | any    | (none)  |
{: #table-recipient-id-option title="The&nbsp;Recipient-ID&nbsp;Option.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
C=Critical, U=Unsafe, N=NoCacheKey, R=Repeatable" align="center"}

Note to RFC Editor: Following the registration of the CoAP Option Number 24, please replace "TBD24" with "24" in the figure above. Then, please delete this paragraph.

The option value can have an arbitrary length, including zero length to indicate intent to use the empty string as Recipient ID.

This document particularly defines how this option is used in messages protected with OSCORE. That is, when the option is included in an outgoing message, the option value specifies the new OSCORE Recipient ID that the sender endpoint intends to use with the other endpoint sharing the OSCORE Security Context.

Therefore, the maximum length of the option value is equal to the maximum length of OSCORE Sender/Recipient IDs. As defined in {{Section 3.3 of RFC8613}}, this is determined by the size of the AEAD nonce of the used AEAD Algorithm in the OSCORE Security Context.

If the length of the Recipient ID included in the Recipient-ID option is zero, the option value SHALL be empty (Option Length = 0).

The Recipient-ID Option is of class E in terms of OSCORE processing (see {{Section 4.1 of RFC8613}}).

### Forward Message Flow {#example-client-initiated-id-update}

{{fig-id-update-client-init}} shows an example of the OSCORE ID update procedure, run stand-alone and in the forward message flow, with the client acting as initiator. On each peer, SID and RID denote the OSCORE Sender ID and Recipient ID of that peer, respectively.

{{sec-id-update-in-kudos-forward}} provides a different example of the OSCORE ID update procedure, as run integrated in an execution of KUDOS and in the forward message flow (see {{Section 4.3.5 of I-D.ietf-core-oscore-key-update}}).

~~~~~~~~~~~ aasvg
          Client                             Server
       (initiator)                         (responder)
            |                                   |
CTX_A {     |                                   | CTX_A {
 SID = 0x01 |                                   |  SID = 0x00
 RID = 0x00 |                                   |  RID = 0x01
}           |                                   | }
            |                                   |
            |            Request #1             |
Protect     |---------------------------------->| /temp
with CTX_A  | OSCORE {                          |
            |  ...                              |
            |  kid: 0x01                        | Verify
            | }                                 | with CTX_A
            | Encrypted Payload {               |
            |  ...                              |
            |  Recipient-ID: 0x42               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
            |            Response #1            |
            |<----------------------------------| Protect
            | OSCORE {                          | with CTX_A
            |  ...                              |
Verify      | }                                 |
with CTX_A  | Encrypted Payload {               |
            |  ...                              |
            |  Recipient-ID: 0x78               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
CTX_B {     |                                   | CTX_B {
 SID = 0x78 |                                   |  SID = 0x42
 RID = 0x42 |                                   |  RID = 0x78
}           |                                   | }
            |                                   |
            |            Request #2             |
Protect     |---------------------------------->| /temp
with CTX_B  | OSCORE {                          |
            |  ...                              |
            |  kid: 0x78                        | Verify
            | }                                 | with CTX_B
            | Encrypted Payload {               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
            |            Response #2            |
            |<----------------------------------| Protect
            | OSCORE {                          | with CTX_B
            |  ...                              |
Verify      | }                                 |
with CTX_B  | Encrypted Payload {               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
Discard     |                                   |
CTX_A       |                                   |
            |                                   |
            |            Request #3             |
Protect     |---------------------------------->| /temp
with CTX_B  | OSCORE {                          |
            |  ...                              |
            |  kid: 0x78                        | Verify
            | }                                 | with CTX_B
            | Encrypted Payload {               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
            |                                   | Discard
            |                                   | CTX_A
            |                                   |
            |            Response #3            |
            |<----------------------------------| Protect
            | OSCORE {                          | with CTX_B
            |  ...                              |
Verify      | }                                 |
with CTX_B  | Encrypted Payload {               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
~~~~~~~~~~~
{: #fig-id-update-client-init title="Example of the OSCORE ID update procedure with Forward Message Flow" artwork-align="center"}

Before the OSCORE ID update procedure starts, the client (the server) shares with the server (the client) an OSCORE Security Context CTX\_A with Sender ID 0x01 (0x00) and Recipient ID 0x00 (0x01).

When starting the OSCORE ID update procedure, the client determines its new intended OSCORE Recipient ID 0x42. Then, the client prepares a CoAP request targeting an application resource at the server. The request includes the Recipient-ID Option, with value the client's new Recipient ID 0x42.

The client protects the request with CTX\_A, i.e., by using the keying material derived from the client's current Sender ID 0x01. The protected request specifies the client's current Sender ID 0x01 in the 'kid' field of the OSCORE Option. After that, the client sends the request to the server as Request \#1.

Upon receiving, decrypting, and successfully verifying the OSCORE message Request \#1, the server retrieves the value 0x42 from the Recipient-ID Option, and determines its new intended OSCORE Recipient ID 0x78. Then, the server prepares a CoAP response including the Recipient-ID Option, with value the server's new Recipient ID 0x78.

The server protects the response with CTX\_A, i.e., by using the keying material derived from the server's current Sender ID 0x00. After that, the server sends the response to the client.

Then, the server considers 0x42 and 0x78 as its new Sender ID and Recipient ID to use with the client, respectively. As shown in the example, the server practically installs a new OSCORE Security Context CTX\_B where: i) its Sender ID and Recipient ID are 0x42 and 0x78, respectively; ii) the Sender Sequence Number and the Replay Window are re-initialized (see {{Section 3.2.2 of RFC8613}}); iii) anything else is like in the OSCORE Security Context used to encrypt the OSCORE message Response \#1.

Upon receiving, decrypting, and successfully verifying the OSCORE message Response \#1, the client considers 0x78 and 0x42 as the new Sender ID and Recipient ID to use with the server, respectively. As shown in the example, the client practically installs a new OSCORE Security Context CTX\_B where: i) its Sender ID and Recipient ID are 0x78 and 0x42, respectively; ii) the Sender Sequence Number and the Replay Window are re-initialized (see {{Section 3.2.2 of RFC8613}}); iii) anything else is like in the OSCORE Security Context used to decrypt the OSCORE message Response \#1.

From then on, the client and the server can exchange messages protected with the OSCORE Security Context CTX\_B, i.e., according to the new OSCORE Sender/Recipient IDs and using new keying material derived from those.

That is, the client sends the OSCORE message Request \#2, which is protected with CTX\_B and specifies the new client's Sender ID 0x78 in the 'kid' field of the OSCORE Option.

Upon receiving the OSCORE message Request \#2, the server retrieves the OSCORE Security Context CTX\_B, according to its new Recipient ID 0x78 specified in the 'kid' field of the OSCORE Option. Then, the server decrypts and verifies the response by using CTX\_B. Finally, the server prepares a CoAP response Response \#2, protects it with CTX\_B, and sends it to the client.

Upon receiving the OSCORE message Response \#2, the client decrypts and verifies it with the OSCORE Security Context CTX\_B. In case of successful verification, the client confirms that the server is aligned with the new OSCORE Sender/Recipient IDs, and thus discards the OSCORE Security Context CTX\_A.

After that, one further exchange occurs, where both the CoAP request and the CoAP response are protected with the OSCORE Security Context CTX\_B. In particular, upon receiving, decrypting, and successfully verifying the OSCORE message Request \#3, the server confirms that the client is aligned with the new OSCORE Sender/Recipient IDs, and thus discards the OSCORE Security Context CTX\_A.

### Reverse Message Flow {#example-server-initiated-id-update}

{{fig-id-update-server-init}} shows an example of the OSCORE ID update procedure, run stand-alone and in the reverse message flow, with the server acting as initiator. On each peer, SID and RID denote the OSCORE Sender ID and Recipient ID of that peer, respectively.

{{sec-id-update-in-kudos-reverse}} provides a different example of the OSCORE ID update procedure, as run integrated in an execution of KUDOS and in the reverse message flow (see {{Section 4.3.6 of I-D.ietf-core-oscore-key-update}}).

~~~~~~~~~~~ aasvg
          Client                             Server
       (responder)                         (initiator)
            |                                   |
CTX_A {     |                                   | CTX_A {
 SID = 0x01 |                                   |  SID = 0x00
 RID = 0x00 |                                   |  RID = 0x01
}           |                                   | }
            |                                   |
            |            Request #1             |
Protect     |---------------------------------->| /temp
with CTX_A  | OSCORE {                          |
            |  ...                              |
            |  kid: 0x01                        | Verify
            | }                                 | with CTX_A
            | Encrypted Payload {               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
            |            Response #1            |
            |<----------------------------------| Protect
            | OSCORE {                          | with CTX_A
            |  ...                              |
Verify      | }                                 |
with CTX_A  | Encrypted Payload {               |
            |  ...                              |
            |  Recipient-ID: 0x78               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
            |            Request #2             |
Protect     |---------------------------------->| /temp
with CTX_A  | OSCORE {                          |
            |  ...                              |
            |  kid: 0x01                        | Verify
            | }                                 | with CTX_A
            | Encrypted Payload {               |
            |  ...                              |
            |  Recipient-ID: 0x42               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
            |                                   | CTX_B {
            |                                   |  SID = 0x42
            |                                   |  RID = 0x78
            |                                   | }
            |                                   |
            |            Response #2            |
            |<----------------------------------| Protect
            | OSCORE {                          | with CTX_A
            |  ...                              |
Verify      | }                                 |
with CTX_A  | Encrypted Payload {               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
CTX_B {     |                                   |
 SID = 0x78 |                                   |
 RID = 0x42 |                                   |
}           |                                   |
            |                                   |
            |            Request #3             |
Protect     |---------------------------------->| /temp
with CTX_B  | OSCORE {                          |
            |  ...                              |
            |  kid: 0x78                        | Verify
            | }                                 | with CTX_B
            | Encrypted Payload {               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
            |            Response #3            |
            |<----------------------------------| Protect
            | OSCORE {                          | with CTX_B
            |  ...                              |
Verify      | }                                 |
with CTX_B  | Encrypted Payload {               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
Discard     |                                   |
CTX_A       |                                   |
            |                                   |
            |            Request #4             |
Protect     |---------------------------------->| /temp
with CTX_B  | OSCORE {                          |
            |  ...                              |
            |  kid: 0x78                        | Verify
            | }                                 | with CTX_B
            | Encrypted Payload {               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
            |                                   | Discard
            |                                   | CTX_A
            |                                   |
            |            Response #4            |
            |<----------------------------------| Protect
            | OSCORE {                          | with CTX_B
            |  ...                              |
Verify      | }                                 |
with CTX_B  | Encrypted Payload {               |
            |  ...                              |
            |  Application Payload              |
            | }                                 |
            |                                   |
~~~~~~~~~~~
{: #fig-id-update-server-init title="Example of the OSCORE ID update procedure with Reverse Message Flow" artwork-align="center"}

Before the OSCORE ID update procedure starts, the client (the server) shares with the server (the client) an OSCORE Security Context CTX\_A with Sender ID 0x01 (0x00) and Recipient ID 0x00 (0x01).

At first, the client prepares a CoAP Request \#1 targeting an application resource at the server. The client protects the request with CTX\_A, i.e., by using the keying material derived from the client's current Sender ID 0x01. The protected request specifies the client's current Sender ID 0x01 in the 'kid' field of the OSCORE Option. After that, the client sends the request to the server as Request \#1.

Upon receiving, decrypting, and successfully verifying the OSCORE message Request \#1, the server decides to start an OSCORE ID update procedure. To this end, the server determines its new intended OSCORE Recipient ID 0x78. Then, the server prepares a CoAP response as a reply to the just received request and including the Recipient-ID Option, with value the server's new Recipient ID 0x78.

The server protects the response with CTX\_A, i.e., by using the keying material derived from the server's current Sender ID 0x00. After that, the server sends the response to the client as Response \#1.

Upon receiving, decrypting, and successfully verifying the OSCORE message Response \#1, the client retrieves the value 0x78 from the Recipient-ID Option, and determines its new intended OSCORE Recipient ID 0x42. Then, the client prepares a CoAP request targeting an application resource at the server. The request includes the Recipient-ID Option, with value the client's new Recipient ID 0x42.

The client protects the request with CTX\_A, i.e., by using the keying material derived from the client's current Sender ID 0x01. The protected request specifies the client's current Sender ID 0x01 in the 'kid' field of the OSCORE Option. After that, the client sends the request to the server as Request \#2.

Upon receiving, decrypting, and successfully verifying the OSCORE message Request \#2, the server retrieves the value 0x42 from the Recipient-ID Option. Then the server considers 0x42 and 0x78 as the new Sender ID and Recipient ID to use with the client, respectively. As shown in the example, the server practically installs a new OSCORE Security Context CTX\_B where: i) its Sender ID and Recipient ID are 0x42 and 0x78, respectively; ii) the Sender Sequence Number and the Replay Window are re-initialized (see {{Section 3.2.2 of RFC8613}}); iii) anything else is like in the OSCORE Security Context used to encrypt the OSCORE message Request \#2.

Then, the server prepares a CoAP response, as a reply to the just received request, and protects it with CTX\_A, i.e., by using the keying material derived from the server's current Sender ID 0x00. After that, the server sends the response to the client as Response \#2.

Upon receiving, decrypting, and successfully verifying the OSCORE message Response \#2, the client considers 0x78 and 0x42 as the new Sender ID and Recipient ID to use with the server, respectively. As shown in the example, the client practically installs a new OSCORE Security Context CTX\_B where: i) its Sender ID and Recipient ID are 0x78 and 0x42, respectively; ii) the Sender Sequence Number and the Replay Window are re-initialized (see {{Section 3.2.2 of RFC8613}}); iii) anything else is like in the OSCORE Security Context used to decrypt the OSCORE response.

From then on, the client and the server can exchange messages protected with the OSCORE Security Context CTX\_B, i.e., according to the new OSCORE Sender/Recipient IDs and using new keying material derived from those.

That is, the client sends the OSCORE message Request \#3, which is protected with CTX\_B and specifies the new client's Sender ID 0x78 in the 'kid' field of the OSCORE Option.

Upon receiving the OSCORE message Request \#3, the server retrieves the OSCORE Security Context CTX\_B, according to its new Recipient ID 0x78 specified in the 'kid' field of the OSCORE Option. Then, the server decrypts and verifies the response by using CTX\_B. Finally, the server prepares a CoAP response, protects it with CTX\_B, and sends it to the client as Response \#3.

Upon receiving the OSCORE message Response \#3, the client decrypts and verifies it with the OSCORE Security Context CTX\_B. In case of successful verification, the client confirms that the server is aligned with the new OSCORE Sender/Recipient IDs, and thus discards the OSCORE Security Context CTX\_A.

After that, one further exchange occurs, where both the CoAP request and the CoAP response are protected with the OSCORE Security Context CTX\_B. In particular, upon receiving, decrypting, and successfully verifying the OSCORE message Request \#4, the server confirms that the client is aligned with the new OSCORE Sender/Recipient IDs, and thus discards the OSCORE Security Context CTX\_A.

### Determining New OSCORE Identifiers Ahead of Network Migration {#new-identifiers-before-migration}

Peers may use the OSCORE ID update procedure to determine new OSCORE IDs in advance of a network path change. However, peers must not begin using these new identifiers on the current path prior to switching to a new network. Using a new identifier on the old path would allow observers to correlate activity across paths, defeating the unlinkability that updating the OSCORE IDs is intended to provide. To be effective, new identifiers must only be used for sending packets once the network migration is complete. Provisioning new OSCORE IDs ahead of time ensures that migration can proceed without delay, but care must be taken to ensure that premature use does not enable linkability.

To accomplish this, the peers execute the ID update procedure as normal (in the forward or reverse message flow), with the following difference: The peers must not begin using the OSCORE Security Context CTX\_B until after the network migration has taken place. Thus, both peers will be in a position to derive CTX\_B, but will not transition to use it until the first request protected with CTX\_B is transmitted, which must take place after network migration.

### Additional Actions for Execution {#id-update-additional-actions}

After having experienced a loss of state, a peer MUST NOT participate in a stand-alone OSCORE ID update procedure with another peer, until having performed a full-fledged establishment/renewal of an OSCORE Security Context with the other peer (e.g., by running KUDOS {{I-D.ietf-core-oscore-key-update}} or the authenticated key establishment protocol EDHOC {{RFC9528}}).

More precisely, a peer has experienced a loss of state if it cannot access the latest snapshot of the latest OSCORE Security Context CTX\_OLD or the whole set of OSCORE Sender/Recipient IDs that have been used with the triplet (Master Secret, Master Salt, ID Context) of CTX\_OLD. This can happen, for instance, after a device reboots.

Furthermore, when participating in a stand-alone OSCORE ID update procedure, a peer performs the following additional steps.

* When a peer sends an ID update message, the value of the Recipient-ID Option that the peer specifies as its new intended OSCORE Recipient ID MUST fulfill both the following conditions: it is currently available as Recipient ID to use for the peer (see {{Section 3.3 of RFC8613}}); and the peer has never used it as Recipient ID with the current triplet (Master Secret, Master Salt, ID Context).

* When receiving an ID update message, the peer MUST abort the procedure if it has already used the identifier specified in the Recipient-ID Option as its own Sender ID with the current triplet (Master Secret, Master Salt, ID Context).

In order to fulfill the conditions above, a peer has to keep track of the OSCORE Sender/Recipient IDs that it has used with the current triplet (Master Secret, Master Salt, ID Context) since the latest update of the OSCORE Master Secret (e.g., performed by running KUDOS).

## Preserving Observations Across ID Updates

When running the OSCORE ID update procedure stand-alone or integrated in an execution of KUDOS, the following holds if Observe {{RFC7641}} is supported, in order to preserve ongoing observations beyond a change of OSCORE identifiers.

* If a peer intends to keep active beyond an update of its Sender ID the observations where it is acting as CoAP client, then the peer:

   - MUST store the value of the 'kid' parameter from the original Observe requests, and retain it for the whole duration of the observations, throughout which the client MUST NOT update the stored value associated with the corresponding Observe registration request; and

   - MUST use the stored value of the 'kid' parameter from the original Observe registration request as value for the 'request_kid' parameter in the external_aad structure (see {{Section 5.4 of RFC8613}}), when verifying notifications for that observation as per {{Section 8.4.2 of RFC8613}}.

* If a peer is acting as CoAP server in an ongoing observation, then the peer:

   - MUST store the value of the 'kid' parameter from the original Observe registration request, and retain it for the whole duration of the observation, throughout which the peer MUST NOT update the stored value associated with the corresponding Observe registration request; and

   - MUST use the stored value of the 'kid' parameter from the original Observe registration request as value for the 'request_kid' parameter in the external_aad structure (see {{Section 5.4 of RFC8613}}), when protecting notifications for that observation as per {{Section 8.3.1 of RFC8613}}.

# Security Considerations

The OSCORE ID update procedure is a mechanism to mitigate the risk of tracking by on-path adversaries. By enabling endpoints to update their identifiers, either in response to specific events or on a regular basis, this approach prevents correlations that could otherwise be drawn between OSCORE messages on different network paths. While the ID update procedure is effective in reducing linkability across paths, it is best used in together with encryption of the OSCORE Partial IV.

The encryption of the Partial IV defends against correlation of messages based on sequence number patterns, which can still occur even when identifiers are updated. This combined approach significantly improves unlinkability of messages, making it more difficult for adversaries to infer relationships between them or to track communicating parties over time and across paths.

Usage of the OSCORE ID update procedure in isolation may not completely prevent adversaries from recognizing traffic patterns that reveal message ordering or frequency. In addition adversaries can infer identities of peers based on the sequence numbers (reset to zero or incrementing values), which may still allow for tracking peers across networks.

Note that while the OSCORE ID update procedure updates the OSCORE identifiers used, other information from the network communication may still be used to track the peers. Thus, it is recommended to combine usage of the OSCORE ID update procedure also with the following:

* Changing the network address (e.g., intentionally, or due to mobility, or NAT rebinding).
* Changing the link-layer address (e.g., MAC address randomization).

# IANA Considerations # {#sec-iana}

This document has the following actions for IANA.

Note to RFC Editor: Please replace all occurrences of "{{&SELF}}" with the RFC number of this specification and delete this paragraph.

## CoAP Option Numbers Registry ## {#iana-coap-options}

IANA is asked to enter the following option number to the "CoAP Option Numbers" registry within the "Constrained RESTful Environments (CoRE) Parameters" registry group.

| Number |     Name     | Reference  |
|--------|--------------|------------|
| TBD24  | Recipient-ID | {{&SELF}}  |
{: #tab-iana-recipient-id-option title="New CoAP Option Number" align="center"}

Note to RFC Editor: Following the registration of the CoAP Option Number 24, please replace "TBD24" with "24" in the table above. Then, please delete this paragraph.

--- back

# Examples of OSCORE ID update procedure Integrated in KUDOS # {#sec-id-update-in-kudos}

The following section shows two examples where the OSCORE ID update procedure is performed together with the KUDOS procedure for updating OSCORE keying material.

## Forward Message Flow # {#sec-id-update-in-kudos-forward}

{{fig-kudos-and-id-update-client-init}} provides an example of the OSCORE ID update procedure, as run integrated in an execution of KUDOS and in the forward message flow (see {{Section 4.3.5 of I-D.ietf-core-oscore-key-update}}). On each peer, SID and RID denote the OSCORE Sender ID and Recipient ID of that peer, respectively.

~~~~~~~~~~~ aasvg
                     Client                  Server
                   (initiator)            (responder)
                        |                      |
CTX_OLD {               |                      | CTX_OLD {
 SID = 0x01             |                      |  SID = 0x00
 RID = 0x00             |                      |  RID = 0x01
}                       |                      | }
                        |                      |
Generate N1             |                      |
                        |                      |
CTX_1 = updateCtx(      |                      |
        X1,             |                      |
        N1,             |                      |
        CTX_OLD )       |                      |
                        |                      |
                        |      Request #1      |
Protect with CTX_1      |--------------------->| /.well-known/kudos
                        | OSCORE {             |
                        |  ...                 |
                        |  d flag: 1           | CTX_1 = updateCtx(
                        |  x: X1               |         X1,
                        |  nonce: N1           |         N1,
                        |  ...                 |         CTX_OLD )
                        |  kid: 0x01           |
                        | }                    | Verify with CTX_1
                        | Encrypted Payload {  |
                        |  ...                 | Generate N2
                        |  Recipient-ID: 0x42  |
                        |  ...                 | CTX_NEW = updateCtx(
                        | }                    |           Comb(X1,X2),
                        |                      |           Comb(N1,N2),
                        |                      |           CTX_OLD )
                        |                      |
                        |      Response #1     |
                        |<---------------------| Protect with CTX_NEW
                        | OSCORE {             |
                        |  ...                 |
CTX_NEW = updateCtx(    |  Partial IV: 0       |
          Comb(X1,X2),  |  ...                 |
          Comb(N1,N2),  |                      |
          CTX_OLD )     |  d flag: 1           |
                        |  x: X2               |
Verify with CTX_NEW     |  nonce: N2           |
                        |  ...                 |
Discard CTX_OLD         | }                    |
                        | Encrypted Payload {  |
Update SID and          |  ...                 | Update SID and
RID in CTX_NEW          |  Recipient-ID: 0x78  | RID in CTX_NEW
                        |  ...                 |
CTX_NEW {               | }                    | CTX_NEW {
 SID = 0x78             |                      |  SID = 0x42
 RID = 0x42             |                      |  RID = 0x78
}                       |                      | }
                        |                      |

The actual key update process ends here.
The two peers can use the new Security Context CTX_NEW.

                        |                      |
                        |      Request #2      |
Protect with CTX_NEW    |--------------------->| /temp
                        | OSCORE {             |
                        |  ...                 |
                        |  kid: 0x78           | Verify with CTX_NEW
                        | }                    |
                        | Encrypted Payload {  | Discard CTX_OLD
                        |  ...                 |
                        |  Application Payload |
                        | }                    |
                        |                      |
                        |      Response #2     |
                        |<---------------------| Protect with CTX_NEW
                        | OSCORE {             |
                        |  ...                 |
Verify with CTX_NEW     | }                    |
                        | Encrypted Payload {  |
                        |  ...                 |
                        |  Application Payload |
                        | }                    |
                        |                      |
~~~~~~~~~~~
{: #fig-kudos-and-id-update-client-init title="Example of the OSCORE ID update procedure with Forward Message Flow and Integrated in a KUDOS Execution." artwork-align="center"}

## Reverse Message Flow # {#sec-id-update-in-kudos-reverse}

{{fig-kudos-and-id-update-server-init}} provides an example of the OSCORE ID update procedure, as run integrated in an execution of KUDOS and in the reverse message flow (see {{Section 4.3.6 of I-D.ietf-core-oscore-key-update}}). On each peer, SID and RID denote the OSCORE Sender ID and Recipient ID of that peer, respectively.

~~~~~~~~~~~ aasvg
                      Client                 Server
                   (responder)            (initiator)
                        |                      |
CTX_OLD {               |                      | CTX_OLD {
 SID = 0x01             |                      |  SID = 0x00
 RID = 0x00             |                      |  RID = 0x01
}                       |                      | }
                        |                      |
                        |      Request #1      |
Protect with CTX_OLD    |--------------------->| /temp
                        | OSCORE {             |
                        |  ...                 |
                        |  kid: 0x01           |
                        | }                    | Verify with CTX_OLD
                        | Encrypted Payload {  |
                        |  ...                 | Generate N1
                        |  Application Payload |
                        | }                    | CTX_1 = updateCtx(
                        |                      |         X1,
                        |                      |         N1,
                        |                      |         CTX_OLD )
                        |                      |
                        |      Response #1     |
                        |<---------------------| Protect with CTX_1
                        | OSCORE {             |
                        |  ...                 |
CTX_1 = updateCtx(      |  Partial IV: 0       |
        X1,             |  ...                 |
        N1,             |  d flag: 1           |
        CTX_OLD )       |  x: X1               |
                        |  nonce: N1           |
Verify with CTX_1       |  ...                 |
                        | }                    |
Generate N2             | Encrypted Payload {  |
                        |  ...                 |
CTX_NEW = updateCtx(    |  Recipient-ID: 0x78  |
          Comb(X1,X2),  |  ...                 |
          Comb(N1,N2),  | }                    |
          CTX_OLD )     |                      |
                        |                      |
                        |      Request #2      |
Protect with CTX_NEW    |--------------------->| /.well-known/kudos
                        | OSCORE {             |
                        |  ...                 |
                        |  d flag: 1           | CTX_NEW = updateCtx(
                        |  x: X2               |           Comb(X1,X2),
                        |  nonce: N2           |           Comb(N1,N2),
                        |  y: w                |           CTX_OLD )
                        |  old_nonce: N1       |
                        |  kid: 0x01           |
                        |  ...                 |
                        | }                    | Verify with CTX_NEW
                        | Encrypted Payload {  |
                        |  ...                 | Discard CTX_OLD
                        |  Recipient-ID: 0x42  |
                        |  ...                 |
                        |  Application Payload |
                        | }                    |
                        |                      |
Update SID and          |                      | Update SID and
RID in CTX_NEW          |                      | RID in CTX_NEW
                        |                      |
 CTX_NEW {              |                      | CTX_NEW {
  SID = 0x78            |                      |  SID = 0x42
  RID = 0x42            |                      |  RID = 0x78
 }                      |                      | }
                        |                      |

The actual key update process ends here.
The two peers can use the new Security Context CTX_NEW.

                        |                      |
                        |      Response #2     |
                        |<---------------------| Protect with CTX_NEW
                        | OSCORE {             |
                        |  ...                 |
Verify with CTX_NEW     | }                    |
                        | Encrypted Payload {  |
Discard CTX_OLD         |  ...                 |
                        |  Application Payload |
                        | }                    |
                        |                      |
                        |      Request #3      |
Protect with CTX_NEW    |--------------------->| /temp
                        | OSCORE {             |
                        |  ...                 |
                        |  kid: 0x78           | Verify with CTX_NEW
                        | }                    |
                        | Encrypted Payload {  |
                        |  ...                 |
                        |  Application Payload |
                        | }                    |
                        |                      |
                        |      Response #3     |
                        |<---------------------| Protect with CTX_NEW
                        | OSCORE {             |
                        |  ...                 |
Verify with CTX_NEW     | }                    |
                        | Encrypted Payload {  |
                        |  ...                 |
                        |  Application Payload |
                        | }                    |
                        |                      |
~~~~~~~~~~~
{: #fig-kudos-and-id-update-server-init title="Example of the OSCORE ID update procedure with Reverse Message Flow and Integrated in a KUDOS Execution." artwork-align="center"}

# Document Updates # {#sec-document-updates}
{:removeinrfc}

## Version -02 to -03 ## {#sec-02-03}

* Editorial improvements.

* Improved security considerations.

## Version -01 to -02 ## {#sec-01-02}

* Split long section into subsections.

* Updated references.

## Version -00 to -01 ## {#sec-00-01}

* Revised and extended error handling.

* Specify that the Recipient-ID option may need to be empty.

* Failure cases when running the ID update procedure integrated with KUDOS.

* Clarifications and editorial improvements.

## Version -00 ## {#sec-00}

* Split out material from Key Update for OSCORE draft into this new document.

* Extended terminology.

* Editorial improvements.

# Acknowledgments # {#acknowledgment}
{:numbered="false"}

The authors sincerely thank {{{Christian Amsüss}}}, {{{Carsten Bormann}}}, {{{John Preuß Mattsson}}}, and {{{Göran Selander}}} for their feedback and comments.

The work on this document has been partly supported by VINNOVA and the Celtic-Next projects CRITISEC and CYPRESS; and by the H2020 projects SIFIS-Home (Grant agreement 952652) and ARCADIAN-IoT (Grant agreement 101020259).

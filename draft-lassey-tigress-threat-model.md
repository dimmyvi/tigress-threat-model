---

title: "TIGRESS Threat Model"
category: info

docname: lassey-tigress-threat-model-latest
submissiontype: IET
number:
date:
consensus: false
v: 0
area: ART
workgroup: TIGRESS WG
keyword:
 - tigress
 - threat model
venue:
  group: TIGRESS
  type: Working Group
  mail: tigress@ietf.org
  arch: https://datatracker.ietf.org/wg/tigress/about/
  github: lassey/tigress-threat-model
  latest: https://github.com/bslassey/tigress-threat-model/edit/main/draft-lassey-tigress-threat-model.md
author:
 -
    fullname: Brad Lassey
    organization: Google
    email: lassey@google.com

normative:

informative:


--- abstract

TODO Abstract


--- middle

# Introduction
The TIGRESS Working Group is chartered to deliver a protocol for transfering copies of digital credentials. The charter specifies certain goals:

## Privacy goals:

* The relay server should not see sensitive details of the share

* The relay server should not be able to provision the credential itself,
acting as an intermediary for the recipient (person-in-the-middle,
impersonation attack)

* Aside from network-level metadata, the relay server should not learn
information about the sender or receiver

## Security goals:

* Ensure only the intended recipient is able to provision the credential

* Ensure the credential can only be provisioned once (anti-replay)

* Ensure the sender has the intent to transfer (proof of the fact that the
share initiation is attributed to a valid device and a user)

## Functional goals:
* Allow a sender to initiate a share and select a relay server

* Allow a recipient to view the share request, and provision the credential
associated with the share upon receipt

* Allow a sender and a recipient to perform multiple round trip communications
within a limited time frame

* Not require that both the sender and recipient have connectivity to the relay
server at the same time

* Support opaque message content based on the credential type

* Support a variety of types of credentials, to include those adhering to
public standards (e.g., Car Connectivity Consortium) and proprietary (i.e.,
non-public or closed community) formats

From these goals we can derive a threat model.

# Threat Model



| Item |   Asset    |   Threat               |     Impact                       |     Mitigation                           | Comment |
|:-----|:-----------|:-----------------------|:---------------------------------|:-----------------------------------------|:--------|
|   1  | Owner's DCK| Kicking-off arbitrary key sharing by spoofing user identity | DCK becomes shared with arbitrary user/adversary allowing them access to the Owner's car| 1) User auth (face/touch ID), 2) Secure Intent |   |
| 2    | Content on Intermediary server | Content recovery by brute forcing secret | Exposure of encrypted content and key redemption | 1) Strong source of randomness for salt, 2) At least 128 bit key length, 3) Limitted TTL of the mailbox |   |
| 3    | Content on Intermediary server | Content recovery by intercepting secret | Ability to decrypt content on Intermediary server | 1) Physical separation between content and secret, e.g. secret sent as URI fragment to recipient, 2) Optional second factor(e.g. Device PIN, Activation Options - please refer to CCC Technical Specification) can be proposed to the user via user notification based on security options of selected primary sharing channel (used to share URL with secret) |  |
| 4    | Content on Intermediary server | Access to content by multiple arbitrary  users/devices | 1) Adversary can go to partner and redeem the shared key, 2) Adversary can send push notifications | 1) Mailboxes identified by version 4 UUID defined in {{!RFC4122}}(hard to guess/bruteforce), 2) Mailboxes 'tied' to sender and recipient (trust on first use via deviceClaim), 3) TTL limit for mailboxes, 4) Mailboxes deleted after pass redemption |   |
 5    | Content on Intermediary server | Compromised Intermediary server | 1) Adversary can redeem the sharedKey, 2) Adversary can send push notifications | 1) Separation between content and secret, e.g. secret sent as URI fragment to recipient, 2) TTL limit for mailboxes |    |
| 6    | Content on Intermediary server and Push Tokens | Unauthenticated access to mailbox on Intermediary server | 1) Adversary can redeem the sharedKey, 2) Adversary can send push notifications | 1) Mailboxes identified by version 4 UUID defined in {{!RFC4122}}(hard to guess/bruteforce), 2) Mailboxes 'tied' to sender and recipient (trust on first use via deviceClaim), 3) TTL limit for mailboxes, 4) Mailboxes deleted after pass redemption |   |
| 7    | Content on Intermediary server | User stores non-credential information in mailbox (e.g. "cat pictures") | Service abuse, Adversary can use Intermediary server as cloud storage | 1) Mailboxes have size limit, 2) Mailboxes have TTL |   |
| 8    | Device PIN | Receiver device compromised (redemption before friend) | Device PIN can exposure and forwarding to an advarsary | Activation Options as defined in {{CCC-Digital-Key-30}}, Section 11.2 Sharing Principles, subsection 11.2.1.3. Activation Options |    |
| 9    | Device PIN | Weak PIN can be easily guessed | Anyone with share URL in their possession can guess the PIN and redeem the key | 1) Use of strong RNG as a source to generate Device PIN, 2) Long enough PIN (e.g. 6 digits) as per {{NIST-800-63B}} recommendations, 3) Limit the number of retries (e.g. Device PIN retry counter + limit) as per {{NIST-800-63B}} recommendations | {{NIST-800-63B}}, section 5.1.1.1 Memorized Secret Authenticators |
| 10    | Device PIN | Eavesdropping on weak msg channels/app | PIN exposure would allow one with possession of share URL and Secret to redeem key | In person, out of band PIN transfer, e.g. voice channel |  |
| 11    | Device PIN | PIN recovery via timing attack | Adversary with shared URL in possession can recover PIN based on the response delay, in the case where the PIN verification is not invariant | 1) Time invariant compare, 2) PIN retry counter/limit |  |
| 12    | Device PIN retry counter/limit | Device PIN brute force | Device PIN successful guess | 1) Use of strong RNG as a source to generate Device PIN, 2) Long enough PIN (e.g. 6 digits) as per  {{NIST-800-63B}} recommendations, 3) Limit the number of retries (e.g. Device PIN retry counter + limit) as per {{NIST-800-63B}} recommendations | {{NIST-800-63B}}, section 5.1.1.1 Memorized Secret Authenticators |
| 13    | Sharing Invitation | Messaging channel eavesdropping  | Share invitation forwarding and DCK redemtion by malicious party | 1) Send invitation and Device PIN via different channels, e.g. Device PIN can be shared out of band (over voice), 2) Use of E2E encrypted msg apps/chhannel |   |
| 14    | Sharing Invitation | Voluntary/Involuntary forwarding by Friend | DCK redemption before Friend | Use of messaging apps with anti-forwarding mechanisms(e.g. hide link, copy/past prevention) |   |
| 15    | Sharing Invitation | Friend device compromise allow malware to forward invitation to an adversary | Share invitation forwarding and key redemption by malicious party | Activation Options as defined in {{CCC-Digital-Key-30}}, Section 11.2 Sharing Principles, subsection 11.2.1.3. Activation Options |   |
| 16    | Sharing Invitation | User mistakenly shares with the wrong person | DCK redemption by adversary/not intended user | 1) Send invitation and Device PIN via different channels, e.g. Device PIN can be shared out of band (over voice), 2) DCK revocation |   |
| 17    | Sharing Invitation | Owner device compromise allow malware to forward invitation to an adversary | Share invitation forwarding and key redemption by malicious party | Activation Options as defined in {{CCC-Digital-Key-30}}, Section 11.2 Sharing Principles, subsection 11.2.1.3. Activation Options |   |
| 18    | Sharing Invitation | Friend device OEM account take over | DCK provisioning on adversary's device | 1) Binding to deviceClaim, 2) Device PIN shared out of band, 3) Activation Options as defined in {{CCC-Digital-Key-30}}, Section 11.2 Sharing Principles, subsection 11.2.1.3. Activation Options |   |
| 19    | User's credentials, payment card details, etc| Phishing attacks leveraging malicious Java Script in preview page | 1) Preview page URL fragement contains encryption key - meaning malicious JS could use key to decrypt contents, 2) Malicious Java Script can phish for user credentials, payment card information, or other sensitive data | 1) Properly vet Java Script that is embedded in the preview page, 2) Define strong content security policy |   |


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

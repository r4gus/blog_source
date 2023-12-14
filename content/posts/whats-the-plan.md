+++
title = "FIDO2: everything perfect?"
date = 2023-12-14T17:27:36+01:00
math = true
author = "David P. Sugar"
cover = ""
tags = ["fido2", "fido", "ctap", "ctap2", "webauthn", "passkey", "passkeys"]
keywords = ["fido2", "fido", "ctap", "ctap2", "webauthn", "passkey", "passkeys"]
description = "Some personal concerns regarding FIDO"
showFullContent = false
readingTime = false
hideComments = false
+++

Around a year ago, I found myself captivated by the FIDO2 authentication protocol, also known as [PassKey](https://fidoalliance.org/passkeys/). The promises made by the FIDO Alliance, a consortium featuring tech giants like Microsoft, Google, and Apple, were compelling - envisioning a future where passwords become obsolete, replaced by a faster, easier, and more secure authentication system across all devices (see Passkeys).

If you've been keeping an eye on cybersecurity news, you're likely familiar with the significant incidents stemming from phishing attacks. These malicious activities often involve tricking individuals into revealing sensitive information, such as passwords or inadvertently approving actions on a second-factor app.

Over the past year, I've delved into the FIDO2 specifications and embarked on creating my own FIDO "ecosystem." This involved developing a [CBOR library](https://github.com/r4gus/zbor), a [FIDO2 library](https://github.com/r4gus/keylib) for authenticator and client implementation, and introducing [PassKeeZ](https://github.com/r4gus/keypass), a Linux-centric FIDO2 authenticator. While the concept behind FIDO2 is compelling, I've also come across aspects that seem less compelling. This blog post isn't a deep dive into how FIDO2 operates; instead, it's a reflection on concerns I have regarding the management of the "FIDO project" and some misconceptions that seem prevalent.

NOTE: If FIDO2 is new to you, I recommend doing some research before continuing. Without this background, the following might not make much sense.

## Platform authenticators not a thing ?!

Diving into the [CTAP2 specification](https://fidoalliance.org/specs/fido-v2.2-rd-20230321/fido-client-to-authenticator-protocol-v2.2-rd-20230321.html#transport-specific-bindings), one quickly notices a crucial limitation - only three transport-specific bindings are explicitly listed: USB, NFC, and Bluetooth (and a hybrid). Why is this a significant hurdle? Allow me to elucidate: The CTAP2 spec defines how an authenticator and a client (typically your web browser) should communicate. If an authenticator doesn't adhere to one of these specified transport bindings, it becomes incompatible with WebAuthn, rendering it useless for web-based authentication. Notably absent from the spec are platform APIs for Linux, Mac, or Windows, posing a challenge for those looking to offer platform authenticators, similar to the functionality of password managers like KeePassXC. As a workaround for Linux, one can leverage the uhid module to create a virtual USB device.

Whenever I raise this concern, the responses often downplay the relevance of FIDO2 authenticators running on the same platform by arguing that platform authenticators are not covered by FIDO2. However, this assertion is fundamentally flawed for several reasons, and even the FIDO Alliance has embraced the concept, referring to platform authenticators as "Passkey."

Firstly, the CTAP2 specification itself introduced the idea of "platform authenticators." Secondly, the FIDO Alliance's [certification program](https://fidoalliance.org/certification/authenticator-certification-levels/) outlines three security levels (L1, L2, and L3), with L1 emphasizing protection against phishing and scalable attacks through software and security best practices. The website cites examples like "Downloaded app making use of Touch ID on iOS" and "FIDO2 built into a downloadable web browser app" as L1, signaling the acceptance and promotion of platform authenticators.

Some argue that a compromised system compromises the security of a platform authenticator. While true, the same holds for various sensitive assets like SSH and PGP keys and encrypted KDBX database files. Additionally, since 2023, Microsoft and Apple have introduced their own platform authenticators.

With these points in mind, the lingering question is: If platform authenticators are acknowledged and endorsed by the FIDO Alliance, why are there no system APIs for Linux, Mac, etc., specified in the standard?

To be frank, I don't have a concrete answer, but I can speculate.

1. A few years ago (when FIDO2 was introduced), the market for authenticators was relatively small, and the concept of cross-device FIDO credentials (Passkeys) wasn't as prevalent. Credentials were tightly bound to devices, potentially limiting interest in developing platform authenticators.
2. Companies, such as Yubico, sell authenticators for around $50. Without readily available platform authenticators, those seeking an alternative to passwords are pushed to purchase authenticators to adopt the new protocol. The same companies shaping FIDO standards may have a vested interest in not "flooding the market" with more affordable alternatives.
3. Market dominance likely plays a significant role. If you've ever watched a Louis Rossman video, you're probably aware that most of the big companies don't like to share. They tend to guard their territory, as evidenced by certain companies receiving preferential treatment regarding FIDO2 system APIs. Although nothing official is specified in the standard, companies like Microsoft have their APIs (e.g., for Windows Hello) for browser communication. Interestingly, system APIs are defined for the FIDO UAF protocol. Unfortunately, UAF is not compatible with WebAuthn (WebAuthn is part of FIDO2, not UAF).

## Meta Data Service

Another aspect that raises a red flag for me is the [FIDO Meta Data Service](https://fidoalliance.org/specs/mds/fido-metadata-service-v3.0-ps-20210518.html). In a nutshell, vendors can upload metadata about their certified products to a central MDS server. Servers utilizing FIDO2 for authentication can then download this metadata to enforce specific policies. While this approach makes perfect sense for services with high-security standards like internal company networks, governments, or critical infrastructure, I'm skeptical about its application to less critical (web) services.

The concern is that we might be on the slippery slope towards a dystopian scenario where only platform authenticators survive, that have the financial means to pay the certification fees. The potential compromise here would be for the FIDO Alliance to provide the MDS service free of charge to application developers lacking strong financial support, should the MDS become the defacto standard across all major services on the web. Without such measures, there's a genuine fear that projects like PassKeeXC (a password manager I hold in high regard – and if you share the sentiment, I encourage you to [contribute to their cause](https://keepassxc.org/donate/)) could become relics of the past. This could leave users with a limited selection: Passkey for Windows (courtesy of Microsoft), Passkey for Mac (courtesy of Apple), Passkey for Android (courtesy of Google), and Linux... well, at least for now, you can use my authenticator ;).

## Conclusion

As I eagerly observe the development of the FIDO2 standard, I can't help but wonder about its acceptance among the average user. Despite the critical perspectives shared in this article, it's crucial to clarify that I'm rooting for FIDO2 to succeed. But rooting for its success is not synonymous with ignoring the potential pitfalls.

In this journey towards a passwordless future, the hope is that FIDO2 evolves while preserving an open standard that respects the freedom of both users and developers. 

In the end, the success of FIDO2 is not just about technological advancements; it's about fostering an ecosystem where security, accessibility, and user freedom coexist harmoniously. Let's navigate this evolution with optimism, mindful of the responsibility to keep the digital landscape inclusive and respectful of individual choices.

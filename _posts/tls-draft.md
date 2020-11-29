---
layout: post
title:  ""
date:   2020-11-10 14:00:00 +0800
categories: tls 
---

### Preface
The concept of TLS/SSL has always escaped me, so I decided to take the time to understand it and to write about it. Many thanks to a friend for always entertaining my questions.  

### TLS vs SSL
Both TLS and SSL are used to secure internet communications. So why are there two? TLS is just the updated version of the now deprecated SSL. TLS will be the focal point of this blog post. 

### TLS
TLS secures a connection. An example of a secured connection would be HTTPS; HTTPS is just HTTP secured with TLS.

When a connection wants to be encrypted and secured using TLS, *something* needs to initialize in order to secure the connection. This *something* refers to a TLS handshake. 

> What is meant by a secured connection? A connection is secured where both parties involved in the communication **agrees on a key to be used to encrypt and decrypt information (i.e. a symmetric key)** and other parties are not privy to this. The TLS handshakes helps to facilitate this. 

### TLS Handshake
All the jargons on TLS like public keys, private keys, certificates that we often hear about are ingredients necessary for the TLS handshake to occur.

The following is not the most comprehensive and detailed description of the handshake process, but it seeks to capture the most important areas of it. 

1. Client (e.g. a web browser) tries to establish a connection to the server. It sends a client random (32-byte random number), the TLS version it can support, and other metadata to the server.
1. Server sends back a server random, a [certificate](certificate) (it also contains the public key of the server), and...

*Detail about TLS handshake*

*Detail about CSR*

Bold **signs** and **encrypts**

*Detail about signing vs encryption*

The recipient can encrypt some verification metadata using their private key which can only be decrypted by their public key. This is to allow the sender to verify the authenticity of the recipient.

### Certificate


### Public Key & Private Key
In the past, I used to believe that a public key can only encrypt while a private key can only decrypt. I had a misconception of this, and cannot be more wrong. 

> When an asymmetric keypair *k1*, *k2* is generated (in the case of RSA). Mathematically, we can use *k1* to encrypt and only *k2* can be used to decrypt this. Conversely, when we use *k2* to encrypt, we can only use *k1* to decrypt. It's our choice whether to use *k1* or *k2* as the public or private key. 

In the TLS protocol, we want only the recipient to read the content of a message. Thus, we select a key (from the keypair, owned by the recipient) to be used to decrypt the content and keep this private to the recipient, hence the name, private key. To allow senders to sent encrypted messages to a recipient, the other key (from the keypair, owned by the recipient) has to be distributed publicly, hence the name, public key. 



### References

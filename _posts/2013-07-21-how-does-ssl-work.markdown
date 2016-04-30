---
layout: post
title: "How does SSL Work?"
date: 2013-04-21
categories: [Security]
keywords: "SSL, TSL, Security, Secure Sockets Layer, SSL Explaination"
description: "An explanation how SSL/TSL works, including asymmetric and symmetric encryption."
---
There are plenty of resources on the internet that explains this very well, such as [here](http://technet.microsoft.com/en-us/library/cc783349\(v=ws.10\).aspx),
[here](http://security.stackexchange.com/questions/20803/how-does-ssl-work), [here](https://www.youtube.com/watch?v=iQsKdtjwtYI)
and [here](http://crypto.stackexchange.com/questions/1824/why-do-we-need-asymmetric-algorithms-for-key-exchange). So why
blog about something that is already explained? Well, it's my way of achieving a deeper understanding, and having a
single place I can refer to. If it helps someone else, well that's just a bonus.

Understanding SSL and it's successor TSL involves grasping a couple of concepts, which I will explain below. Let's start 
with what it stands for: **Secure Socket Layer** and **Transport Layer Security**. Now days, it's almost solely TLS that 
is used in the wild, even though SSL is still the more popular acronym.

SSL and TSL are cryptographic protocols that provide communication security over the internet. These protocols solve two 
key issues required to have a secure communication channel:

+ Verification of the entity I am communicating with (*you are who you say you are*)
+ Prevent eavesdropping of the communications between two by other parties (*only you and I know what we are saying to each other*)

The most common usage of SSL\TSL is the one of a website served over HTTPS. In this instance, the client is a web
browser and the server is the website. However, the client can be any program such as a SSH client e.g. PuTTY.

After a TCP connection is established, a handshake is initiated by the client. The handshake can be broken into the
following steps:

+ **Client Hello**
    + The client sends a bunch of information such as: Highest TLS version supported, a random value, list of supported
      [cipher suites](http://en.wikipedia.org/wiki/Cipher_suite "cipher suites") and compression methods by the client.
+ **Server Hello**
    + The server sends:
        + The chosen TLS version, cipher suite and compression method from the client's list.
        + A random number and session id.
        + Its digital certificate.
        + Server Hello Done message to indicate the completion of the handshake negotiation.
+ **Authenticate Server's Certificate**
    + This involves understanding the concepts of [asymmetric encryption](http://en.wikipedia.org/wiki/Public-key_cryptography "asymmetric encryption"),
      [symmetric encryption](http://en.wikipedia.org/wiki/Symmetric-key_algorithm "symmetric encryption") and
      [certificate authority](https://en.wikipedia.org/wiki/Certificate_authority "certificate authority").
    + We verify a bunch of properties on the certificate, including the public key, making sure it is a valid certificate.

---

**Asymmetric Encryption**, also known as public-key encryption is a system requiring two keys, a public key and a
private key, which is secret. The public key is used for encrypting and the private key is for decrypting. The public
key is different to the private key, although mathematically related.

**Symmetric Encryption** is when the key used to encrypt and decrypt a message is identical. Hence, the key must known
to both parties in the conversation and they must both keep it private.

**Certificate Authority (CA)** is an entity that issues digital certificates.

#### Why are certificate authorities important ?#### 
Well, it ties into asymmetric encryption. When the server sends their digital
certificate, with it's public key, how do we trust it? How do I know it's not someone else in the middle, sending me a
dodgy digital certificate pretending to be the server?

Since it is not practical to store all the public keys for all the certificates in the world, the concept of a
certificate authority was established.

For example, when [Facebook](http://www.facebook.com" "Facebook) sends it's certificate, it mentions that the
certificate is issued by [VeriSign](http://www.verisign.com/ "VeriSign"), which is a certificate authority. To sign a
certificate you must know the private key, which is *only* known to the CA. This prevents a an attacker falsely signing
a certificate, and claiming to be Facebook for example.

<div class="centered">
    <a href="/images/facebookcert.png" target="_blank">
        <img src="/images/facebookcert.png"  alt="Facebook Certificate" style="width: 199px; height: 244px" />
    </a>
</div>

#### So where do this list of trusted CAs come from ?#### 
When a operating system or browser is installed, a list of trusted of certificate authorities comes with it, along with
their public keys. This list can be modified, where you can remove a CA you do not trust, and similarly you can make
your own and add it (however, no one other than yourself will be trusting this certificate authority).

If you want to view your list of trusted CAs on a windows 7 machine, then follow these instructions:

+ Open up Microsoft Management Console (Start &rarr; type mmc &rarr; Press ENTER)
+ Go to the File menu, and select Add/Remove Snap-in option
+ Select the Certificates snap-in and click the Add button
+ Select My user account and click Finish
+ Press the OK button
+ Expand the Certificates tree &rarr; Trusted Root Certification Authorities &rarr; click on Certificates
+ You should see something like the image below

<div class="centered">
    <a href="/images/certmanager.png" target="_blank">
        <img src="/images/certmanager.png"  alt="Certificate manager" style="width: 570px; height: 321px" />
    </a>
</div>

#### How can we be sure a certificate authority is going to behave responsibly ?#### 
This is a *point of unverifiable trust* in this system. We must trust that the certificate authority is going behave
responsibly, and not give out certificates just to anyone that asks for it. When a large corporation such as Facebook
uses a certificate authority, it would be safe to assume that there are procedures placed in the CA to make sure
everything is done according to the rules. Issuing a certificate requires the registrant to prove ownership of the
domain they are requesting the certificate for.

---

+ **Send Pre-Master Secret to Server (Client Key Exchange)**
    + The client then generates a random secret key that eavesdroppers cannot be figure out.
    + The pre-master secret is then encrypted with the server's public key and then sent to the server.
+ **Master Secret**
    + The server can use its private key and decrypt the pre-master secret sent by the client.
    + Both the client and server now have the pre-master secret. They both use this and the previously exchanged random
      numbers to generate a master secret.
    + The master secret is then used to generate symmetric keys used to encrypt and decrypt information after the SSL
      handshake is done.
+ **Client Finished**
    + The client will send a message to the server denoting that all messages from now on will be encrypted using the
      generated symmetric key.
    + The client then sends a separate encrypted message (known as the <em>Finished</em> message) with a hash of all the
      handshake messages to the server.
    + The server will try to decrypt this message and verify it. If it fails, the connection should be torn down.
+ **Server Finished**
    + The server will also send a message to the client denoting that all messages from now on will be encrypted using
      the generated symmetric key.
    + The server will then send its encrypted <em>Finished</em> message to the client.
    + The client will decrypt and verify the message.
+ **Hello Application Layer**
    + From here on out, all the messages are encrypted with generated symmetric key.

This is an overview of how SSL works, but for a more in depth understanding, read
[Jeff Moser's comprehensive explanation](http://www.moserware.com/2009/06/first-few-milliseconds-of-https.html "Jeff Moser's comprehensive explanation").

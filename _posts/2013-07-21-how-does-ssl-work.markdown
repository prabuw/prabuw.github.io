---
layout: post
title: How does SSL Work?
categories:
- Security
tags:
- Cryptography
- Security
- SSL
- TLS
status: publish
type: post
published: true
meta:
  geo_public: '0'
  _publicize_pending: '1'
author: 
---
<p align="justify">There are plenty of resources on the internet that explains this very well, such as <a href="http://technet.microsoft.com/en-us/library/cc783349(v=ws.10).aspx" target="_blank">here</a>, <a href="http://security.stackexchange.com/questions/20803/how-does-ssl-work" target="_blank">here</a>, <a href="https://www.youtube.com/watch?v=iQsKdtjwtYI" target="_blank">here</a> and <a href="http://crypto.stackexchange.com/questions/1824/why-do-we-need-asymmetric-algorithms-for-key-exchange" target="_blank">here</a>. So why blog about something that is already explained? Well, it’s my way of achieving a deeper understanding, and having a single place I can refer to. If it helps someone else, well that’s just a bonus.</p>
<p align="justify">Understanding SSL and it’s successor TSL involves grasping a couple of concepts, which I will explain below. Let’s start with what it stands for: <em><strong>Secure Socket Layer</strong></em> and <strong><em>Transport Layer Security</em></strong>. Now days, it’s almost solely TLS that is used in the wild, even though SSL is still the more popular acronym. </p>
<p align="justify">SSL and TSL are cryptographic protocols that provide communication security over the internet. These protocols solve two key issues required to have a secure communication channel:</p>
<ul>
<li>
<div align="justify">Verification of the entity I am communicating with (<strong><em>you are who you say you are</em></strong>)</div>
<li>
<div align="justify">Prevent eavesdropping of the communications between two by other parties (<strong><em>only you and I know what we are saying to each other</em></strong>)</div>
</li>
</ul>
<p align="justify">The most common usage of SSL\TSL&nbsp; is the one of a website served over HTTPS. In this instance, the client is a web browser and the server is the website. However, the client can be any program such as a SSH client e.g. PuTTY.</p>
<p align="justify">After a TCP connection is established, a handshake is initiated by the client. The handshake can be broken into the following steps:</p>
<ol>
<li>
<div align="justify"><strong><em>Client Hello</em></strong></div>
<ul>
<li>
<div align="justify">The client sends a bunch of information such as: Highest TLS version supported, a random value, list of supported <a href="http://en.wikipedia.org/wiki/Cipher_suite" target="_blank">cipher suites</a> and compression methods by the client.</div>
</li>
</ul>
<li>
<div align="justify"><strong><em>Server Hello</em></strong></div>
<ul>
<li>
<div align="justify">The server sends:</div>
<ul>
<li>
<div align="justify">The chosen TLS version, cipher suite and compression method from the client’s list. </div>
<li>
<div align="justify">A random number and session id.</div>
<li>
<div align="justify">Its digital certificate.</div>
<li>
<div align="justify">Server Hello Done message to indicate the completion of the handshake negotiation.</div>
</li>
</ul>
</li>
</ul>
<li>
<div align="justify"><strong><em>Authenticate Server’s Certificate</em></strong></div>
<ul>
<li>
<div align="justify">This involves understanding the concepts of&nbsp; <a href="http://en.wikipedia.org/wiki/Public-key_cryptography" target="_blank">asymmetric encryption</a>, <a href="http://en.wikipedia.org/wiki/Symmetric-key_algorithm" target="_blank">symmetric encryption</a>&nbsp; and <a href="https://en.wikipedia.org/wiki/Certificate_authority" target="_blank">certificate authority</a>. </div>
<li>
<div align="justify">We verify a bunch of properties on the certificate, including the public key, making sure it is a valid certificate.</div>
</li>
</ul>
</li>
</ol>
<p align="justify">
<hr>
<p align="justify"><em><strong>Asymmetric Encryption</strong>, also known as public-key encryption is a system requiring two keys,&nbsp; a public key and a private key, which is secret. The public key is used for encrypting and the private key is for decrypting. The public key is different to the private key, although mathematically related.</em></p>
<p align="justify"><font face="Georgia"><em><strong>Symmetric Encryption </strong>is when the key used to encrypt and decrypt a message is identical. Hence, the key must known to both parties in the conversation and they must both keep it private.</em></font></p>
<p align="justify"><em><strong>Certificate Authority (CA) </strong>is an entity that issues digital certificates. </em></p>
<p align="justify">Why are certificate authorities important? Well, it ties into asymmetric encryption. When the server sends their digital certificate, with it’s public key, how do we trust it? How do I know it’s not someone else in the middle, sending me a dodgy digital certificate pretending to be the server?</p>
<p align="justify">Since it is not practical to store all the public keys for all the certificates in the world, the concept of a certificate authority was established.</p>
<p align="justify">For example, when <a href="http://www.facebook.com" target="_blank">Facebook</a> sends it’s certificate, it mentions that the certificate is issued by <a href="http://www.verisign.com/" target="_blank">VeriSign</a>, which is a certificate authority. To sign a certificate you must know the private key, which is <em>only</em> known to the CA. This prevents a an attacker falsely signing a certificate, and claiming to be Facebook for example.</p>
<p align="justify"><a href="http://pwee167.files.wordpress.com/2013/08/facebookcert.png"><img title="facebookcert" style="background-image:none;float:none;padding-top:0;padding-left:0;margin-left:auto;display:block;padding-right:0;margin-right:auto;border-width:0;" border="0" alt="facebookcert" src="http://pwee167.files.wordpress.com/2013/08/facebookcert_thumb.png" width="199" height="244"></a></p>
<p align="justify"><em>So where do this list of trusted CAs come from?</em> </p>
<p align="justify">When a operating system or browser is installed, a list of trusted of certificate authorities comes with it, along with their public keys. This list can be modified, where you can remove a CA you do not trust, and similarly you can make your own and add it (however, no one other than yourself will be trusting this certificate authority).</p>
<p align="justify">If you want to view your list of trusted CAs on a windows 7 machine, then follow these instructions: </p>
<ul>
<li>
<div align="justify">Open up Microsoft Management Console (Start –> type mmc –> Press ENTER)</div>
</li>
<li>
<div align="justify">Go to the File menu, and select Add/Remove Snap-in option</div>
</li>
<li>
<div align="justify">Select the Certificates snap-in and click the Add button</div>
</li>
<li>
<div align="justify">Select My user account and click Finish</div>
</li>
<li>
<div align="justify">Press the OK button</div>
</li>
<li>
<div align="justify">Expand the Certificates tree&nbsp; –> Trusted Root Certification Authorities –> click on Certificates</div>
</li>
<li>
<div align="justify">You should see something like the image below</div>
</li>
</ul>
<p align="justify"><a href="http://pwee167.files.wordpress.com/2013/08/image.png"><img title="image" style="border-top:0;border-right:0;background-image:none;border-bottom:0;float:none;padding-top:0;padding-left:0;margin-left:auto;border-left:0;display:block;padding-right:0;margin-right:auto;" border="0" alt="image" src="http://pwee167.files.wordpress.com/2013/08/image_thumb.png" width="570" height="321"></a></p>
<p align="justify"><em></em>&nbsp;</p>
<p align="justify"><em>How can we be sure a certificate authority is going to behave responsibly?</em></p>
<p align="justify">This is a <em>point of unverifiable trust</em> in this system. We must trust that the certificate authority is going behave responsibly, and not give out certificates just to anyone that asks for it.&nbsp; When a large corporation such as Facebook uses a certificate authority, it would be safe to assume that there are procedures placed in the CA to make sure everything is done according to the rules. Issuing a certificate requires the registrant to prove ownership of the domain they are requesting the certificate for.</p>
<p align="justify">
<hr>
<ol start="4">
<li>
<div align="justify"><strong><em>Send Pre-Master Secret to Server (Client Key Exchange)</em></strong></div>
<ul>
<li>
<div align="justify">The client then generates a random secret key that eavesdroppers cannot be figure out.</div>
<li>
<div align="justify">The pre-master secret is then encrypted with the server’s public key and then sent to the server. </div>
</li>
</ul>
<li>
<div align="justify"><strong><em>Master Secret</em></strong></div>
<ul>
<li>
<div align="justify">The server can use its private key and decrypt the pre-master secret sent by the client.</div>
<li>
<div align="justify">Both the client and server now have the pre-master secret. They both use this and the previously exchanged random numbers to generate a master secret.</div>
<li>
<div align="justify">The master secret is then used to generate symmetric keys used to encrypt and decrypt information after the SSL handshake is done.</div>
</li>
</ul>
<li>
<div align="justify"><strong><em>Client Finished </em></strong></div>
<ul>
<li>
<div align="justify">The client will send a message to the server denoting that all messages from now on will be encrypted using the generated symmetric key.</div>
<li>
<div align="justify">The client then sends a separate encrypted message (known as the <em>Finished</em> message) with a hash of all the handshake messages to the server. </div>
<li>
<div align="justify">The server will try to decrypt this message and verify it. If it fails, the connection should be torn down.</div>
</li>
</ul>
<li>
<div align="justify"><strong><em>Server Finished</em></strong></div>
<ul>
<li>
<div align="justify">The server will also send a message to the client denoting that all messages from now on will be encrypted using the generated symmetric key.</div>
<li>
<div align="justify">The server will then send its encrypted <em>Finished</em> message to the client.</div>
<li>
<div align="justify">The client will decrypt and verify the message.</div>
</li>
</ul>
<li>
<div align="justify"><strong><em>Hello Application Layer</em></strong></div>
<ul>
<li>
<div align="justify"><font face="Georgia">From here on out, all the messages are encrypted with generated symmetric key.</font></div>
</li>
</ul>
</li>
</ol>
<p align="justify">
<p align="justify">&nbsp;</p>
<p align="justify">This is an overview of how SSL works, but for a more in depth understanding, read <a href="http://www.moserware.com/2009/06/first-few-milliseconds-of-https.html" target="_blank">Jeff Moser’s comprehensive explanation</a>.</p>

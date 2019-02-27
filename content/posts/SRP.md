---
title: "What is The Secure Remote Password (SRP) protocol?"
date: 2018-02-27
tags: ["SRP", "Secure Remote Password", "key exchange", "security", "cryptography"]
draft: false
repo: "https://www.onaralili.com/posts/srp"
---
I have decided to publish materials I wrote for my Computer and Network Security course. It could help someone out there, so why keep it on my hard drive.
You can obviously get shorter and straightforward information from following Wikipedia page.
https://en.wikipedia.org/wiki/Secure_Remote_Password_protocol

## So, what is it?
The Secure Remote Password (SRP) protocol is a great method for securing remote access to many applications. It has been developed by Thomas Wu at Stanford University to
enable the secure authentication based on username and password. This
article discusses cryptographically strong SRP authentication. The protocol
allows both sides (server and client) exchange common key and authenticate
a user to the server. After authentication, they exchange message via a key
they agree upon using encryption algorithms such as AES.
One of the main concern in network authentication methods is securely authenticating a user to server or service with minimal effort. Nowadays, many users use simple, short passwords for authentication which raises security concerns. The SRP solves traditional problems such as weak keys and doesn’t require certificate infrastructure and a user are free to not to store long-term keys. It considered easy to deploy and implement and secure than conventional challenge-response systems. The protocol is developed under the idea that users like to use passwords that are easy to use and remember. As time being, there are well-known attacks against network authentication protocols such as simple passive attacks, sniffing attacks. Many other protocols require a server to store a password p or some hashed version h(p) which is not considered safe since once the server gets compromised hashed password can be cracked using a dictionary attack. On the other hand, SRP offers total protection for both passive and active threats by using DH key exchange like method, therefore it is effected even with a weak password and can be integrated with working applications. It offers perfect forward secrecy, meaning even the server has been hacked, the attacker cannot obtain any information about past and future communication since in every transaction a new key is being derived. Let’s see how SRP provides this security:
In order to for a user to authenticate to a server, it has to go through a setup. Firstly, the client selects a random salt s and computes its has x= H(s,p) and also verifier. The verifier as the name suggests is a password verifier and it is computed v= gX. When the server receives the password verifier and calculated hash it stores salt s and the verifier but not calculated the hash. The server store the information that came from the client by the index I.
The important point here is that the salt s must be enforced by the server in order to prevent the identical passwords look the same. At this point, salt s is being shared between the client and the server then later uses this information to make sure that the password a user provides matches with the one server has and establish a session key. After sharing a session key K, they need to complete authentication and provide each other information that their keys are identical, one way to do it is following:  
1. Alice sends Bob: M1 = H[H(N) XOR H(g) | H(I) | s | A | B | Kalice].
2. Bob verifies M1 and sends Alice: M2 = H(A | M1 | Kbob).
3. Alice verifies M2.

In the above description, Alice sends Bob hashed H(N) where N is a large enough safe prime XORed with H(g) and s as a salt. A and B represent a random ephemeral key in order to provide perfect forward secrecy.

## The Security features and Comparison
The Secure Remote Password is one of the password-authenticated key agreement (PAKE) methods. These methods establish a secure key based on only a password while providing strong security against eavesdroppers and <a href="https://en.wikipedia.org/wiki/Man-in-the-middle_attack">man in the middle attacks.</a> There are two kinds of PAKE mechanisms Balanced and Augmented.
Balanced PAKE enables participants to consume the same shared password to authenticate a key. For instance, SPEKE and EKE are considered balanced PAKE.
Augmented PAKE is the case where the server doesn’t hold any information unless brute-force attack is carried. For this method, we can give as an example SRP
We will go through security features of Encrypted Key Exchange (EKE) and SRP and compare their advantages and disadvantages over each other. EKE is a pioneer of the password-authentication agreement and has been tested and implemented many times. Although some vulnerability has been discovered it updated and survived to nowadays. The most important security difference between EKE/SPEKE and SRP is that both EKE and SPEKE passwords are ”password-like”, meaning if an attacker obtains access to the server, he/she can potentially impersonate users on the database. Even though there are versions that remove this vulnerability, it has a high-performance cost. On the other hand, SRP is protected both from active and passive attacks and the passwords are stored don’t give any information to an intruder.

## The SRP integrations

**Telnet**

In order to avoid eavesdropping attacks, SRP has been implemented into Telnet protocol. SRP is used to prevent sending passwords in a plaintext. The SRP improves security by against:
Password sniffing attacks, meaning if the attacker listening to the network cannot see the password over the network
Dictionary attacks, it would take a huge computational time to even start trying to break the password.
SRP requires no certifications, keyrings etc. which makes it simple to understand and implement.
SRP Telnet package is realized as a Telnet authentication option. The SRP authentication provides automatic negotiation and in case both sides support SRP, they will try to authenticate. Otherwise, the authentication protocol downgrades to another mechanism that both participants support. The implementation enables users to command with standard commandline commands such as *telnet ipaddress* instead of using some specific authentication commands such as ssh or kinit. The password will be provided as usual after telnet IPaddress command and the secure session will be established. A simple secure authentication will look like following:
```
$ telnet 217.x.x.x
Trying 217.x.x.x...
Connected to 217.x.x.x
Escape character is ’^]’.
[ Trying SRP ... ]
[ Using 1024-bit modulus for ’tjw’ ]
SRP Password:         (Password is typed locally)
[ SRP authentication successful ]
```
**SSH**

SSH is a tool that enables to log in and runs commands on a remote server. SSH employees RSA public key exchange to ensure a secure connection. Even though SSH consider secure it has implementation problems as well as it sends passwords via a secure connection. The latter one leads to the issue when one of the participants is being compromised. The attacker can listen to incoming traffic and capture incoming passwords and try them
on different machines. This is where SRP plays a great role to force security even a server is being hacked. Since in SRP one cannot obtain information about the password. As we mentioned SSH uses RSA and it is a patented method, therefore, license required. However, the SRP can provide a secure connection to SSH even without having RSA implementation.

**TLS**

As SSH, TLS utilizes public-key cryptography. Nowadays, services authenticate users via username and passwords. TLS doesn’t fit well with this idea. Hence, using the SRP protocol provides secure connection using username and password over an insecure network and avoids from an eavesdropper. The SRP is implemented using the standard handshake message exchange protocol. The exchange goes as follows:
1. Client sends "Client Hello" message to the Server
2. The Server sends "Server Hello"  and Certificate,
Server Key Exchange (N, g, s, B) and
"Server Hello Done" message to the client.
3. The Client sends  Client Key Exchange (A), [Change cipher spec]
and "Finished’ messages to notify finalized exchange.
4. The Server response with "Finished" message.
Both sides start exchanging Application Data.

## The SRP implementation and OpenSSL

In this section, we are going to play around OpenSSL SRP support. As we mentioned above discussion one of the important part of the SRP is verifier. First, we start creating the server verifier file running OpenSSL SRP. As we can see from the guide, -gn command is used to provide g and N values which are required for a verifier, just like [DH key exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange). Given a large enough g and N and output file name (pwd.srpv), we run following command:
```OpenSSL SRP -srpvfile pwd.srpv -add -gn 1536 onar```
After this, the system asks us password twice, after entering and verifying the password the system generates the verifier. It is worth to mention that pwd.srpv has to exist in the file system. Now, we created the verifier and it is ready to load to the server. Using OpenSSL type *SRP_VBASE* to store and utilize the verifier.
At this point, the server should be configurated to show the client side that it supports SRP cipher suits. We can check available ciphers by running 
```openssl ciphers -v — grep SRP``
In order to show the user available ciphers suits, we include suits in the Client Hello message. It goes without saying that username should also be included when the server response to the authentication request. Now, we can finally test implementation on the client side by requesting:
```
./ssl_client -v --SRP localhost:2432
Cipher suites:
SRP-DSS-AES-256-CBC-SHA
SRP-RSA-AES-256-CBC-SHA
SRP-AES-256-CBC-SHA
SRP-DSS-3DES-EDE-CBC-SHA
SRP-RSA-3DES-EDE-CBC-SHA
SRP-3DES-EDE-CBC-SHA
SRP-DSS-AES-128-CBC-SHA
SRP-RSA-AES-128-CBC-SHA
SRP-AES-128-CBC-SHA
```

SSH
---

# What is this talk?

- Understanding what ssh is and how it functions.

# what does the audience hope to gain?

Hopefully by studying SSH's brain as implemented by SSH-1.5 protocol, we can see what security problems it fixes and how it does so.
My goal is to gauge my onw understanding of the concepts herein,

# what is SSH?

Secure, client/server protocol for transmitting data over a network safely.
Authentication can be done by password, host, public key, plus optional integration with other authentication systems.

SSH is not a true shell, rather creates a channel to run shell on remote computer

# where SSH?

Connect between 2 or more computer accounts with a high degree of security
Distributed applications that must communicate over a network securely
Let other people use you computer account, but with limited use

**Before SSH**:
Telnet and Berkley r-commands. This did not encrypt data sent through a network

# A more in depth look

SSH is a protocol , not a product, its a specification of how to conduct secure communication over a network

**Guarantees of SSH**
Authentication: grants access to a system via a digital proof, and a test

Encryption: scrambles data such that it can only be read by intended recipient

Integrity: guarantee data in transit is unaltered.

# Terminology

Local computer, host, machine - computer user is logged in
Remote computer, host, machine -  machine running the ssh server
Local user
Remote user
Server - ssh server program
Server machine
client
client machine - computer running an ssh client

# Related technologies

- PGP: encrypts one file at atime, different between PGP and SSH is like that between a batch job and an interactive process. ssh encrypts an ongoing 
- Kerberos: solve similar problems but in different scopes, SSH is lightweight while kereberos require significant infrastructure changes.
- IPSEC: authentication and encryption at the ip level
- SSL: An SSL participant proves its identity by a digital certificate, a set of cryptographic data. A
certificate indicates that a trusted third party has verified the binding between an identity
and a given cryptographic key
- SSL_Enhanced Telnet and FTP
- Firewalls

# The escape character

**tilde `~`**
 gets the attention of the ssh client, you can add some extra predefined character to effect the escape e.g ^Z will suspend the connection while `~ .` will terminate the connection

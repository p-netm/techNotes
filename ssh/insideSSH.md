# Inside SSH(SSH-1.5)

Deep dive into the bits and bytes

**Features of SSH**

1. Privacy - end-end encryption based on security keys negotiated during an ssh session, this keys are destroyed with the session.
2. Integrity - underlying transport is TCP/IP, TCP/IP has data integrity checks, why is it not enough?
   1. its performed on a per-packet based, prone to replay attacks
   2. ssh integrity is performed on the whole data stream, MD5 and SHA1 ; CRC-32(SSH1)  
3. Authentication - verifying someone's identity, ssh connection involves 2 authentications server authentication and client authentication, helps protect against man-in the middle attacks
4. Authorization - what someone may do or may not do
5. Forwarding/tunnelling - encapsulating another TCP-based service within an SSH session

## A little about cryptography

Encryption is the process of converting plaintext into a cipher text. cipher is th algol used to perform the scrambling and includes algorithms like RSA, DSA.

Encryption algorithms are described as either symmetric or asymmetric.
Symmetric use a similar key for encryption and decryption.Symmetric suffer from a key distribution problem

Asymmetric replaces the single key with a pair of related keys, public and private.
Data encrypted with public key can only be decrypted with the private key, its infeasible to derive the private key from the public key

Modern encryption program implements both symmetric and asymmetric keys to secure large amounts of data. Asymmetric is slower than symmetric by magnitudes

## HAsh Functions

Used to hash data in transit for integrity checking.
eg MD5 and SHA-1, CRC
hash function should be collision-proof, H(m) != H(m');

## Architecture of an ssh system

Server: program allowing ssh connections to machine, usually sshd

client: program that connects to ssh servers, e.g ssh and scp

Session: ongoing connection between client and server

keys:

- user key: identify user to the server, public and persistent
- host key: identify server/machine to the user, public and persistent
- server_key:  protects session key, generated regularly, preserver perfect forward secrecy
- session key: randomly generated for encrypting the communication between the client and server, it is shared securely
- key generator: program that creates persistent keys
- known hosts database: collection of known hosts
- agent: program that caches user keys in memory, responds to key-related operations
- random seed: pool of random data to initialize pseudo-random number generators
- configuration file

### Inside SSH-1

Aspects to cover:

- how the secure session is established
- Authentication by password, public key
- integrity checking
- data compression

**Establishing a secure connection**
Starting from scratch the ssh client and server agree on an encryption algol, generate and share a secret session key

- The client contacts the server
  - Client sends connection request to servers tcp port 22
- The client and server disclose the ssh protocol version they support
- client and server switch to a packet-based protocol
  - Client and server switch to a packet-based protocol over underlying tcp, packets have an extra random 1-8 bytes padding to foil known-plaintext attacks
- Server identifies itself and provides session parameters
  - Server sends host key
  - Server key
  - Server sends eight random bytes, check bytes, client should include them in its next response, protect against some ip spoofing attacks
  - Server sends list of encryption, compression, and authentication methods it supports
  - Both compute a common 128 bit session id, which is used in some subsequent protocol operations to uniquely identify this SSH session. This is an MD5 hash of the host key, server key, and check bytes taken together.
  - Client steps:
    - Checks if host name has a known host key in known keys database
- Client sends the server a session key
  - client randomly generates a new key, purpose to encrypt and decrypt client and server's messages. key is secured by encrypting twice, first with server's public key and with the server key. client sends session key with check bytes, and choice of algorithms to use
- Both sides then turn on encryption and complete server authentication
  - After sending the session key, both sides begin encrypting session data.
  - Client must wait for a valid confirmation response from the server, server authentication is implicit, it needs should be able to decrypt the session key sent by the client
- Secure connection is established

Now the client attempts to authenticate itself to the server. (SSH-1.5)methods available include:

- Kerberos
- Rhosts
- RhostsRSA
- Public-key
- TIS
- Password

Client attempts the above in a predefined arbitrary order.

- Password authentication
  - user supplies a password, client transmits to server securely, 
- Public-key authentication
  - public-key cryptography, client proves that it has a secret key, the private component of an authorized known public key.
  - client sends server a request for public-key authentication 
  - key sent includes a key's modulus as an identifier
  - if found, server looks for entry matching key
  - server retrieves the key and notes any restrictions on its use
  - depending on restrictions, server generates random 256-bit string, encrypts with client's public key
  - the client receives challenge, decrypts its, combines challenge with session identifier, hashes result and returns the response.
    - hashing op prevents misuse of the clients private key, as well as protect against chosen-plaintext attack
  - The server computes the hash of the challenge and sessionId, if the client's reply matches, the authentication succeeds.
- Trusted host authentication
  - establish trust relationships between machines,
    - if logged as user Andrew on machine, and connects to account Bob on machine B using trusted host, machine B ssh server checks id of host A, makes sure its trusted, then checks if the connection was initiated by a privileged program(trusted) the idea is that the program will not lie about andrews identity.
    - easily subverted, naming services can be subverted, ip addresses spoofed, privileged ports aren't so privileged, some OS's do not support privileged ports.
    - SSH supports this as rhosts, but its switched off by default, It has another authentication flow RhostsRSA which adds security by using public-key to authorize a client machine, further instead of using privileged ports, the program running on the client machine needs to prove that it has access to the private key, the private key is always protected so only clients with special privileges can access them

**Integrity checking**

- CRC-32 prone to insertion attack
- This sort of check is sufficient for detecting accidental changes to data, but isn't effective against deliberate corruption

**Compression**

- Uses GNU gzip utility
- Used for file transfers, x forwarding, helps with speed during encryption

### Threats ssh can counter

1. Eavesdropping
   1. content cannot be decrypted by snooper
2. Name service and Ip spoofing
   1. attacker subverts naming service, network-related programs may be coerced to connect to wrong machine, attacker can personate a host by stealing use of its ip addresses
   2. prevented by, cryptographically verifies the server host identity
3. Connection hijacking
   1. SSH can't prevent hijacking, since this is a
weakness in TCP, which operates below SSH
   2. detects if session is modified in transit, and suts down the connection.
4. Man in the middle attacks
   1. stopped by server host authentication, its important that the client checks the server-supplied public host key against known host, client can be spoofed during the first connection
   2. limit authentication methods vulnerable to this attack
5. Insertion attack
   1. dues to weak integrity checking algorithms

### Threats SSH doesn't prevent

- Password cracking:
  - Use public-key instead, avoid strange computers
  - Nevertheless, a password is still a weak form of authentication, and you must take care with it. You must choose a good password, memorable to you but not obvious to anyone else, and not easily guessable.
- IP and TCP attacks:
  - Ssh provides not protection against attacks that break or prevent setup of TCP connections e.g SYN flood
- Traffic analysis:
  - ssh port is widely known, as such its easy to monitor amount of data, source and destinations of data as well as timing.
- Covert channels: Means of signaling information in an unanticipated and unnoticed
fashion.
- Carelessness
  - Mit der Dummheit kämpfen Götter selbst vergebens.
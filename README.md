# Enclaves
 
 An instant messaging system that ensures security through:
  - End-to-End encyption with AES-256 encryption during transit, and at rest on the routing server.
  - User Interface enforced procedures for new user on-boarding that protect against man-in-the-middle attacks.
  - The server and client implementations being completely open-source so that anyone can review the source code, and build, run and operate their own encrypted instant messaging server.

## 1. Introduction
Communication on the internet between ordinary people has come to depend almost exclusively on centralized instant messaging systems that are operated by third party organisation. At the very best these instant messaging systems are open source, but without access to the operator’s servers, it is impossible to verify any claims regarding to what data is collected about the users. Users must trust that service providers are being honest about the security and privacy of their systems, and are building and deploying the applications whose source code they publish when they are open-souce. 

What is needed is an instant messaging system that secures its users communication secrecy through; end-to-end message encryption between users, UX enforced procedures for adding users to the system that protects against man-in-the-middle attacks, and, both server and client being completely open source so that any group of individuals or organisation can review the source code, then download it and build, run and operate their own self-contained messaging service

## 2. Design
### 2.1 Encryption Scheme
Enclaves uses a messaging protocol that uses a unique AES-256 encryption and authentication key pair for each user to user conversation. A message to another user is encrypted using the AES-256-CBC algorithm with randomised initialisation vectors and a unique encryption key for that user to user communication. This encypted message is then appended with an authentication code generated with the AES-256-CBCMAC algorithm, using a separate authentication key that is unique for this communication.

The routing server contains a table containing for each user an identifier number, encryption key and authentication key, this is the only information stored by the server. Messages sent between the user and the server contain only sender identifier number, recipient identifier number, and the encrypted and authenticated user to user message which the server can't decrypt. The server simply receives the message, and forwards it on to the recipient.

#### 2.1.1 User 1 Actions
![User 1 Actions](https://github.com/will-rigby/Enclaves/blob/main/images/User%201%20Actions.PNG?raw=true)

#### 2.1.2 Server Actions
![Server Actions](https://raw.githubusercontent.com/will-rigby/Enclaves/main/images/Server%20Actions.PNG)

#### 2.1.3 User 2 Actions
![User 2 Actions](https://github.com/will-rigby/Enclaves/blob/main/images/User%202%20Actions.PNG?raw=true)

### 2.2 Man-In-The-Middle Defense
While Diffie-Hellman (DH) key exchange protects against eavesdroppers, particularly if the keys used are large enough, Man-in-the-middle attacks are difficult to prevent against. If a well-prepared attacker is intercepting all communication to the server and have sufficient knowledge of the messaging scheme, they could hijack any key exchanges when a new user joins the server. The best protection against man-in-the-middle attack is by using trusted channels to share keys. In the design of Enclaves servers, users receive their server-user key-pair direct from the servers terminal or GUI interface, or they can receive it directly from another "trusted" user, with the new user connecting to the server using the key-pair they received from the server via their "trusted" friend, and immediately generating a new key-pair their trusted friend is not a party to.

When a user joins the server for the first time, and is connected securely with their unique user-server encryption & authentication keys, the server shares with the user, the IDs of all the existing users. The new user will immediately initiate Diffie-Hellman key exchange with each existing user, to obtain unique user-to-user encryption and authentication key pairs, over this secure encrypted and authenticated channel through the routing server. However if users can meet in person, or use other channels such as phone calls, then they can directly share their DH public keys with each other and generate new AES-256 user-user encryption and authentication key pairs without relying on doing key-sharing through the server.

If a server is compromised by an attacker gaining possession of it, any existing users' communication is already encrypted and authenticated and their communication between each other will remain secure, no matter how the attacker tries to modify the server. However any users who join the server after the security is compromised will have all their conversations to new or existing users compromised if the attacker modifies the server to hijack the DH key sharing and record the encryption and authentication key pairs, if the public keys for the DH key exchange is done through the hacked server. If the users DH key exchange is done in person or by an authenticated (but not secure) channel such as a telephone call, the attacker maybe able to eavesdrop the key share (and potentially obtain the key if RSA encryption is broken in the future), but they will not be able successfully obtain the AES-256 keys through a man-in-the-middle attack.

If a user is compromised and their user software is modified then obviously any conversations with that user will compromised, but any new users who use the compromised user to join the server by obtaining their user-server AES-256 encryption and authentication key pairs can be compromised if the attacker is also able to do a man-in-the-middle attack on the server. The compromised user after receiving the AES-256 key pair for the new user and giving it to them, if they can intercept communication between the new user and the server can perform a man-in-the-middle attack on all future DH key sharing between that new user and any other user on the server. However if the new user does the DH Key Sharing with new users in person or over the phone, then this will defeat the attacker.

To summarise, the design of Enclaves means the communication can only be successfully attacked if either the server or a user are compromised (by either the user's computer or the server being hacked or in the possession of the hacker). In either case this will only break the security of communication with users who joined the server after it was compromised, and users who join after the server was compromised can defeat this attack if they do the DH key-exchange in person. Users who joined before the compromise will be unaffected when communicating with other users who joined before the compromise.

### 2.3 Server Application
The enclave server itself uses an application encryption-key to store its data at rest, to further protect users from third parties. While the user-user conversation keys ensure that if a server is compromised the pending messages are unable to be read by the party in control of the server, this also ensures that if the third party is unable to obtain this server encryption key, they cannot obtain the server-user encryption keys or identify the users who sent or are being sent messages that are being stored at rest on the server. 

#### 2.3.1 User Data Storage
The user data is stored as an 80 byte file. The first eight bytes are an initialisation vector for the AES-CBC encryption algorithm used. Then next 32 bytes are the encryption key for that user, and the last 32 bytes are the authentication key. The file itself is a 64-bit integer in decimal format followed by a “.user” file extension. 

#### 2.3.2 Pending Message Storage
 The pending messages are stored in a similar format with the 16 Byte initialisation vector followed by a 64 byte Recipient Id and then a 64 byte Sender Id, then the message payload, this message has a file name that is a 64-bit integer in decimal format followed by a “.msg” file extension.

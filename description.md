# Overview of the Application
Overview:
This application simulates a secure communication channel between a client and a server, with a VPN acting as a man-in-the-middle to forward encrypted messages between the client and the server. The client and server communicate using Transport Layer Security (TLS) to ensure that their messages are encrypted and authenticated. The Certificate Authority (CA) signs the server's certificate, enabling the client to verify the server’s identity before initiating secure communication.

Client: Initiates the connection, requests the server’s certificate, and exchanges messages securely using a symmetric key.
Server: Responds to the client, processes messages, and uses the symmetric key for secure communication.
VPN: Acts as a relay between the client and server, forwarding messages without reading or altering them.
CA: Signs the server’s certificate to establish trust.

# Format of an Unsigned Certificate
Unsigned Certificate Format: used to represent server's public key, IP address, and port number before it is signed by the CA.
Format:
<public_key>|<server_ip>|<server_port>
Example:
If the server's public key is (15178, 56533), its IP is 192.168.1.1, and its port is 8080, the unsigned certificate would look like:
(15178, 56533)|192.168.1.1|8080

# Example Output
Client Output:
client starting - connecting to VPN at IP 127.0.0.1 and port 55554
Encoded message: 192.168.1.10~IP~8080~port~hello
Sent encoded message to VPN: 192.168.1.10~IP~8080~port~hello
message sent, waiting for reply
Received raw response: 'olleh' [16 bytes]
Decoded message 'olleh' from server
client is done!

VPN Output:
VPN starting - listening for connections at IP 127.0.0.1 and port 55554
Connected established with ('127.0.0.1', 56800)
Received client message: 'b'192.168.1.10~IP~8080~port~hello'' [47 bytes]
Parsed message: 
  SERVER_IP: 192.168.1.10
  SERVER_PORT: 8080
  MESSAGE: Hello, Server!
Connecting to server at IP 192.168.1.10 and port 8080
Server connection established, sending message 'hello' to the server
Received server response: 'b'Message received: olleh'' [31 bytes]
Forwarding server response to client
VPN is done!

Server Output:
server starting - listening for connections at IP 192.168.1.10 and port 8080
Sending signed certificate '(15178, 56533)|192.168.1.10|8080' to the client
Received encrypted symmetric key 'EncryptedKey123' from the client
Decrypted symmetric key: 12345
Responding 'olleh' to the client
Sending encoded response 'olleh' back to the client
server is done!

CA Output:
Certificate authority starting - listening for connections at IP 127.0.0.1 and port 55553
Connection established with ('127.0.0.1', 56789)
Received client message: 'b'key'' [3 bytes]
Sending the certificate authority's public key (56409, 56533) to the client
Received client message: 'b'$((15178, 56533)|192.168.1.10|8080)'' [43 bytes]
Signing '((15178, 56533)|192.168.1.10|8080)' and returning it to the client.
Received client message: 'b'done'' [4 bytes]
('127.0.0.1', 56789) has closed the remote connection - listening

# TLS Handshake
TLS handshake establishes a secure communication channel by exchanging certificates and generating a shared symmetric key. Below are the steps:
1. Client Requests the Server’s Certificate:
The client requests the server’s certificate to verify its identity.
2. Server Sends Its Certificate:
The server sends its signed certificate (including the public key) to the client, which proves the server's identity and allows the client to verify that it is communicating with the correct server.
3. Client Verifies the Server’s Certificate:
The client checks the server’s certificate against the public key of the Certificate Authority (CA). If the certificate is valid, the handshake continues.
4. Client Generates a Symmetric Key:
The client generates a symmetric key, encrypts it with the server's public key, and sends it to the server. This symmetric key will be used to encrypt and decrypt subsequent messages between the client and server.
5. Server Decrypts the Symmetric Key
The server uses its private key to decrypt the symmetric key sent by the client. The server can now use the symmetric key to securely communicate with the client.
6. Secure Communication Begins:
From this point on, the client and server communicate using the symmetric key, ensuring confidentiality and integrity of the messages. Messages between the client and server are now secure, and the VPN cannot read or alter the messages.

# Two Ways in Which Our Simulation Fails to Achieve Real Security, and How These Failures Might Be Exploited
1. The client fetches the CA's public key over an insecure channel. An attacker could intercept the CA’s public key during transmission, substitute their own key, and intercept or manipulate messages between the client and server. This type of Man-in-the-Middle (MITM) attack would compromise the entire TLS handshake.
2. The server uses eval() to parse the certificate (specifically the server's public key). An attacker could craft a malicious certificate with harmful Python code embedded, which would be executed by eval(). This could allow arbitrary code execution on the server, compromising the system's security.

# Acknowledgements
I worked with Zhirou Liu on this project.

# Command-Line Traces
Client Trace:
python secure_client.py --server_ip 192.168.1.10 --server_port 8080 --message "hi"
client starting - connecting to VPN at IP 127.0.0.1 and port 55554
Encoded message: 192.168.1.10~IP~8080~port~hi
Sent encoded message to VPN: 192.168.1.10~IP~8080~port~hi
message sent, waiting for reply
Received raw response: 'ih' [16 bytes]
Decoded message 'ih' from server
client is done!

VPN Trace:
VPN starting - listening for connections at IP 127.0.0.1 and port 55554
Connected to client
Received message from client: 192.168.1.10~IP~8080~port~hi
Forwarding message to server at 192.168.1.10:8080
Received server response: 'ih'
Forwarding server response to client
VPN is done!

Server Trace:
server starting - listening for connections at IP 192.168.1.10 and port 8080
Sending signed certificate '(15178, 56533)|192.168.1.10|8080' to the client
Received encrypted symmetric key 'EncryptedKey123' from the client
Decrypted symmetric key: 12345
Responding 'ih' to the client
Sending encoded response 'ih' back to the client
server is done!

# OSI Model
> Created: 2023-06-13 17:46

Seven layers that computer systems use to communicate over a network. The first standard communication model.

Modern internet is **not based on OSI** but on the simpler TCP/IP model. However OSI is still widely used.

## The OSI 7 Layers

![[OSI-7-layers.jpg.webp]]

### Application Layer

+ Used by end user software.
+ Protocols to send and receive information, present meaningful data to users.
+ HTTP, FTP, POP, SMTP, DNS.

### Presentation Layer

+ Prepares data for the application layer
+ Defines how devices encode, encrypt, and compress data so it is received correctly on the other end.

### Session Layer

+ Creates communication channels, called sessions.
+ Responsible for opening sessions, ensuring they remain open and functional while data is being transferred, and closing them when communication ends
+ Checkpointing -- if a 100 megabyte file is being transferred, the session layer could set a checkpoint every 5 megabytes. In the case of a disconnect or a crash after 52 megabytes have been transferred, the session could be resumed from the last checkpoint, meaning only 50 more megabytes of data need to be transferred.

### Transport Layer

+ Breaks session layer data into "segments", these are reassembled on the receiving end.
+ Flow control -- controlling data rate.
+ Error control -- checking if data was received incorrectly (and sending it again if so)
+ TCP/UDP

### Network Layer

+ Breaking up segments into network packets (and reassembling them)
+ Routing packets by discovering the best path across a physical network
+ Uses network addresses (typically IP address) to route packets
+ IP

### Data Link Layer

+ Establishes and terminates a connection between two physically connected nodes on a network.
+ Facilitates data transfer between two devices on the _same_ network
+ Breaks up packets into *frames*.
+ Two parts
	+ Logical Link Control (LLC): identifies network protocols, performs error checking, syncs frames
	+ Media Access Control (MAC): uses MAC addresses to connect devices and define permissions.

### Physical Layer

+ Physical cable / wireless connection.
+ Defines the connector etc.
+ Responsible for bit rate control.

## OSI vs TCP/IP Model

![[OSI-vs.-TCPIP-models.jpg.webp]]

TCP/IP is **older than** the OSI model. Collapses several OSI layers into one.

TCP/IP does **not take responsibility** for sequencing and ack functions, leaving those to underlying transport layer.

Other important differences:

- TCP/IP is a functional model designed to solve specific communication problems, and which is based on specific, standard protocols. OSI is a generic, protocol-independent model intended to describe all forms of network communication.
- In TCP/IP, most applications use all the layers, while in OSI simple applications do not use all seven layers. Only layers 1, 2 and 3 are mandatory to enable any data communication.

----

## References
1. https://www.imperva.com/learn/application-security/osi-model/
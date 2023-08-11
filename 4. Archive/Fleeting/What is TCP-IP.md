# What is TCP/IP?
> Created: 2023-06-13 18:25
> Related: [[OSI Model]]

IP:
+ address system of the Internet and has the core function of delivering [packets](https://www.cloudflare.com/learning/network-layer/what-is-a-packet/) of information from a source device to a target device
+ IP does not handle packet ordering or error checking
+ Such functionality requires another protocol, often the Transmission Control Protocol (TCP).

IP makes sure the pieces arrive at their destination address. TCP can be thought of as the puzzle assembler on the other side who puts the pieces together in the right order, asks for missing pieces to be resent, and lets the sender know the puzzle has been received. TCP maintains the connection with the sender from before the first puzzle piece is sent to after the final piece is sent.

IP is a connectionless protocol, which means that each unit of data is **individually addressed and routed** from the source device to the target device, and the target **does not send an acknowledgement** back to the source. That’s where protocols such as TCP come in. TCP is used in conjunction with IP in order to **maintain a connection** between the sender and the target and to **ensure packet order**.

----

## References
1. https://www.cloudflare.com/learning/ddos/glossary/tcp-ip/
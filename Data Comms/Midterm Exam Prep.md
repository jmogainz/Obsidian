All devices in IoT are hosts or end systems.
    End systems - sit at edge of internet
        Host Web browser program, Web server program, email client program, email server program
            Clients and servers
    Access the internet
        Access network is the network that physically connects an end system to the first router on path to destination
        Two most prevalent types of broadband residential access - DSL (digital subscriber line), and cable
            DSL typically comes from local telephone company (telco) 
                DSL modem uses existed telephone line (twisted-pair copper wire)
                    Modem converts digital to high Hz analog tones, for transmission over wire
                Exchanges data with DSLAM (access multiplexer) located in telco's central office
                    DSLAM converts back to digital, sends to internet, also separates phone from data
                    Customer: splitter divides phone from data, sends data to DSL modem
                Data and phone lines encoded with different frequencies (all share same line)
                Maximum rate limited by distance (should be 5 to 10), gauge of copper line, and degree of e-interference
            Cable makes use of Cable television infrastructure
                Fiber-optics connect the cable head end to neighborhood-level junctions (500 to 5000 homes)
                    Coaxial is then used to reach individual houses and apartments
                    HFC (hybrid fiber coax)
                    Cable modem connects to PC through ethernet port
                    CMTS (similar to DSLAM), cable modem termination system, A2DC
                    All downloads come from same cable head (fiber node), bandwidth can be slow if downloading at same time
            FTTH: Fiber to the home
                AONs and PONs: Verizon uses PON, less than a 100 homes on 1 fiber optic cable
                    Homes have a ONT (optical network terminator)
                    Head has a OLT (optical line terminator), OLT converts optical to electrical signals
                Can direct ethernet from Modem (ONT), or use router wirelessly
            Satellite Link: StarBand or HughesNet
Different links can transmit data at different rates (bits/sec)
Packets are segments of data with header strapped on top
    Packets - trucks
    Comm links - highways and roads
    Packet switches - intersections
    Hosts - buildings
Most common Packet Switches - routers and link-layer switches
    Link-layer switches - access networks
    Routers - network core
Sequence of comm links and packet switches is route or path
Each ISP is in itself a network of packet switches and comm links
    Provide cable modem or DSL (high-speed local area network access)
    Mobile wireless access
    Lower Tier ISP are interconnected through nation and international upper-tier ISPs (Level 3 Comms - Verizon and NTT)
        Consist of high-speed routers interconnected with high-speed fiber-optic links
    Managed independently, runs IP, conforms to certain naming and address conventions
IP protocol specifies the format of the packets that are sent and received among routers and end systems
Internet's principal protocols are known as TCP/IP
IETF develops Internet Standards form RFCs
    IEEE 802 LAN/MAN Standards Committee specified Ethernet and wireless WiFi Standards
    It is import for systems to interoperate, hence Standards
Distributed apps - involve multiple end systems that exchange data with each other
End systems attached to the Internet provide a socket interface
    Set of rules that the sending program must follow so Internet can deliver to destination
    Internet provides services for your application
Protocols
    hardware-implemented protocols in two physically connected computers control the flow of bits on the “wire” 
    between the two
    network interface cards; congestion-control protocols in end systems control the rate at which packets
    are transmitted between sender and receiver; protocols in routers determine a packet’s path from
    source to destination


Ethernet is common LAN technology to connect end system to edge router
    Connect to ethernet switch
        Servers may get 10 to 100x more speed than users on Ethernet switch
Wireless LAN (WiFi) is alternative, have to be within distance to work well
    Base Station is wireless access point, modem provides access to cable internet, router connects base station with with modem

Store and forward delays and queuing delays - packet switches (queuing may depend on congestion)
Packet loss will occur if queue is full (either new or old)
Use Forwarding table to send on different comm links

Total Nodal Delay - Nodal processing delay + queuing delay + transmission delay + propagation delay
Processing delay - time required to examine packets header and determine outbound link
    Also time to check for bit-level errors
    Microseconds
Queuing delay - Dependent on earlier arrived packets
    If no other packet is being transmitted, and queue is empty, delay is 0
    Micro to Milliseconds
Transmission delay - R bits/sec. 10 Mbps Ethernet Link. L-bits. L/R is delay.
    Microseconds to milli
    Time required to push (transmit) all packet bits into the link
    (Has nothing to with distance between routers)
Propagation delay - time required to propagate a bit, propagates at speed of the link.
    Fiber, twisted-pair copper - range of 2*10^8 meters/sec to 3*10^8 meters/sec
    Less than or equal to speed of light
    Distance / speed of light
    Milliseconds 
    (Has nothing to do with packet length or transmission rate of link)

Traffic intensity - La/R (a is rate of packets arriving) (L is bits per packet) (R is transmission rate)
    If La/R > 1, rate of arrival is greater than rate leaving, queue starts to form
    Design system so traffic intensity is no greater than 1

Throughput is the actual download or bits/sec arrival speed the destination actually gets
    Bottle-necked by the channel with the minimum speed (in a situation where internet is accessed, typically the access network)

Application layer - HTTP (web document requests and transfer), SMTP (handles email), FTP (file transfers), DNS
Transport layer - Internet's transport layer transports application-layer messages between application endpoints.
    TCP and UDP
    Packet is a segment
    (conceptual technique for delivering reliable information)
Network layer - moving network layer packets
    Packet is a datagram
    TCP or UDP passes segment and destination address to network layer
    Provides service of delivering the segment to the transport layer in the dest
    IP protocol, routing protocols - determine routes and paths from source to dest
    Called IP Layer
    (THIS CAN BE THOUGHT OF AS NETWORK (multiple routers and end systems) THAT ACTUALLY CARRIES SEGMENTS)
Link Layer - delivers datagram to next node along route
    At each node, the link layer passes datagram back up to network layer
    Ethernet, WiFi, Cable access DOCSIS
        Datagram my be handled by several different link layer protocols throughout path
    Link layer packet is frame
Physical layer - moves individual bits within the frame from one node to the next
    Ethernet has many physical layer protocols, one for twisted-pair copper wire,
    one for coaxial cable, and another for fiber and so on. 
        In each case, bits are moved along the link in different ways


Web browsers implement the client side of HTTP
    Web servers implement the server side, house Web objects, each addressable by URL
        Apache and Microsoft Internet Information Server
    Browsers simply display objects returned from HTTP request

Cookies are stored in browser files, and are sent in http header to show server who use is
Persistent vs non-persistent connection
Web cache substantially reduces response time, and traffic to http server (link to internet, leads to congestion) (local storage of https responses)
    Cache may be stale, conditional GET verifies it is latest version
CDNs (content distribution networks) - distributes many geographically distributed caches throughout the Internet, localizing much traffic
    Dedicated CDN (google or netflix)
    Shared (Akamai or Limelight) (third party - distributes content on behalf of multiple content providers)
    Enter Deep and Bring Home design philosophies

Email composed of user agents, mail servers, and SMTP
    User agents - Outlook and Apple Mail - 
    Mail Servers - Alice mail server places message in outgoing queue
        - Bob has mailbox in recipient mail server that maintains and manages messages
        If message cannot be delivered, it sits in Alice's queue and retries until no more
    SMTP uses TCP
        SMTP server - receives mail on mail server
        SMTP client - mail server send mail

SMTP is push protocol
    TCP connection is initiated by sender
HTTP is pull protocol
    TCP connection is initiated by receiver

HTTP encapsulates text and images as separate HTTP response messages
SMTP places all of the message's objects into one message

SMTP cannot access messages from mail server (it is a push protocol)
    HTTP, IMAP, POP3

2.4 is DNS Servers

NO LONGER always-on infrastructure servers
P2P, owned by users, no service providers
    Takes work load off of server
Popular - BitTorrent, many torrent client conform to BitTorrent protocol
    Just as browsers conform to HTTP
Distribution time for P2P is much better, and more scalable (never more than 1 hour, for N number of peers)
    Peers help out in the distribution process
Torrent - collection of all peers participating in distribution (those who are seeds)
    Tracker - infrastructure node
        Peer joins a torrent - periodically notifies Tracker that it is still in Torrent
        Tracker sends subset of torrent to new member
        At any given time, each peer will have a subset of chunks from the file, with different peers having different subsets
        (in case people are leaving and entering)
        TCP connection, Alice will recursively find chunks that she does not have
            Rarest first is technique for finding all chunks fast as possible
            Alice will also give priority to those who are supplying data at the highest rate
                In return, those same people will get data from her
                This gaurantees highest rate data transfer
    Unchoked, optimistically unchoked, choked
    Load is balanced (files are replicated), easy to find info
        Dis: lots of network overhead, lots and lots of file copies, Not for secure things

CHORD DHT - Distributed hash table - distributing the indexing functionality over all peers, structured P2P network
    Scalable Peer-to-Peer Lookup Protocol for Internet Applications
    Key based search functionality
        identifier space [0, 2^M-1]
            Node identifier - secure hash function SHA-1 on IP address
                160 bit hash value
            Data identifier - secure hash function, SHA-1, on Content Name (torrent or song name)
        Pretty much RANDOM ids, no collision
        p's ID is p, is an M-bit binary string in identifier space
        succ(ID) - closest clockwise node ID
        p maintains link to closest clockwise neighbor q
            successor link
            allows for (inefficient) routing
        node p stores all data with identifiers between PREDECESSOR and itself
        forward request to successor until data is found
    
    Finger LINKs - finger link i to node at distance 2^i from p clockwise, [0, 1, ..., M-1]
        Finger table = routing table
        Greedy routing for k value k
            Repeat this until node holding item corresponding to k
                Use link which takes you closest to (but not past) k
                Forward request to that node

Messages broken down into chunks, tp-layer header to each chunk: segment, encapsulated within network-layer packet: datagram
    Routers do not examine the fields of tp-layer segment, only datagram
    On Receive, network layer extracts tp-layer, and passes to tp-layer of receiver
    UDP should not be "datagram"
    IP is not good enough, does not ensure - delivery, orderly delivery, integrity of delivery

Demultiplexing - delivering the data in tp-layer segment to the correct socket
Multiplexing - creating segments to send, encapsulated with dest socket and header info 

Port number 0-1023 are well known. Reserved for well-known application protocols (FTP and HTTP)
    16-bit number

UDP benefits (however it adds nothing to simple IP)
    Finer application-level control over what data is sent, and when
    No connect establishment
    No connection state
    Small packet header overhead
        Has 4 fields, 2-bytes each, src port, dest port, # of bytes (header plus data), checksum (rarely used) 

Reliable Data Transfer
    Bit errors: packet transmitted, propagates, or is buffered
        Error Detection
        Receiver Feedback
        Retransmission
        Wait for ACKSF
            ACKS corrupted
                resend: duplicate packets
        Add sequence number
            Receiver knows if retransmitted or not
            Only need 1-bit bc sender only puts one packet on channel at a time
        Duplicate ACKS if bad message
    Packet Loss 
        Wait time (judiciously chosen)
            May be too short, luckily we have sequence numbers

    Slow transfer (stop and wait is too slow, wasting channel capacity, and hardware capabilities)
        Pipelining
            Sequence number must be increased (for each in-transit packet)
            Buffering of packets (incase they need to be sent again)
            Go-Back-N and selective repeat
        Go-Back-N
            Sliding-window protocol
            N is window size
            Coumulative ack - x to n received
            Timeout event - full resend
            If received packet out of order, discards packet, and sends ack for prev packet
            No need to buffer out of order packets and bring in later,
                If n is lost, saving n+1 is useless, bc sender will resend both anyways
            Sender keeps track of head and tail and nextseqnum, receiver only needs expectedseqnum
            Window can slide forward with each successful ACK
            CAN LEAD TO HUGE MASSIVE RETRANSMISSION, which would then actually slow dictation
        Selective Repeat
            sender and receiver windows will not always coincide
            Page 266


        

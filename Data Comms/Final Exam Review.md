IP address 32-bits: 8bit-8bit-8bit-8bit
	Subnet(high), host(low)
	Subnet does not require intervening router
	Classful addressing - fixed length subnet
		8, 16, 24
		A, B, C
		2^8 = 256 networks possible: Ex
	    2^24 = 16, 777, 216 nodes
		WASTEFUL
	CIDR - Classless InterDomain Routing
		a.b.c.d/x (x is subnet portion bit#)
		Blocks of addresses allocated by ICANN
		167. 199.170.82/27
			First address: 167.199.170.64
			Last address: 167.199.170.95
			MASK: leftmost(1), remaining(0)
			First = addr AND mask
			Last = addr OR NOT(mask)

Network Layer
	Forwarding - between routers
	Routing - determine route between src and dest
Routing table
	Range of addr: output link
	longest prefix match
Packets at link layer dont have standard size (Packet Fragmentation)
	MTUs - limits the max amount of data at network layer
	Datagram may not fit in MTU
	Fragment and Reassemble
		Fragmented at src and reassembled at dest (in end systems)
		IP datagram headers have identification, flag, fragmentation offset fields
		Last frag has flag set to 0, IP has no built in reliability
		Routers have different size MTUs, and reassembly puts more strain on router performance
IP Datagram 
	20 bytes of IP and 20 bytes of TCP = 40 bytes of overhead
	Max is 65,535 bytes
	Slide 37 Lecture 13
	
DHCP
	All 1s, sends to everyone on subnet
	Routers ignore
	Client host selects longest lease time
	
NAT - network address translation
	Slide 48 Lecture 13
	Port number is linked to src addr and port
		So 16-port has lats of simultaneous connections all under one IP
	Routers shoulder only process up to layer 3
	Routers should not interfere with reliability and not violate end-to-end principle
	Should be solved by IPv6 instead

Classless Addressing, DHCP, and NAT conserve IPv4 space
	Resulted in slow-rate of adoption for IPv6

IPv6 Motivation
	Header format helps speed processing/forwarding
	Header changes to facilitate QoS (reduce packet loss, latency, and network reliability)

IPv6
	128-bit address vs 32-bit from 4
	It has much simpler header
	20 bytes for 4 and 40 bytes for 6 (header length)
	Only need payload length to determine datagram size
	1280 bytes are minimum fragmentation split (minimum handled by link layer)
	ICMP messages should alert source that datagram is too big
		ICMP is routers and network layer communication protocol
		Carried as IP payload, just like TCP and UDP
	Header: Slide 56 Lecture 13

Transition from 4 to 6
	Tunneling - IPv6 datagram carried as payload in IPv4 datagram among IPv4 routers

Routing Algorithm (Robustness Para. on pg. 440)
	General paradigm (not in itself an actual protocol)
	Dijkstras's (LS algorithm) ![[Pasted image 20220505000920.png]]
	Link costs are known to all nodes
		Flooding network information to all nodes so they all have the same info
			GLOBAL routing algorithm
	Computes least cost paths from one source node to all other nodes
		Hence, forwarding table for that node

Flooding
	All nodes keep table of (node, cost) pairs
	Initially, costs are all infinity except for node itself and neighbors
	Each node does:
		Receives older table, ignores it
		Receives newer table, keeps it and throws out old (new nodes discovered can be added)
		If new info, propagate to all other neighbors (besides who sent)
	Eventually converge if system is not too dynamic

Distance Vector Algorithm
	Another paradigm
		Distributed routing
		Uses Bellman-Ford
	From time-to-time, each node sends its own DV to neighbors
	When x receives new DV estimate from neighbor, it updates its own DV using B-F
	Slide 12 Lecture 15
	Iterative, asynchronous
	Each local iteration caused by:
		Local link cost change
		DV update message from neighbor
	Distributed:
		Node notifies neighbors only when its DV changes
		Neighbors then notify their neighbors if necessary
	(Link state - each router tells WHOLE network about its neighbors)

Autonomous Systems Motivations
	Page 442
	Gateway routers go to inter AS
	Most common Intra:
		RIP: Routing Information Protocol
			Bellman-Ford based
		OSPF: Open Shortest Path First
			Dijkstra Based
	RIP:
		Slide 24 and up Lecture 15
		DV protocol
		Distance metric: # hops
		Max hops = 15
			limit use to ASes with diameter <= 15
		DVs exchanged with neighbors every 30 sec roughly
			UDP, port 520
		Request messages - used by routers that just initialized or had time-out for some entries
		Response messages - sent back to answer a request, OR every 25-30 seconds (random periodic timer)
		Router pushes updates to neighbors
		If there is no advertisement heard after 180 sec, then neighbor/link is declared dead
			Routes via neighbor invalidated
			New advertiesements sent to**** neighbors
			Neighbors in turn send out new advertisements
	OSPF:
		Slide 25 Lecture 155
		LS packet dissemination
		Route computation using Dijkstra's algorithm
	Link State Advertisements
		Announcing link costs
		Announcing attacked networks
		Cost metric for setting edge weights (celing of ref. BW (100 Mbps) divided by interface BW)
	Inter-AS responsibilites:
		Learn which dest are reachable through AS2 or AS3 (ex)
		Propagate this reachability info to all routers in AS1
		BGP - Border Gateway Protocol:
			AS operators can purchase sevice from each other (transit service)
			Shared agreement (peering)
			Each AS uses a common intra-domain protocol (rip or ospf)
			Each router knows how to navigate own AS
			Does not know how to reach other AS networks
				BGP needed at every border router (external)
					Establishes routes between ASes
				BGP needed internally as well
					Prop BGP routes from one side of AS to other (and sent to next ISP)
					
Link Layer
	First layer that we see node-to-node (not end-to-end)
	CONCEPTUALLY divided into two sublayers ->
	DLC - data link control - addresses issues common to both point-to-point and broadcat links such as error
	detection
	MAC - media access control - deals with broadcast (shared) links specifically
	Error Detection and Correction
		Single-bit error
		Burst error - noise for a period
		Parity Check - add 1 additional bit to dataword to make codeword (number of 1s is now even)
			Detects odd number of errors (will miss if errors are even)
		2D Parity Check - Slide 41 and up Lecture 15
			Still cant determine where error occured (could be in any row)
		CRC - cyclic redundancy check - m + r for codeword
			Slide 42 Lecture 15
			Append r 0s to dataword
			Divide by generator (r + 1) value mod 2
			Quotient is discarded
			Dataword concatenated with remainder is codeword
			(Dividend must have as many (non-trivial) bits as divisor)
			Decode - divide codeword by G; if R = 0, then accept. Else, reject.
			GENERATOR is known by both
				DETECT:
					All single-bit errors
					Any 2 errors
					All odd # of bit errors
					All burst errors of length at most r
					Used in Ethernet, 802.11, and Bluetooth
			Frame check sequence in 802.11
	Multiple Access Protocols:
		Random Access:
			ALOHA
			CSMA/CD
			CSMA/CA
		Controlled Access Protocols:
			Reservation
			Polling
			Token Passing
		Channelization Protocols:
			TDMA
			CDMA
			FDMA
		GOAL: One node sends at R bps, Multiple sends at R/M bps
			Fully decentralized - no special node to coordinate transmissions - point of failure
			SIMPLE! (Do not be overcomplicated)
		Random:
			No a priori coordination among nodes
			Specifies
				Detect collisions
				Recover from collisions
			Slotted ALOHA:
				![[Pasted image 20220504233705.png]]
				Pros:
					Single Active node can continuously transmit at full rate of channel
					Highly decentralized
					Simple
				Cons:
					Collisions often
					Empty slots
					Clock synchronization
				Each node transmits in each slot with PROB p
				Efficiency is PROB that ANY node succeeds
					Slide 40 Lecture 16
					At best: channel is used for useful transmissions 37% of the time
			Pure ALOHA:
				Simpler, no synchronization
				When frame first arrives, transmit immediately
				Collision probability increases (left over transmission from halfway through prev slot)
				At best: 18% of the time, success
			ALOHA causes lots of collisions
			Carrier Sense Multiple Access (CSMA):
				LISTEN BEFORE TRANSMISSION
					Then transmit entire frame
				Collisions still occur due to propagation delay
					Nodes may not hear each others transmission before sending
				Entire packet transmission time is WASTED
				ENTER /CD (Collision Detection):
					Detects collisions within short time
					Colliding transmissions aborted, reducing channel wastage
					Easy in wired LANS: 
						measure signal strengths
						compare transmitted and received signals
					Difficult in wireless LANS:
						Received signal strength overwhelmed by local transmission strength
					Deferral time - if collide, abort and resend after a random amount of time
					Use randomized algo - binary exp. backoff
					BEB is slide 36 in Lecture 17
						Takeaway:
							Gets at least Omega(1/log n) throughput
							Medium access control with deadlines and robust to disruption (active research)
	Addressing:
		Router interface has both IP addresses and link-layer address
		MAC - 48 bits - HEX (8 bytes:8:8: so on)
			Move frames in LAN
		MAC address allocation administered by IEEE
			Manufacturer buys portions of MAC address space (to assure uniqueness)
		DNS used for IP
		ARP used for MAC
			ARP packet is broadcasted as frame with FFFFFFFF MAC address
			Intended receiver responds after recognizing its IP address
				A may send B ARP packet over MAC broadcast using B's IP address
				B replies with his own MAC
				A caches IP-to-MAC address pair in its ARP table until information becomes old
			ARP is "plug-and-play"
			Nodes create their ARP tables without intervention from net admin
			Send OUTSIDE of LAN
				A sends to router R an A-to-B datagram encapsulated in frame R's MAC address as dest
				R decapsulates frame, knows B's MAC, and encapsulated datagram inside frame with B's MAC as dest address
	Ethernet:
		Dominant wired LAN technology
		Frame:
			Preamble: 8 bytes of 0s and 1s (alternating, except for last byte)
			Dest/Source Addresses: 6-byte link-layer address of destination and source
			Type: 2 bytes for specifying upper-layer protocol, changed to length for 802.3 standard
			Data: 1500 bytes maximum (min 46 bytes for min frame length of 64)
			CRC: 4 bytes
		Each station in ETH network: has network interface card
			provides MAC address
		Uses a Bus Config: Everything is technically broadcast; all stations receive everything
		Traditional MAC protocol used: CSMA/CD
			Connectionless: no handshaking between NICs (network interface cards)
			Unreliable: receiving NIC does not send acks or nacks to sending NIC
	Switches
		Store and forward frames
		Examine incoming MAC addresses
		Selectively forward from to one-or-more outgoing links when frame is to be forwarded
		Plug-and-play, self-learning
			Switches do not need to be configured
		Flooding is used when switch table is empty
		Learns else:
			Frame received carries sender location; switch thus learns MAC, interface, and sets TTL
		Unknown location:
			Send broadcast
		Known:
			Send on interface link
		Interconnect hosts with Ethernet switch
			Star config
				Switch in center with each spoke running ethernet protocol (nodes do not collide)
	Wireless LANs
		Issues
			Medium is frequency spectrum
			No longer point-to-point, transmission is necessarily broadcast (exception with directional antennas)
			Less wires (good), but not being physically connected to Internet is issue of mobility
		Wire LAN was connected by switch (then router to outside world)
		Wireless is connected via AP (base station) (then router to outside world)
		Wirelss env. is more error prone
			Attenuation (fading) (distance increases signal weakness)
			Interference (other sigs and senders)
			Multipath Propagation
				Signals are reflected off objects
				Destructive interference
		Problems with CSMA/CD:
			Detect Collisions: host needs to send and receive simultaneously
				many wireless devices have single transceiver (radio)
				transmission drowns out ability to detect other signals
			Detecting Collisions HARD
			Deciding idleness is HARD too, but not impossible like detecting collisions
				WSNs (Wireless Sensor Networks) 
					received signal strength indicator (RSSI) to perform clear-channel assessment (CCA)
				Even if device decides channel is idle, interference may still be too high for reliable trans.
			Enter DCF
		DCF
			Key MAC technique for IEEE 802.11
			Uses CSMA/CA and backoff
			Comp 1
				Channel is idle, wait for interframe space (IFS)
			Comp 2
				Contention window
			Comp 3
				Since there can still be collisions
				WE NEED ACKS
			Each station:
				Wait for DIFS (distributed interframe space) and send frome if still idle
				2.) Else, use BEB to choose random backoff countdown. Freeze while channel sensed busy. Send when counter reaches 0.
				Wait for ACK. If received, restard CSMA/CA. Else, re-enter BEB and try again at 2.
			Destination station:
				Provide 'some' reliability
				If frame received, then wait for SIFS (short inter-frame spacing)
				Send back ACK
			Hidden Term and RTS/CTS (studied up)
			CCA: is the channel clear
				Carrier sense and energy detection (physical layer mechanisms)
			NAV: Network Allocation Vector
				Calculated wait time that channel is in use
				Uses data stored in MAC frame (maybe provided by RTS/CTS)
				Virtual Carrier Sensing
		Frame Types:
			Control
				channel acquisition, carrier sensing, acknowledgements
			Data
				pack horse moving data from station to station
			Management
				join and departures from network, mobility issues
		APs
			Has service set ID (SSID) and assigned channel number
			Issues Beacon frames periodically
				client responds with association request frame
				AP responds with association response frame
		802.11 has 11 available channels
			Industrial, Scientific, Medical (ISM) band
			non-overlapping if separated by 4 (exclusive) channels
			start at channel 1, how many non-overlapping channels?
				three non-overlapping channels exist (typicall 1, 6, 11)
			Frame
				Slide 41 Lecture 20
				Stores Addr 1 - Dest MAC, 2 - AP's MAC, 3 - Router's interface MAC
				Addr 3 allows AP to determine appropriate destination MAC when constructing ETH frame
	Mobile User of BSS
			Slide 22 Lecture 21
			Moving between two BSSs connected by a switch
				part of the same subnet
				can keep same IP address
			Moving between two BSSs connected by a router
				different subnets
				must obtain new IP
			Problems with Scenario 1
				Switch now has a false entry (interface tied to MAC is wrong)
				SOLVE: AP2 broadcasts ETH frame with H1's src addr
					Switch updates and fixes entry
				OVERALL: No Problem
					Not mobile from network layer perspective
					If same AP, not even from link layer perspective
				Wireless in a car?
				See slides 25 Lecture 21
				
Big Picture 
	Lecture 21
	Note that DHCP servers send out DNS server IPs as well
	DNS is port 53
	
Mobile Telecommunications
	MTSO - switching office
		connects calls between mobile units
	PUBLIC -> MOBILE -> Base Station Controller -> Base Transceiver -> Mobile User
	BS includes the last 3
		Control Channels - exchange info for call setup and maintaining calls
		Traffic channels - carry voice or data connection between users
	MTSO is also connected to fixed subscribers in telecomms network
		Collects info for billing
		Performs handoffs
		Handles call setup
	Call Setup
		Mobile Unit scans for strongest setup control channel
		Handshake between MTSO and MU
			Identify MU and register location
		Scanning is always happening
		MU wants to call...wait for idleness
		Send number of called unit to BS
		BS forwards to MTSO
		How does MTSO find called unit?
			Worst case: pages all BSes
		MU respond to page, BS forwards to MTSO, MTSO sets up circuit between BSs
			Selects traffic channels in each BSs cell
		Each BS notifies respective MU when call is complete link
		Signal goes from BS to MTSO to BS
	AMPS (1G)
		SLide 50 Lecture 22
		And so on
	ME includes transceiver and SIM card
		SIM stores subscriber info and encryption keys
	GSM (2.5G) just needs SIM, no phone similarity
	GSM was popular bc of swappable nature of sim Card
	Each BS defines a cell up to 35 km
		Responsibilities
			Reserving Radio frequencies
			Managing handoffs
	NS (Network Subsystem) Also known as GSM core network:
		Provides link between cellular net and public switched telecomm net
		controls handoffs between cells
		authenticates users
		MSC (Mobile switching center)
			4 databases
			HLR and EIR (Home location registry and Equipment identity registry)
				HLR - stores SIM data (stores associated telephone number)
				EIR - tracks type fo eequipment that exists at mobile station, can block illegitimate equip
				VLR (Visitor Location registry) - stores data on roaming subs currently within region covered by MSC
					Inbound call, MSC locate sub via HLR
					Outbound call, VLR initiates
				AuC (Authentication center database) - holds auth/encry keys for all subs in HLR and VLR
				User joins, first authenticated, then records created for HLR/VLR
	GSM uses CC, MM, RR
	GPRS (2.5 G)
		Creating datagram switching capability
			Connectivity to internet
			Roughly 100 kbps, enabled MMS
	4G
		63 Lecture 22
	Two candidates for 4G
		LTE and 802.16 WiMax
	LTE
		Overall architecture is called the Evolved Packet System (EPS)
		Consists of Radio access net + Evolved Packet Core
BUNCH OF RANDOM FACTS OF MOBILE COMMS IN LECTURE 22
CRYPTOGRAPHY INFO IN LECTURE 23
LECTURE 24 IS MORE CRYPTOGRAPHY STUFF, and SSL
	
	
	
			
	
			
			
			
			
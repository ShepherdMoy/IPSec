# IPSec Configuration 
Secure your site to site connection over the internet with IPSec .
In this example :

Isakmp phase 1
----------------------------
crypto isakmp policy 10
 encr aes
 authentication pre-share
 group 5

Isakamp Phase 2 (the IPSec tunnel)
-----------------------------------
crypto ipsec transform-set BRANCH_B esp-aes esp-sha-hmac
 mode tunnel

The access list defining the interesting traffic 
------------------------------------------------
ip access-list extended BRANCH_B_HQ
 permit ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255

The Crypto Map ,tying it all together ,the 
-------------------------------------------
crypto map CRMAP 10 ipsec-isakmp (the map with phase 1 and phase 2 attributes )
 set peer 15.0.0.1            (the peer aka the end encypting/decrypting point  )
 set transform-set BRANCH_B   (the transform set sedifened in pahse 2)
 match address BRANCH_B_HQ    (the NAT rule defined for the interesting traffic to be encypted while traversing the tunnel  )

 the results 
 ------------
 Branch_B#show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
25.0.0.1        15.0.0.1        QM_IDLE           1001 ACTIVE

IPv6 Crypto ISAKMP SA

----------------------------------------
Branch_B#show crypto isakmp peers
Peer: 15.0.0.1 Port: 500 Local: 25.0.0.1
 Phase1 id: 15.0.0.1
Branch_B#


-----------------------------------------
Branch_B#show crypto isakmp sa detail
Codes: C - IKE configuration mode, D - Dead Peer Detection
       K - Keepalives, N - NAT-traversal
       T - cTCP encapsulation, X - IKE Extended Authentication
       psk - Preshared key, rsig - RSA signature
       renc - RSA encryption
IPv4 Crypto ISAKMP SA

C-id  Local           Remote          I-VRF  Status Encr Hash   Auth DH Lifetime Cap.

1001  25.0.0.1        15.0.0.1               ACTIVE aes  sha    psk  5  23:25:03
       Engine-id:Conn-id =  SW:1

IPv6 Crypto ISAKMP SA

Branch_B#



-------------------------
Branch_B#show crypto ipsec sa

interface: GigabitEthernet0/0
    Crypto map tag: CRMAP, local addr 25.0.0.1

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (192.168.2.0/255.255.255.0/0/0)
   remote ident (addr/mask/prot/port): (192.168.1.0/255.255.255.0/0/0)
   current_peer 15.0.0.1 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 199, #pkts encrypt: 199, #pkts digest: 199
    #pkts decaps: 199, #pkts decrypt: 199, #pkts verify: 199
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 25.0.0.1, remote crypto endpt.: 15.0.0.1
     plaintext mtu 1438, path mtu 1500, ip mtu 1500, ip mtu idb GigabitEthernet0/0
     current outbound spi: 0x48BAC7D3(1220200403)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x18906F9D(412118941)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 1, flow_id: SW:1, sibling_flags 80000040, crypto map: CRMAP
        sa timing: remaining key lifetime (k/sec): (4224103/1454)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0x48BAC7D3(1220200403)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 2, flow_id: SW:2, sibling_flags 80000040, crypto map: CRMAP
        sa timing: remaining key lifetime (k/sec): (4224103/1454)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     outbound ah sas:

     outbound pcp sas:
Branch_B#


End result :Ping from HQ LAN 
------------------------------
HQ#ping 192.168.2.1 source 192.168.1.1 repeat 10
Type escape sequence to abort.
Sending 10, 100-byte ICMP Echos to 192.168.2.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.1.1
!!!!!!!!!!
Success rate is 100 percent (10/10), round-trip min/avg/max = 61/93/206 ms
HQ#


Ping from Branch_B LAN 
-------------------------
Branch_B#ping 192.168.1.1 source 192.168.2.1 repeat 10
Type escape sequence to abort.
Sending 10, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.2.1
!!!!!!!!!!
Success rate is 100 percent (10/10), round-trip min/avg/max = 75/105/203 ms
Branch_B#

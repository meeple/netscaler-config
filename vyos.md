interface ethernet eth0 address 192.168.100.253/24
nat source...
protocols ospf area 0 network 192.168.100.0/24
protocols static route 0.0.0.0/0 next-hop 192.168.100.254


set interface ethernet eth1 address 192.168.68.253/24
set interface ethernet eth2 address 192.168.100.253/24

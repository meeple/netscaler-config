The goal of this configuration is to create a catch-all layer 2 vserver (mac redirection) which will load-balance all traffic with the exception of traffic destined to a list of subnets and ip addresses. 

To select bypassed traffic the `listen policy` feature will be used.

Due to the limitation of a maximum listen policy size ([Netscaler system limits](https://support.citrix.com/s/article/CTX118716-netscaler-appliance-system-limits) - Length of the policy rule	1,499), in case of mutiple exceptions they won't fit in a single line so a different approach is required. Netscaler has datasets and patsets feature which have much higher limit of number of entries (Number of Patset/Dataset bindings	5000), which can be used inside appexpert policies with equals_any operator, but unfortunately as for now, equals_any is not available inside listen policies.

The trick here is to use typecasting of the dataset.

Enable L2 mode:  
`enable ns mode L2 Edge`

Add vlan:  
`add vlan 2`  

Bind interfaces to vlan:
```
bind vlan 2 -ifnum 1/1
bind vlan 2 -ifnum 1/2
bind vlan 2 -ifnum 1/3
```

Create pattern set with a list of *destination* ip addresses and/or subnets to bypass:

```
add policy dataset ds_bypass ipv4
bind policy dataset ds_bypass "10.0.0.0/8"
bind policy dataset ds_bypass "192.168.0.0/16"
bind policy dataset ds_bypass 8.8.8.8
```

Separate pattern set with a list of *source* ip addresses and/or subnets to bypass:

```
add policy dataset ds_bypass_src ipv4
bind policy dataset ds_bypass_src "10.0.0.0/8"
bind policy dataset ds_bypass_src "192.168.0.0/
```

Vserver configuration is based on following CTX article:
[How to Use NetScaler to Load Balance Transparent Network Devices Such As Firewall](https://support.citrix.com/s/article/CTX218537-how-to-use-netscaler-to-load-balance-transparent-network-devices-such-as-firewall?language=en_US)


Add server:  
`add server proxy 192.168.0.10`


Create service (or service-groups as in the aforementioned article):  
`add service svc-proxy-any proxy ANY * -gslb NONE -maxClient 0 -maxReq 0 -cip DISABLED -usip YES -useproxyport NO -sp OFF -cltTimeout 120 -svrTimeout 120 -CKA NO -TCPB NO -CMP NO`

The goal is to redirect multiple ports. This can be done either with a single vserver with appripriate listen policy or with multiple vservers listening on a single port:
```
add lb vserver vs-any-80 ANY * 80 -persistenceType NONE -Listenpolicy "CLIENT.INTERFACE.ID.EQ(\"1/1\") && client.IP.DST.TYPECAST_TEXT_T.EQUALS_ANY(\"ds_bypass\").NOT && client.IP.SRC.TYPECAST_TEXT_T.EQUALS_ANY(\"ds_bypass_src\").NOT" -m MAC -cltTimeout 120
add lb vserver vs-any-443 ANY * 443 -persistenceType NONE -Listenpolicy "CLIENT.INTERFACE.ID.EQ(\"1/1\") && client.IP.DST.TYPECAST_TEXT_T.EQUALS_ANY(\"ds_bypass\").NOT && client.IP.SRC.TYPECAST_TEXT_T.EQUALS_ANY(\"ds_bypass_src\").NOT" -m MAC -cltTimeout 120
```

Bind services to vservers:
```
bind lb vserver vs-any-80 svc-proxy-any
bind lb vserver vs-any443 svc-proxy-any
```





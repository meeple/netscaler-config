Currently, it is not possible with bot module alone but it could be achieved on ADC via variables, rate-limit, responder, rewrite modules.

Please find below the sample configuration that blocks the client (based on IP) for 1hr (3600sec) if client from same ip-address access 3 requests in 120sec (=120 000ms) that results in 404 response status. All the policies below could be augmented with message action to generate the appropriate log-message. In addition, the below configuration could be used to derive/enhance for similar other use cases.


Add a stream selector for client (CLIENT.IP.SRC) and identifier to limit 3 request per 120000ms
Change the selector to device-fingerprint, sessionid, or any other client property accessible PI expression  
`add stream selector test_sel_ip_spray CLIENT.IP.SRC`
Change the threshold from 3 req per 120000ms to appropriate value  
`add ns limitIdentifier test_identifier_ip_spray -threshold 3 -timeSlice 120000 -selectorName test_sel_ip_spray`

Create a policy variable to hold the client and timestamp(in secs) at which the ratelimit exceeeds. *_rwpol_ip_spray sets the variable test_blockip_varmap.  
`add ns variable test_blockip_varmap -type "map(text(15),ulong,32000)" -expires 86400`
`add ns assignment test_set_blockip_varmap -variable "$test_blockip_varmap[client.ip.src.typecast_text_t]" -set 'sys.time'`
`add ns assignment test_unset_blockip_varmap -variable "$test_blockip_varmap[client.ip.src.typecast_text_t]" -set '0'`


Rewrite policy that sets the policy variable based on the conditions in response.  
Change the http.res.status.eq(404)  to any other condition (for example looking in response body for login failures instead of 404 responses).  
`add rewrite policy test_rwpol_ip_spray "http.res.status.eq(404) && SYS.CHECK_LIMIT(\"test_identifier_ip_spray\")" test_set_blockip_varmap`

Add a responder action for responding with configured response page.  
`add responder action test_respact_ip_spray respondwith q{"HTTP/1.1 403 Forbidden\r\nContent-Length:26\r\n\r\nPlease try after some time"}``

Add a responder policy to check the existence of ip in the variable map. If the value exists and (diff(current_time, time_at_which_value_is_set) < 60), policy evaluates to true and sends the responder page.  
Please note '3600' used here, blocks the access for 3600 secs, change appropriately.  
`add responder policy test_resppol_ip_spray "$test_blockip_varmap.valueexists(client.ip.src.typecast_text_t) && ($test_blockip_varmap[client.ip.src.typecast_text_t] != 0) && (sys.time.typecast_num_at.sub($test_blockip_varmap[client.ip.src.typecast_text_t]).le(3600))" test_respact_ip_spray`

Add responder policy test_resppol_unset_ip_spray to unset the variable  
`add responder policy test_resppol_unset_ip_spray "$test_blockip_varmap.valueexists(client.ip.src.typecast_text_t)" test_unset_blockip_varmap`

Please bind the rewrite policy  
`bind lb vserver lb1 -policyName test_rwpol_ip_spray -priority 100 -gotoPriorityExpression END -type RESPONSE`

Please bind the responder polices in the same order and at consecutive priorities.  
`bind lb vserver lb1 -policyName test_resppol_ip_spray -priority 100 -gotoPriorityExpression END -type REQUEST`
`bind lb vserver lb1 -policyName test_resppol_unset_ip_spray -priority 101 -gotoPriorityExpression END -type REQUEST`

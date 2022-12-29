`add stream selector selector_url_a "http.REQ.HEADER(\"User-Agent\")" "http.REQ.URL.CONTAINS(\"url_a\")"`  
`add stream selector selector_url_b "http.REQ.HEADER(\"User-Agent\")" "http.REQ.URL.CONTAINS(\"url_b\")"`  
`add stream selector selector_url_c "http.REQ.HEADER(\"User-Agent\")" "http.REQ.URL.CONTAINS(\"url_c\")"`  
`add stream selector selector_url_d "http.REQ.HEADER(\"User-Agent\")" "http.REQ.URL.CONTAINS(\"url_d\")"`  

`add ns limitIdentifier limitid_a -timeSlice 30000 -mode REQUEST_RATE -limitType SMOOTH -selectorName selector_url_a`  
`add ns limitIdentifier limitid_b -timeSlice 30000 -mode REQUEST_RATE -limitType SMOOTH -selectorName selector_url_b`  
`add ns limitIdentifier limitid_c -timeSlice 30000 -mode REQUEST_RATE -limitType SMOOTH -selectorName selector_url_c`  
`add ns limitIdentifier limitid_d -timeSlice 30000 -mode REQUEST_RATE -limitType SMOOTH -selectorName selector_url_d`  

`add responder policy resp_pol_a http.REQ.URL.CONTAINS(\"url_a\")" && "SYS.CHECK_LIMIT(\"limitid_a\") && resp_act_respondwith429 -logAction ratelimit_log_action`  
`add responder policy resp_pol_b http.REQ.URL.CONTAINS(\"url_b\")" && "SYS.CHECK_LIMIT(\"limitid_b\") &&resp_act_respondwith429 -logAction ratelimit_log_action`  
`add responder policy resp_pol_c http.REQ.URL.CONTAINS(\"url_c\")" && "SYS.CHECK_LIMIT(\"limitid_c\") && resp_act_respondwith429 -logAction ratelimit_log_action`  
`add responder policy resp_pol_d http.REQ.URL.CONTAINS(\"url_d\")" && "SYS.CHECK_LIMIT(\"limitid_d\") && resp_act_respondwith429 -logAction ratelimit_log_action`  

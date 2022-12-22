`add stream selector Jeca1 "http.REQ.HEADER(\"User-Agent\")" "http.REQ.URL.CONTAINS(\"prolece\")"`  
`add stream selector Jeca2 "http.REQ.HEADER(\"User-Agent\")" "http.REQ.URL.CONTAINS(\"leto\")"`  
`add stream selector Jeca3 "http.REQ.HEADER(\"User-Agent\")" "http.REQ.URL.CONTAINS(\"jesen\")"`  
`add stream selector Jeca4 "http.REQ.HEADER(\"User-Agent\")" "http.REQ.URL.CONTAINS(\"zima\")"`  

`add ns limitIdentifier Jeca_LI1 -timeSlice 30000 -mode REQUEST_RATE -limitType SMOOTH -selectorName Jeca1`  
`add ns limitIdentifier Jeca_LI2 -timeSlice 30000 -mode REQUEST_RATE -limitType SMOOTH -selectorName Jeca2`  
`add ns limitIdentifier Jeca_LI3 -timeSlice 30000 -mode REQUEST_RATE -limitType SMOOTH -selectorName Jeca3`  
`add ns limitIdentifier Jeca_LI4 -timeSlice 30000 -mode REQUEST_RATE -limitType SMOOTH -selectorName Jeca4`  

`add responder policy Jeca_resp_pol1 "SYS.CHECK_LIMIT(\"Jeca_LI1\") && http.REQ.URL.CONTAINS(\"prolece\")" resp_act_respondwith429 -logAction ratelimit_log_action`  
`add responder policy Jeca_resp_pol2 "SYS.CHECK_LIMIT(\"Jeca_LI2\") && http.REQ.URL.CONTAINS(\"leto\")" resp_act_respondwith429 -logAction ratelimit_log_action`  
`add responder policy Jeca_resp_pol3 "SYS.CHECK_LIMIT(\"Jeca_LI3\") && http.REQ.URL.CONTAINS(\"jesen\")" resp_act_respondwith429 -logAction ratelimit_log_action`  
`add responder policy Jeca_resp_pol4 "SYS.CHECK_LIMIT(\"Jeca_LI4\") && http.REQ.URL.CONTAINS(\"zima\")" resp_act_respondwith429 -logAction ratelimit_log_action`  

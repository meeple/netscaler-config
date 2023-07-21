create a map variable which stores the apikey and counts number of requests:   
`add ns variable session_map -type map(text(6),ulong,100) -init 0 -expires 1`

text(6) - key is 6 characters long  
100 - max number of entries in session_map  

create an assignment action to increase counter:  
`add ns assignment increment_session -variable '$session_map[http.req.header("Apikey").value(0)]' -add 1`

create a responder policy to trigger increment_session:  
`add responder policy increment_session_policy 'http.req.url.contains("headers")' increment_session`

responder action to send error page:  
`add responder action error_too_busy respondwith '"HTTP/1.1 429 Too Many Requests\r\n\r\n"'`

responder policy to trigger error page:    
`add responder policy check_busy_policy "http.req.url.contains(\"headers\") && $session_map[http.req.header(\"Apikey\").value(0)]>3" error_too_busy`


add logging policy to check current values:  
`add audit messageaction log_sessionmap EMERGENCY "\"log sessionmap: apikey header: \"+http.REQ.HEADER(\"Apikey\").VALUE(0)+\", session map:\"+$session_map[http.req.header(\"Apikey\").value(0)]"`  
`add responder policy log_sessionmap_policy true NOOP -logAction log_sessionmap`

bind policies:  
`bind responder global log_sessionmap_policy 5 NEXT`  
`bind responder global increment_session_policy 10 NEXT`  
`bind responder global check_busy_policy 20`

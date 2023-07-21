#Jesli certyfikat klient jest nieważny – RESET:

`add responder policy resp_cert_valid CLIENT.SSL.CLIENT_CERT.IS_VALID.NOT RESET`

#logowanie emaila z certu:
`add audit messageaction log_email_from_cert CRITICAL CLIENT.SSL.CLIENT_CERT.SUBJECT`
`add responder policy resp_pol_login "CLIENT.SSL.CLIENT_CERT.SUBJECT.CONTAINS(\"emailAddress\")" NOOP -logAction log_email_from_cert`



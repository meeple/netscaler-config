http callout example


Let’s assume that the client preforms a request to the email endpoint http://itsaappdev01.centraltest.roottest.alphatest.ab:3001/api/v1/msg/emailSend which expects as input/body the client {id} - CitrixAPIGW should invoke http:///itsaappdev01.centraltest.roottest.alphatest.ab:3000/api/v1/validation/accounts?id={} beforehand and only if the second webservice returns an email address for the respective client {id} CitrixAPIGW should proceed with the initial request – else an error message (e.g. Unknown email address for {id}) should return to the client.

opcja 1
get http://hello.kuba.lab/email?id=kuba
callout http://httpbin.kuba.lab?id=kuba
if email returned OK
if no email respond with "Uknown email address"


opcja 2
post http://hello.kuba.lab/email body { id: "kuba" }
callout http://httpbin.kuba.lab?id=kuba
if email returned OK
if no email respond with "Uknown email address"

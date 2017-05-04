# Errors


The Beeminder API uses the following error codes.
Check your HTTP response code, but also we will usually pass back a JSON object with a key `errors` that will hopefully illuminate what went wrong.


Error Code | Meaning
---------- | -------
400 | Bad Request
401 | Unauthorized -- Wrong / missing key. (Check your parameter name?)
404 | Not Found -- Couldn't find that resource, or path requested. 
406 | Not Acceptable -- Check the format of your params? 
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.

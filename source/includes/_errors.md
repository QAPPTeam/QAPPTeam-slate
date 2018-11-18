# Errors

<aside class="notice">
useful are not fully implemented, most errors do not currently return the correct error code
</aside>

API uses the following error codes:


Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is invalid.
401 | Unauthorized -- Your API key is wrong.
404 | Not Found -- The specified user or study could not be found.
405 | Method Not Allowed
406 | Not Acceptable
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.

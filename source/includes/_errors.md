# Errors

The Dispatch API uses the following error codes:


Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is malformed or otherwise invalid
401 | Unauthorized -- Your OAuth2 bearer token is incorrect.
403 | Forbidden -- Your credentials are correct, but you're not allowed to perform this action.
404 | Not Found -- Requested resource could not be found
422 | Unprocessable Entity -- Your request payload did not pass our validation rules. See response body for details.
429 | Too Many Requests -- You've exceeded your rate limit.
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintanance. Please try again later.

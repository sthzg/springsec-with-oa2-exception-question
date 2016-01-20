## Example Requests

Accessing `/protected` with Basic Auth works as assumed:

    # ✓ Basic Auth request to protected resource (= 200)
    curl -X "GET" "http://localhost:8080/protected" \
    	-H "Authorization: Basic dXNlcjpwYXNzd29yZA=="
    	
    	
Accessing it with Oauth2 token works as assumed:

    # Get a token
    curl -X "POST" "http://localhost:8080/oauth/token" \
    	-H "Authorization: Basic b2EyLWNsaWVudDpTRUNSRVQ=" \
    	-H "Content-Type: application/x-www-form-urlencoded" \
    	--data-urlencode "grant_type=password" \
    	--data-urlencode "username=user" \
    	--data-urlencode "password=password"
    	
    # ✓ Access protected resource (= 200)
    curl -X "GET" "http://localhost:8080/protected" \
    	-H "Authorization: Bearer <paste bearer here>"
    	
    	
Now coming to the behavior that I am wondering about, first a sanity check:

    # ✓ Unauthenticated GET with explicit Accept header text/html (= 302)
    curl -X "GET" "http://localhost:8080/protected" \
    	-H "Accept: text/html"


That the previous request returned 302 makes me assume that the basic form login mechanism is generally in place. 
One more sanity check:

    # ✓ Unauthenticated GET with explicit Accept header application/xml (= 401)
    curl -X "GET" "http://localhost:8080/protected" \
    	-H "Accept: text/xml"
    	
    # .. results in 
    <oauth>
      <error_description>Full authentication is required to access this resource</error_description>
      <error>unauthorized</error>
    </oauth>
    
    
## The Question    
    
Everything so far is what I would have expected. Now the one test case that my question originates from is a 
default GET request like it is issued from a browser (tested with Chrome and Firefox). These send the follwoing 
Accept header:

    text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8  # Firefox (latest, OS X)
    text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8  # Chrome (latest, OS X)
    
> Opening up `http://localhost:8080/protected` in a browser always returns the 401 with an XML representation and 
> my question is if that is the expected behavior.

I would have assumed, that since the `/protected` resource is both protected by HTTP Basic and Oauth2, 
it would respond with a `302` to a request that has `text/html` listed as first Accept header.

Otherwise since the first three content types are marked with a 0.9 quality factor it also makes sense that if 
something handles the exception that knows how to produce xml (but not html) it would simply return that instead. 

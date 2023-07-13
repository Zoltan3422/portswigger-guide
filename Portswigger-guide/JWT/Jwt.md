<!-- omit in toc -->
# Table of Contents

- [[#JWT authentication bypass via unverified signature]]
- [[#JWT authentication bypass via flawed signature verification]]
- [[#JWT authentication bypass via weak signing key]]
- [[#JWT authentication bypass via jwk header injection]]
- [[#JWT authentication bypass via jku header injection]]
- [[#JWT authentication bypass via kid header path traversal]]

# JWT authentication bypass via unverified signature
Reference: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature

```ad-done
title: Solution
- n the lab, log in to your own account.
    
- In Burp, go to the **Proxy > HTTP history** tab and look at the post-login `GET /my-account` request. Observe that your session cookie is a JWT.
    
- Double-click the payload part of the token to view its decoded JSON form in the Inspector panel. Notice that the `sub` claim contains your username. Send this request to Burp Repeater.
    
- In Burp Repeater, change the path to `/admin` and send the request. Observe that the admin panel is only accessible when logged in as the `administrator` user.
    
- Select the payload of the JWT again. In the Inspector panel, change the value of the `sub` claim from `wiener` to `administrator`, then click **Apply changes**.
    
- Send the request again. Observe that you have successfully accessed the admin panel.
    
- In the response, find the URL for deleting `carlos` (`/admin/delete?username=carlos`). Send the request to this endpoint to solve the lab.
```

# JWT authentication bypass via flawed signature verification
Reference: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification
```ad-done
title: Solution
1. In the lab, log in to your own account.
    
2. In Burp, go to the **Proxy > HTTP history** tab and look at the post-login `GET /my-account` request. Observe that your session cookie is a JWT.
    
3. Double-click the payload part of the token to view its decoded JSON form in the **Inspector** panel. Notice that the `sub` claim contains your username. Send this request to Burp Repeater.
    
4. In Burp Repeater, change the path to `/admin` and send the request. Observe that the admin panel is only accessible when logged in as the `administrator` user.
    
5. Select the payload of the JWT again. In the **Inspector** panel, change the value of the `sub` claim to `administrator`, then click **Apply changes**.
    
6. Select the header of the JWT, then use the Inspector to change the value of the `alg` parameter to `none`. Click **Apply changes**.
    
7. In the message editor, remove the signature from the JWT, but remember to leave the trailing dot after the payload.
    
8. Send the request and observe that you have successfully accessed the admin panel.
    
9. In the response, find the URL for deleting `carlos` (`/admin/delete?username=carlos`). Send the request to this endpoint to solve the lab.
```

# JWT authentication bypass via weak signing key
Reference: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key

```ad-done
title: Solution
## Part 1 - Brute-force the secret key

1. In Burp, load the JWT Editor extension from the BApp store.
    
2. In the lab, log in to your own account and send the post-login `GET /my-account` request to Burp Repeater.
    
3. In Burp Repeater, change the path to `/admin` and send the request. Observe that the admin panel is only accessible when logged in as the `administrator` user.
    
4. Copy the JWT and brute-force the secret. You can do this using hashcat as follows:
    
    ~~~
    hashcat -a 0 -m 16500 <YOUR-JWT> /path/to/jwt.secrets.list
    ~~~
    
    If you're using hashcat, this outputs the JWT, followed by the secret. If everything worked correctly, this should reveal that the weak secret is `secret1`.
    

### Note

Note that if you run the command more than once, you need to include the `--show` flag to output the results to the console again.

## Part 2 - Generate a forged signing key

1. Using Burp Decoder, Base64 encode the secret that you brute-forced in the previous section.
    
2. In Burp, go to the **JWT Editor Keys** tab and click **New Symmetric Key**. In the dialog, click **Generate** to generate a new key in JWK format. Note that you don't need to select a key size as this will automatically be updated later.
    
3. Replace the generated value for the `k` property with the Base64-encoded secret.
    
4. Click **OK** to save the key.
    

## Part 3 - Modify and sign the JWT

1. Go back to the `GET /admin` request in Burp Repeater and switch to the extension-generated **JSON Web Token** message editor tab.
    
2. In the payload, change the value of the `sub` claim to `administrator`
    
3. At the bottom of the tab, click `Sign`, then select the key that you generated in the previous section.
    
4. Make sure that the `Don't modify header` option is selected, then click `OK`. The modified token is now signed with the correct signature.
    
5. Send the request and observe that you have successfully accessed the admin panel.
    
6. In the response, find the URL for deleting `carlos` (`/admin/delete?username=carlos`). Send the request to this endpoint to solve the lab.
```

# JWT authentication bypass via jwk header injection
Reference: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection

```ad-done
title: Solution
1. In Burp, load the JWT Editor extension from the BApp store.
    
2. In the lab, log in to your own account and send the post-login `GET /my-account` request to Burp Repeater.
    
3. In Burp Repeater, change the path to `/admin` and send the request. Observe that the admin panel is only accessible when logged in as the `administrator` user.
    
4. Go to the **JWT Editor Keys** tab in Burp's main tab bar.
    
5. Click **New RSA Key**.
    
6. In the dialog, click **Generate** to automatically generate a new key pair, then click **OK** to save the key. Note that you don't need to select a key size as this will automatically be updated later.
    
7. Go back to the `GET /admin` request in Burp Repeater and switch to the extension-generated `JSON Web Token` tab.
    
8. In the payload, change the value of the `sub` claim to `administrator`.
    
9. At the bottom of the **JSON Web Token** tab, click **Attack**, then select **Embedded JWK**. When prompted, select your newly generated RSA key and click **OK**.
    
10. In the header of the JWT, observe that a `jwk` parameter has been added containing your public key.
    
11. Send the request. Observe that you have successfully accessed the admin panel.
    
12. In the response, find the URL for deleting `carlos` (`/admin/delete?username=carlos`). Send the request to this endpoint to solve the lab.
```

# JWT authentication bypass via jku header injection
Reference: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jku-header-injection
```ad-done
title: Solution
##### Part 1 - Upload a malicious JWK Set

1. In Burp, load the JWT Editor extension from the BApp store.
    
2. In the lab, log in to your own account and send the post-login `GET /my-account` request to Burp Repeater.
    
3. In Burp Repeater, change the path to `/admin` and send the request. Observe that the admin panel is only accessible when logged in as the `administrator` user.
    
4. Go to the **JWT Editor Keys** tab in Burp's main tab bar.
    
5. Click **New RSA Key**.
    
6. In the dialog, click **Generate** to automatically generate a new key pair, then click **OK** to save the key. Note that you don't need to select a key size as this will automatically be updated later.
    
7. In the browser, go to the exploit server.
    
8. Replace the contents of the **Body** section with an empty JWK Set as follows:
    
    ~~~
    { "keys": [ ] }
    ~~~
1. Back on the **JWT Editor Keys** tab, right-click on the entry for the key that you just generated, then select **Copy Public Key as JWK**.
    
10. Paste the JWK into the `keys` array on the exploit server, then store the exploit. The result should look something like this:
    
    ~~~
    { "keys": [ { "kty": "RSA", "e": "AQAB", "kid": "893d8f0b-061f-42c2-a4aa-5056e12b8ae7", "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw" } ] }
    ~~~

##### Part 2 - Modify and sign the JWT

1. Go back to the `GET /admin` request in Burp Repeater and switch to the extension-generated **JSON Web Token** message editor tab.
    
2. In the header of the JWT, replace the current value of the `kid` parameter with the `kid` of the JWK that you uploaded to the exploit server.
    
3. Add a new `jku` parameter to the header of the JWT. Set its value to the URL of your JWK Set on the exploit server.
    
4. In the payload, change the value of the `sub` claim to `administrator`.
    
5. At the bottom of the tab, click **Sign**, then select the RSA key that you generated in the previous section.
    
6. Make sure that the **Don't modify header** option is selected, then click **OK**. The modified token is now signed with the correct signature.
    
7. Send the request. Observe that you have successfully accessed the admin panel.
    
8. In the response, find the URL for deleting `carlos` (`/admin/delete?username=carlos`). Send the request to this endpoint to solve the lab.
```

# JWT authentication bypass via kid header path traversal
Reference: https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-kid-header-path-traversal

```ad-done
title: Solution
### Generate a suitable signing key

1. In Burp, load the JWT Editor extension from the BApp store.
    
2. In the lab, log in to your own account and send the post-login `GET /my-account` request to Burp Repeater.
    
3. In Burp Repeater, change the path to `/admin` and send the request. Observe that the admin panel is only accessible when logged in as the `administrator` user.
    
4. Go to the **JWT Editor Keys** tab in Burp's main tab bar.
    
5. Click **New Symmetric Key**.
    
6. In the dialog, click **Generate** to generate a new key in JWK format. Note that you don't need to select a key size as this will automatically be updated later.
    
7. Replace the generated value for the `k` property with a Base64-encoded null byte (`AA==`). Note that this is just a workaround because the JWT Editor extension won't allow you to sign tokens using an empty string.
    
8. Click **OK** to save the key.
    

### Modify and sign the JWT

1. Go back to the `GET /admin` request in Burp Repeater and switch to the extension-generated **JSON Web Token** message editor tab.
    
2. In the header of the JWT, change the value of the `kid` parameter to a path traversal sequence pointing to the `/dev/null` file:
    
    ~~~
    ../../../../../../../dev/null
    ~~~

1. In the JWT payload, change the value of the `sub` claim to `administrator`.
    
4. At the bottom of the tab, click **Sign**, then select the symmetric key that you generated in the previous section.
    
5. Make sure that the **Don't modify header** option is selected, then click **OK**. The modified token is now signed using a null byte as the secret key.
    
6. Send the request and observe that you have successfully accessed the admin panel.
    
7. In the response, find the URL for deleting `carlos` (`/admin/delete?username=carlos`). Send the request to this endpoint to solve the lab.
```
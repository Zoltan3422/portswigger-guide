<!-- omit in toc -->
## Table of Contents

- [[#Basic clickjacking with CSRF token protection]]
- [[#Clickjacking with form input data prefilled from a URL parameter]]
- [[#Exploiting clickjacking vulnerability to trigger DOM-based XSS]]
- [[#Multistep clickjacking]]

# Basic clickjacking with CSRF token protection
Reference: https://portswigger.net/web-security/clickjacking/lab-basic-csrf-protected

```ad-hint
title: Quick Solution
Just craft a webpage where there is a 'Click Me' button on top of the delete account. 
![[Csrf token protection]]
```

# Clickjacking with form input data prefilled from a URL parameter
Reference: https://portswigger.net/web-security/clickjacking/lab-prefilled-form-input

```ad-hint
title: Quick Solution
Similar to the previous lab, the only difference is that I had to prefill the input form data. 
![[Input data prefilled]]
```

# Clickjacking with a frame buster script
Reference: https://portswigger.net/web-security/clickjacking/lab-frame-buster-script

```ad-hint
title: Quick Solution
The exploit is **exactly** the same of the previous lab. The only difference is that the target website uses **frame busting**. This technique can easily be circumvented with the ``sandbox="allow-forms"`` attribute. 
![[Frame buster script]]
```

# Exploiting clickjacking vulnerability to trigger DOM-based XSS
Reference: https://portswigger.net/web-security/clickjacking/lab-exploiting-to-trigger-dom-based-xss

```ad-hint
title: Quick Solution
In this case there is a *Submit a feedback* page that contains a DOM-based XSS vulnerability. So the only difference in this lab is that we have to prefill the input with an XSS payload. 
![[Dom based xss]]
```

# Multistep clickjacking
Reference: https://portswigger.net/web-security/clickjacking/lab-multistep

```ad-hint
title: Quick Solution
This last lab is pretty easy, I just had to add a second button instead of just one. Look at the ``multistep.html`` file in the ``exploits`` directory.
![[Multistep]]
```

# Cross Site Scripting (XSS)

Cross-Site Scripting (XSS) attacks are a type of injection, in which malicious scripts are injected into otherwise benign and trusted websites. XSS attacks occur when an attacker uses a web application to send malicious code, generally in the form of a browser side script, to a different end user.

View on [owasp](https://owasp.org/www-community/attacks/xss/#:~:text=Overview,to%20a%20different%20end%20user).

## Install

```
npm i xss
```

## Include

```js
import xss from 'xss';
```

## Usage

```js
app.use(xss());
```

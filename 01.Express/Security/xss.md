# Cross Site Scripting (XSS)

Cross-Site Scripting (XSS) attacks are a type of injection, in which malicious scripts are injected into otherwise benign and trusted websites. XSS attacks occur when an attacker uses a web application to send malicious code, generally in the form of a browser side script, to a different end user.

View on [owasp](https://owasp.org/www-community/attacks/xss/#:~:text=Overview,to%20a%20different%20end%20user).

View [npm package docs](https://www.npmjs.com/package/xss)

## Install

```
npm i xss-clean
```

## Include

```js
import xss from 'xss-clean';
```

## Usage

```js
app.use(xss());
```

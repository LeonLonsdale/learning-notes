# Packages and Libraries

## 1) Base Packages

```
npm i nodemailer
```

# Definitions

# Send Emails with nodemailer.

- [Documentation](https://nodemailer.com)

## 1) Install with npm

```
npm i nodemailer
```

## 2) Setup email environment variables

Add your email host, port, username and password to `env` file.

```
EMAIL_HOST=
EMAIL_PORT=
EMAIL_USER=
EMAIL_PASS=
```

## 3) Setup sendEmail function

The transporter object method `sendMail` returns a promise. So use async function.

```js
export const sendEmail = async (options) => {};
```

Create configuration object.

```js
export const sendEmail = async (options) => {
  const configuration = {
    host: process.env.EMAIL_HOST,
    port: process.env.EMAIL_PORT,
    auth: {
      user: process.env.EMAIL_USER,
      pass: process.env.EMAIL_PASS,
    },
  };
};
```

A transporter is required to send emails. It only needs to be created once. Create the transporter.

```javascript
export const sendEmail = async (options) => {
  const configuration = {
    host: process.env.EMAIL_HOST,
    port: process.env.EMAIL_PORT,
    auth: {
      user: process.env.EMAIL_USER,
      pass: process.env.EMAIL_PASS,
    },
  };

  const transporter = nodemailer.createTransport(options);
};
```

Setup mail options.

```javascript
export const sendEmail = async (options) => {
  const configuration = {
    host: process.env.EMAIL_HOST,
    port: process.env.EMAIL_PORT,
    auth: {
      user: process.env.EMAIL_USER,
      pass: process.env.EMAIL_PASS,
    },
  };

  const transporter = nodemailer.createTransport(options);

  const mailOptions = {
    from: '<from email address>',
    to: options.email,
    subject: options.subject,
    text: options.text, // raw text format
    // html: options.html (used to send HTML instead of raw text)
  };
};
```

# Emails

For this doc, we will use [Mailtrap](https://www.mailtrap.io) to capture emails in developer mode and [SendGrid](https://www.sendgrid.com) for emails in production. Need to create an account on each.

# Basic Emails with Nodemailer

## Nodemailer Documentation.

- [Documentation](https://nodemailer.com)

## Installation

```
npm i nodemailer
```

## Setup email environment variables

Add your email host, port, username and password to `env` file.

```
EMAIL_HOST=
EMAIL_PORT=
EMAIL_USER=
EMAIL_PASS=
EMAIL_FROM=
```

## Setup sendEmail function

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

Use the transporter to send the email:

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

  await transporter.sendMail(mailOptions);
};
```

# Sending More Complex Emails using Templates

## New Environment Variables

```
SENDGRID_USER=
SENDGRID_PASS=
```

## Create a class

We will use this class to create Email objects that we can use to send emails. Our class will accept a copy of the user, and a URL. The URL will by any URL we want to include in the email; such as a password reset link.

```js
export default class Email {
  constructor(user, url) {
    this.to = user.email;
    this.firstName = user.name.split(' ')[0];
    this.url = url;
    this.from = `name ${process.env.EMAIL_FROM}`;
  }
}
```

## Create the transport creation method

```js
newTransport() {
  if (process.env.NODE_ENV === 'production') {
    return nodemailer.createTransport({
      service: 'SendGrid',
      auth: {
        user: process.env.SENDGRID_USER,
        pass: process.env.SENDGRID_PASS,
      }
    });
  }

  // for development, our transporter is set up
  // in the same way as our basic nodemailer

  return nodemailer.createTransport({
    host: process.env.EMAIL_HOST,
    port: process.env.EMAIL_PORT,
    auth: {
      user: process.env.EMAIL_USER,
      pass: process.env.EMAIL_PASS,
    }
  })
}
```

## Create the send email method

For this method, we need to utilise features of our `pug` package, and install a new package to allow us to also send a text-only version of the email.

```
npm i html-to-text
```

```js
import pug from 'pug';
import * as htmlToText from 'html-to-text';
```

We will use the `renderFile` method from `pug` to render a html file from our pug template, without sending it to the browser. The syntax is:

```js
pug.renderFile(locationOfTemplate, {dataTemplateWillUse});
```

From the `html-to-text` package, we will use the `convert()` method to convert the rendered html file to plaintext.

```js
htmlToText.fromString(htmlFile);
```

```js
async send(template, subject) {
  // render the html template
  const html = pug.renderFile(
    `${__dirname}/../views/emails/${template}.pug`,
    {
      firstName: this.firstName,
      url: this.url,
      subject,
    }
  );

  // define the email options
  const mailOptions = {
    from: this.from,
    to: this.to,
    subject,
    text: htmlToText.convert(html),
    html,
  };

  // create the transport
  await this.newTransport().sendMail(mailOptions);
}
```

## Create the send welcome method

Since this method will use the `send()` method, which is `async`, this method must also be `async`.

```js
async sendWelcome() {
  await this.send('templateName', 'emailSubject');
}
```

This method can be duplicated and customised for each type of email we may need to send, such as:

- Password reset requests
- Confirm password changed
- Confirm payments / invoices
- Any other notification

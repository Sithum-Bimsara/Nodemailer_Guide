# 📧 Nodemailer Complete Guide

Welcome to this step-by-step guide on setting up and using **Nodemailer** with Express.js! 🚀 This guide will help you configure a mailing service using Gmail, environment variables, and Postman for testing. Let's get started! 🎉

---

## 🎯 Step 1: Initialize the Project

1. **Create a new directory for the project** 🗂️
   ```sh
   md nodemailer
   ```
2. **Move into the directory** 📂
   ```sh
   cd nodemailer
   ```
3. **Initialize a Node.js project** 📜
   ```sh
   npm init -y
   ```
4. **Install required dependencies** 📦
   ```sh
   npm i express nodemailer dotenv
   ```
5. **Open the project in VS Code** 🖥️
   ```sh
   code .
   ```

---

## ✍️ Step 2: Set Up Gmail Authentication

To send emails using **Nodemailer**, we need to configure Gmail authentication. Follow these steps carefully:

1. **Go to your Google Account** ➡️ Manage Account ➡️ Security ⚙️
2. **Enable 2-Step Verification** 🔐 (Mandatory for App Passwords)
3. **Create an App Password** 🔑:
   - Scroll down to **App Passwords** (Search if necessary 🔎)
   - Click **Generate App Password**
   - Enter the **app name** (e.g., "Nodemailer")
   - Copy the **generated password** 📋

📌 **What is an App Password?**
An **App Password** is a unique code that allows less secure apps (like our Node.js server) to access your Gmail without using your main password. It ensures security while allowing email automation. 🛡️

---

## 🔧 Step 3: Configure Environment Variables

Create a `.env` file in your project root and store the email credentials securely:

```env
EMAIL=your-email@gmail.com
TO_EMAIL=recipient-email@gmail.com
PASSWORD=your-app-password
PORT=8080
```

🚨 **Never share your `.env` file or upload it to GitHub!**

---

## 🏗️ Step 4: Create `index.js` File

Inside your project, create a new file `index.js` and add the following code:

```js
require('dotenv').config();
const nodemailer = require('nodemailer');
const express = require('express');
const app = express();

const transporter = nodemailer.createTransport({
    service: 'gmail',
    host: 'smtp.gmail.com',
    secure: false,
    port: 587,
    auth: {
        user: process.env.EMAIL,
        pass: process.env.PASSWORD
    }
});

app.get('/', (req, res) => {
    res.send('Hello World');
    const mailOptions = {
        from: process.env.EMAIL,
        to: process.env.TO_EMAIL,
        subject: 'Sending Email using Node.js',
        text: 'That was easy!'
    };
    
    transporter.sendMail(mailOptions, (error, info) => {
        if (error) {
            console.log(error);
        } else {
            console.log('Email sent:', info.response);
        }
    });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`App listening on port ${PORT}!`));
```

Run the app using:
```sh
nodemon index.js
```
📌 Open `http://localhost:8080` in your browser. If configured correctly, you should see an email sent confirmation in your terminal! ✅

---

## 📬 Step 5: Create a `Mail` Class

Now, let's modularize the email-sending functionality by creating `mail.js`:

```js
const nodemailer = require('nodemailer');

const transporter = nodemailer.createTransport({
    service: 'gmail',
    host: 'smtp.gmail.com',
    secure: false,
    port: 587,
    auth: {
        user: process.env.EMAIL,
        pass: process.env.PASSWORD
    }
});

class Mail {
    constructor() {
        this.mailOptions = {
            from: {
                address: process.env.EMAIL,
                name: 'Your Name'
            }
        };
    }

    setTo(receiver) { this.mailOptions.to = receiver; }
    setSubject(subject) { this.mailOptions.subject = subject; }
    setText(text) { this.mailOptions.text = text; }
    setHTML(html) { this.mailOptions.html = html; }

    send() {
        transporter.sendMail(this.mailOptions, (error, info) => {
            if (error) console.log(error);
            else console.log('Email sent:', info.response);
        });
    }
}

module.exports = Mail;
```

---

## 🛠️ Step 6: Update `index.js`

Modify `index.js` to use our new `Mail` class:

```js
require('dotenv').config();
const express = require('express');
const Mail = require('./mail');
const app = express();

app.use(express.json());

app.get('/', (req, res) => {
    res.send('Hello World');
    const mail = new Mail();
    mail.setTo(process.env.TO_EMAIL);
    mail.setSubject('Subject');
    mail.setText('Hello from Nodemailer');
    mail.send();
});

app.post('/', async (req, res) => {
    res.send(req.body);
    const { receiver_id, subject, text, html } = req.body;
    const mail = new Mail();
    mail.setTo(receiver_id);
    mail.setSubject(subject);
    mail.setText(text);
    mail.setHTML(html);
    mail.send();
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`App listening on port ${PORT}!`));
```

---

## 🛠️ Step 7: Testing with Postman

1. Open **Postman** and create a **POST request**:
   ```
   URL: http://localhost:8080
   ```
2. Go to **Body** ➡️ **raw** ➡️ **JSON format**
3. Enter the following JSON:
   ```json
   {
       "receiver_id": "recipient-email@gmail.com",
       "subject": "This is a test subject",
       "text": "This is my text message",
       "html": "<h1>Please subscribe my channel!</h1>"
   }
   ```
4. Click **Send** and check the terminal for confirmation. ✅

---

## 📨 Step 8: Dynamic HTML Email

Create a new file `mail.html` for HTML-based emails:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome Email</title>
</head>
<body>
    <h1>Welcome to Our Website!</h1>
    <p>We are thrilled to have you with us.</p>
</body>
</html>
```

Modify `index.js` to read and send this HTML file dynamically! 🎨

```js
const fs = require('fs');
const path = require('path');
const htmlTemplate = fs.readFileSync(path.join(__dirname, 'mail.html'), 'utf8');
mail.setHTML(htmlTemplate);
```

---


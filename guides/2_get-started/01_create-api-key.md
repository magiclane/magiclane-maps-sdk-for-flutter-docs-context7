---
description: Documentation for Create Api Key
title: Create Api Key
---

# Create an API key

In this guide you will learn how to create your free Magic Lane account and your first project, and how to get your API Key.

Ensure that the API key remains secure and is not exposed. If the key is compromised or its integrity is uncertain, promptly revoke it and generate a new one to maintain security and prevent unauthorized usage that may incur additional costs.

### Create your account

Go to the Magic Lane [registration/sign up page](https://developer.magiclane.com/api/login) to create a free account.

You only need to type in your email address and choose a new password. Optionally, you can also enter your name and company.\

Click the `Get started` button after reviewing the terms of service and privacy policy. You will get a confirmation email with a link to activate your account. After you activate your account by clicking on the link you received, you can sign in!

Go to the Magic Lane [login page](https://developer.magiclane.com/api/login) to sign in to your new account.
Type your complete email address, which you used to create the account, and the password you created when you signed up in the previous step.

## Create a project

Once logged in, you are on your [Dashboard](https://developer.magiclane.com/api/dashboard) page. Click the [Projects](https://developer.magiclane.com/api/projects) tab.

On the My Projects page, click the `Create project` button from the top-right side of the page.

A dialog appears, where you can type a name for the new project, and a short description.
When done, click `Create project`.

## Create an API Key

The project is created with an API key with a short expiration date.

Click `Generate additional API Key` to create another key for your project, where you can set an expiration date further into the future.

You can give the new API key a different name, and set any expiration date you want.
Click `Generate` to generate the API key.

Ensure that key management is properly planned and that the key on the client’s device is updated before its expiration date. Failure to do so will result in a watermark being displayed on the map and certain features becoming limited once the key has expired.

You just created your first project and Magic Lane API Key!

You can click on the copy button to the right of the API Key to copy it, and then paste it in your code.

To enhance security, avoid hardcoding the API token in Dart code. Instead, consider one of the following approaches:

- Use a secure backend server to deliver the API token at runtime. This method not only protects the token from reverse engineering but also allows for easier updates or rotation without requiring a new app release.

- Store the API token in platform-specific configuration files (e.g., AndroidManifest.xml, AppDelegate.swift). While not fully secure, this approach offers a slightly better level of protection by leveraging native code and platform restrictions.

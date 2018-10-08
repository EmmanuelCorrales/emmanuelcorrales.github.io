---
layout: post
title:  "AWS: Best practices for new AWS accounts."
date:   2018-02-24 08:30:00 +0800
categories: aws iam
tags: [ aws, IAM]
---

These are the best practices for securing your new AWS account.

Best practices for managing the Root Account:
- Delete your root access keys.
- Activate MFA on your root account.
- Create individual IAM user.
- User groups to assign permissions.
- Apply an IAM password policy.

## Secure the root account:

### Delete your root access keys.
Delete the access keys to disable programatic requests to AWS account because
there is always a possibility that other people can get your root access keys.
Root access keys have no restrictions.

1. Login to the [AWS Management Console](https://console.aws.amazon.com/console/home),
then open **Services** then select **IAM**.
2. Open the dropdown **Security Status**, then click
**Manage Secutiry Credentials**.
3. Click **Continue to Security Credentials**.
4. Expand **Access keys (access key ID and secret access key)**.
5. Click the **Delete** button where the **Status** is **Active**.
6. A popup will show up to confirm if you want to delete your access key. Since
this is a new account, we probably haven't used this access key and it won't
compromise anything so we'll click **Yes**.

You can always recreate your root access keys however it isn't recommended.


### Activate MFA on your root account.
Enable Multi-Factor Authentication for the root account so that even if
anyone finds out the account's password, it won't be compromised unless they
also have access to the device that has the authentication code. We will use the
[Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2)
app as our virtual MFA. There are also other authenticators you can use like
[Authy 2-Factor Authentication](https://play.google.com/store/apps/details?id=com.authy.authy).

1. Download the **Google Authenticator** app
[here](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2).
2. Login to the [AWS Management Console](https://console.aws.amazon.com/console/home),
then open **Services** then select **IAM**.
3. Open the dropdown **Activate MFA on your root account**, then click
**Manage MFA**.
4. Select **A Virtual Device**, then click **Next Step**.
5. A popup with a QR code will show up, scan it using your mobile phone. If you
can't scan it then click the **show secret key for manual configuration** and
type the very long secret configuration key to the app manually.
6. Type the six digit number displayed by your mobile device to the
**Authentication Code 1** box then wait up to 30 seconds for the device to
generate a new number then type the next six digit number into the
**Authentication Code 2** box.
7. Click **Next Step**, and the choose **Finish**.

The next time you login, the app will require you to enter an authentication
code for the google authenticatior.

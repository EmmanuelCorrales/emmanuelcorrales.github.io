---
layout: post
title:  "AWS: What every developer should do with their new account"
date:   2018-02-24 08:30:00 +0800
categories: aws iam
tags: [ aws, iam ]
---
This article is a compilation of what every developer learning AWS should do
with their new account. I'll demonstrate how to secure the root account by
locking away the root user and applying all the best practices recommended by
Amazon Web Services.

### Table of Contents
- [Delete the root access keys.](#delete_access_keys)
- [Activate MFA on the root account.](#activate_mfa)
- [Prefer an admin user over a root user.](#replace_root_user)
- [Setup the AWS CLI](#setup_aws_cli)

## <a name="delete_access_keys" />Delete the root access keys.
Access keys are used to programatically access the AWS Services with your
account. Root access keys have no restrictions and may fall to the wrong hands.
There is no better way to prevent that by deleting it. Even if you are the only
person in the world to have those access keys, there is a possibility that you
may accidentally do something bad and irreversible to your account with root
access. Practice granting any user even yourself with the **least privilege**.
Below are the steps to delete the root access keys via AWS Management Console.

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

Root access keys can be created anytime however it isn't recommended to be used
unless really needed.

## <a name="activate_mfa" />Activate MFA on your root account.
Enable Multi-Factor Authentication for the root account so that even if
anyone finds out the account's password, it won't be compromised unless they
also have access to the device that has the authentication code. I recommend
using the
[Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2)
app as our virtual MFA. There are also other authenticators you can use like
[Authy 2-Factor Authentication](https://play.google.com/store/apps/details?id=com.authy.authy).

1. Download the **Google Authenticator** app
[here](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2).
2. Login to the [AWS Management Console](https://console.aws.amazon.com/console/home),
then open **Services** then select **IAM**.
3. Open the dropdown **Activate MFA on your root account**, then click the
**Manage MFA** button.
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
code from the google authenticator.

## <a name="replace_root_user" />Prefer an admin user over a root user.
We are trying to refrain from using the root user by creating an admin user as
proxy. The admin user has less privileges than the root user but has enough to
manage most of the AWS Services.

### Create a group for Administrators.
There could be many administrators for a single AWS account. It is best practice
to assign policies to a group instead of an individual user. If you ever want to
revoke the user/users privilege as an admin then you'll just have to remove that
user from the group.

1. Click **Groups** under the **Dashboard**.
2. Click the **Create New Group** button.
3. Enter the group name then click the **Next Step** button.
4. Search for the **AdmininstratorAccess** policy, check it then click the
**Next Step** button.
Attaching this policy makes this group an Admin group.
5. Review if the *Group Name* and the attached policy is correct. There should
be **arn:aws:iam:aws:policy/AdministratorAccess** in the policies.

### Create an admin user.
Below are the steps to create an admin user. It won't cover securing it with
MFA however it is still recommended to do it.

1. Click **Users** under the **Dashboard**.
2. Click **Add user** button. This will be a proxy for the root user.
3. Enter **Admin** as the user name. **Admin** is just my recommended user name,
any user name will do.
4. Under the **Select AWS access type**, check both **Programmatic access** and
**AWS Management Console access**.
5. Select **Custom password** then enter the desired password. Since this user
is just a proxy for the root user and I'll be the only one using it then I don't
need an autogenerated password, however if I ever need to create another admin
user for someone else then I'll check both the **Autogenerated password** and
**Require password reset**.
6. Click the **Next Permissons** button.
7. Under **Add user to group** tab, check the **Administrators** group then
click the **Next: Review** button.
8. Review the details of the user then click **Create user** button.
9. Click **Download .csv** to download the access keys and store it in a safe
secure place. This will be used to access your AWS account programatically.

## <a name="setup_aws_cli" />Setup the AWS command line interface.
Install the command line interface.

{% highlight bash %}
pip install awscli --upgrade --user
{% endhighlight %}

Check if it was installed properly.

{% highlight bash %}
aws --version
{% endhighlight %}

Configure the AWS CLI with the downloaded access keys.
{% highlight bash %}
aws configure
{% endhighlight %}

It will ask for the *Access Key ID*, *Secret Access Key*, *default region* and
the output format like below.

{% highlight bash %}
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7IWANHEL
AWS Secret Access Key [None]: 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
Default region name [None]: ap-southeast-1
Default output format [None]: json
{% endhighlight %}

The **access key id** and **secret access key** above are not real access keys
so nobody can use those. The default region should be the region nearest to you,
see [Regions and Endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html).
The output format can be be left blank since json is the default format.

# gitlab-centos-7

Self-hosted GitLab Community Edition on a CentOS 7 cloud server.

<!-- TOC -->

- [gitlab-centos-7](#gitlab-centos-7)
  - [Project](#project)
    - [Setting up CentOS 7 server](#setting-up-centos-7-server)
    - [Installing GitLab Community Edition](#installing-gitlab-community-edition)
    - [Configure OpenSSL](#configure-openssl)
    - [Enable SSL for GitLab](#enable-ssl-for-gitlab)
    - [Testing connection from Workstation](#testing-connection-from-workstation)
  - [Adding Users and Groups to GitLab](#adding-users-and-groups-to-gitlab)
    - [Create a new admin account](#create-a-new-admin-account)
    - [Create a new group](#create-a-new-group)
    - [Assign a role to user for group](#assign-a-role-to-user-for-group)
    - [Create a new project](#create-a-new-project)
  - [References](#references)

<!-- /TOC -->

## Project

### Setting up CentOS 7 server

Before we begin, we're going to need a CentOS 7 cloud server to work with. Create a CentOS 7 cloud server and run the following on it:

```bash
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
sudo yum install -y curl policycoreutils-python openssh-server perl firewalld
sudo systemctl enable sshd
sudo systemctl start sshd
sudo systemctl enable firewalld
sudo systemctl start firewalld
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
```

Next, install a text-based email client and web browser.

```bash
sudo yum install -y lynx mutt
sudo yum update -y
```

### Installing GitLab Community Edition

Add the GitLab package repository.

```bash
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```

Next, install the GitLab package.

```bash
sudo yum install -y gitlab-ce
```

### Configure OpenSSL

Generate a private key for our certificate.

```bash
openssl genrsa -out key.pem 2048
```

Next, we will create a certificate signing request based on this new key. As we're not getting the certificate signed by a third party, just use default for all the prompts.

```bash
openssl req -new -sha256 -key key.pem -out csr.csr
```

*Note: In a production environment, you'll want to fill out the prompts based on your organisation needs.*

We'll sign the certificate request ourselves, and generate a certificate that GitLab will use to encrypt our web connections.

```bash
openssl req -x509 -sha256 -days 365 -key key.pem -in csr.csr -out certificate.pem
```

*Note: Since this is a self-signed certificate, the web browser will not trust it by default.*

### Enable SSL for GitLab

First, create a directory to hold the certificate files and give it proper permissions.

```bash
sudo mkdir /etc/gitlab/ssl
sudo chmod 700 /etc/gitlab/ssl
sudo cp *.pem /etc/gitlab/ssl
```

Next, we'll modify our GitLab configuration file to use these certificate files.

```bash
sudo vim /etc/gitlab/gitlab.rb
```

Modify the following lines:

*Note: You'll need to substitute in your server values for [PUBLIC_IP].*

```
external_url 'https://<PUBLIC_IP>'
nginx['ssl_certificate'] = "/etc/gitlab/ssl/certificate.pem"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/key.pem"
```

Reload the GitLab configuration file.

```bash
sudo gitlab-ctl reconfigure
```

### Testing connection from Workstation

Let's make sure that we can connect to the GitLab server from our development machine by navigating to our `https://[PUBLIC_IP]`.

The first thing that we will need to do is to set a new password for the `root` login of this server. Once this is done, sign in as `root` user with the new password.

## Adding Users and Groups to GitLab

### Create a new admin account

First, create a new admin user by clicking on `Add people` and following the prompts. You may use an internal email, such as `[USERNAME]@[PUBLIC_IP]`, which we'll access from our server using `mutt`.

*Note: You'll need to substitute in your CentOS 7 server values for [USERNAME] and [PUBLIC_IP].*

Next, we'll confirm our new account by accessing our internal inbox. Copy the confirmation URL from the email sent by GitLab and use the `lynx` text browser to navigate to it.

```bash
mutt
lynx "https://[CONFIRMATION_URL]"
```

Accept any default prompts and set a new password for your new admin user account.

### Create a new group

Before creating a new group, relogin to GitLab server using your new admin account. Click on `Create a group` and follow the prompts.

You may now click on `Add people` to create a new user for your group.

### Assign a role to user for group

* Guest
* Reporter
* Developer
* Master
* Owner

### Create a new project

Click on `Create a project` and follow the prompts.

## References

* [Download and install GitLab](https://about.gitlab.com/install/#centos-7?version=ce)
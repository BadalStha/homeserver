After the installation is done login to your user account:

```
homeserver login: your_username
password: your_password
```

The sudo won't be installed at first to install it follow this step:

```
su -
password: your_root_password
apt update
apt install sudo
```

Now the installation is done now lets set the user to be able to use sudo command:

```
usermod -aG sudo yourusername
exit
```

Now log back in with user account details then try:

```
sudo apt install
sudo apt upgrade -y
```
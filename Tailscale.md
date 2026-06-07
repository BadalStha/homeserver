First install tailscale in your device if you are on windows then you can install easily from here: 

```
https://tailscale.com/download/windows
```

If you are on linux then these installation step will be same for you device and your server:

```
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Then it will redirect you to browser if it did not then just `CTRL + Right Click` Then it will open in browser then just sign in  all done.

To check if it was installed successfully:

```
tailscale status
```

To check which which devices are connected in your tailscale network:

```
tailscale ip
```
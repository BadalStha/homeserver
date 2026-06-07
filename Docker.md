Install docker with:

```
sudo apt install docker.io
```

Then start and enable the service:

```
sudo systemctl enable --now docker
```

Verify:

```
docker --versionsudo docker run hello-world
```

Now install docker compose with:

```
sudo apt install docker-compose
```

Add your user to the docker group so you don't need `sudo`:

```
sudo usermod -aG docker $USERnewgrp docker
```
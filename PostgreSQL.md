
Create a directory and a file inside it:

```
mkdir -p postgresql
cd postgresql
nano docker-compose.yml
```

Insert inside this file `docker-compose.yml`:

```
services:  
	postgres:  
		image: postgres:16  
		container_name: postgres  
		restart: always  
		environment:  
			POSTGRES_USER: db_username
			POSTGRES_PASSWORD: db_password  
		ports:  
			- "5432:5432"  
		volumes:  
			- pgdata:/var/lib/postgresql/data  
  
	pgadmin:  
		image: dpage/pgadmin4  
		container_name: pgadmin  
		restart: always  
		environment:  
			PGADMIN_DEFAULT_EMAIL: youremail@gmail.com
			PGADMIN_DEFAULT_PASSWORD: db_password
		ports:  
			- "8080:80"  
		depends_on:  
			- postgres  
  
volumes:  
	pgdata:
```

Start the containers:

```
docker compose up -d
```

Verify:

```
docker ps
```

You should see both `postgres` and `pgadmin` running.

---

### Access pgAdmin

Open:

```
http://serverip:8080
```

Login with:

```
Email: youremail@gmail.com
Password: strongpassword
```

---

### Connect PostgreSQL in pgAdmin

1. Right-click **Servers**
2. Click **Register → Server**

#### General Tab

```
Name: postgres
```

#### Connection Tab

```
Host name/address: postgres
Port: 5432
Maintenance database: postgres
Username: db_username
Password: db_password
```

Enable:

```
✓ Save password
```

Click **Save**.

**Important:** Use `postgres` as the hostname because both containers are on the same Docker Compose network.

---

### Create a Database

Navigate to:

```
Servers
└── postgres
└── Databases
```

Right-click **Databases** → **Create** → **Database**

Example:

```
Database name: 
myappOwner: badal
```

Click **Save**.
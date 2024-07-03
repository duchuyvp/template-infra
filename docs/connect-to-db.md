## This page describes how to connect to the database from outside and inside Local Network

### 1. What is Local Network I am talking about?

If you are using a computer that connected to wifi or LAN in my house, then you are in the Local Network. If you are using a computer that connected to the internet in a coffee shop or your house, then you are not in the Local Network.

### 2. Connecting to the database from inside Local Network

Devices in the Local Network can connect to the database using the IP address of the computer that runs the database.
If nothing goes wrong, the IP address of the computer that runs the database is `192.168.1.100`.
Currently, the following services and port respectively are provided:

- `MySQL: 3306`
- `PostgreSQL: 5432`
- `Redis: 6379`
- `Redis sentinel: 26379`
- `Elasticsearch: 9200`
- `Other Elasticsearch: 9300` (I don't know what this is,  but anyway)
- `MinIO: 9000`

### Example

Example code to connect to the MySQL using `sqlalchemy`:

```python
from sqlalchemy import create_engine
engine = create_engine('mysql://username:password@192.168.1.100:3306/database')

engine.execute("SELECT * FROM table_name")
```

### 3. Connecting to the database from outside Local Network (using Cloudflared)

#### 3.1. Install Cloudflared

You can download the Cloudflared from the [official website](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/).

Check if the installation is successful by running the following command:

```bash
cloudflared -v
cloudflared version 2024.6.1 (built 2024-06-18-1400 UTC)
```

#### 3.2. Create a TCP tunnel

Run the following command to create a TCP tunnel:

```bash
cloudflared access tcp --hostname <public hostname> --url tcp://localhost:<port>


```  

More information about this command do can be found [here](https://developers.cloudflare.com/cloudflare-one/applications/non-http/arbitrary-tcp/).

The following hostnames are available:

- `MySQL: mysql.hoaem.vn`
- ...

#### 3.3 Connect to the database

After creating the tunnel, you can connect to the database using the hostname and localhost port you provided in the previous step.

Example code to connect to the MySQL using `sqlalchemy`:

```python
from sqlalchemy import create_engine
engine = create_engine('mysql://username:password@localhost:<port>/database')

engine.execute("SELECT * FROM table_name")
```

#### Example

```bash
cloudflared access tcp --hostname mysql.hoaem.vn --url tcp://localhost:9320
```

```python
from sqlalchemy import create_engine
engine = create_engine('mysql://username:password@localhost:9320/database')

engine.execute("SELECT * FROM table_name")
```

### 4. Credentials and Permissions

You will need the credentials to connect to databases. Please contact me for the credentials.

Once you have the credentials, you can connect to the database and do anything you want (create database, even drop the database, etc.). I haven't set up any limit permission yet, but I will do it in the future.

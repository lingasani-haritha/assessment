---

## 📝 Solutions & Explanations

### 1️⃣ Start the Database

**Approach:**  
Docker was new to me, so I learned how to build and run the image in GitHub Codespaces.  
**Key Decisions:**  
- Used `docker build` and `docker run` with proper environment variables for password.
- Mapped port 5432 for local access.
- Verified container status with `docker ps`.

**Commands:**
```bash
docker build -t collate-postgres -f docker/Dockerfile_postgres .
docker run -d --name collate-pg -p 5432:5432 -e POSTGRES_PASSWORD=password collate-postgres
docker ps
```
**Challenge:**  
Container exited quickly due to password format; resolved by troubleshooting env variables.

**Screenshot:**  

<img width="940" height="239" alt="image" src="https://github.com/user-attachments/assets/875e7bbf-380c-4fb7-ba1d-c68e9552691f" />


---
---

### 2️⃣ Connect to the Database with Python

**Approach:**  
Installed `psycopg2-binary` and wrote queries in `db_connect.py` to explore the database.

**Commands:**
```bash
pip3 install psycopg2-binary
python db_connect.py
```

**Queries Used:**
```sql
-- 1. Test connection
SELECT version();

-- 2. Count tables
SELECT COUNT(*) FROM information_schema.tables WHERE table_type = 'BASE TABLE';

-- 3. List table names
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_type = 'BASE TABLE'
  AND table_catalog = 'dvdrental'
  AND table_schema NOT IN ('pg_catalog', 'information_schema')
ORDER BY table_schema, table_name;

-- 4. % distinct first_names in actor
SELECT ROUND(100.0 * COUNT(DISTINCT first_name) / COUNT(first_name), 2) AS distinct_percentage
FROM actor;
```

**Output:**
```
PostgreSQL version: PostgreSQL 14.18 (Debian 14.18-1.pgdg120+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 12.2.0-14) 12.2.0, 64-bit

--------------------------------------------------

Total number of tables (all schemas): 75

All tables (schema.table):
information_schema.sql_features
information_schema.sql_implementation_info
...
public.actor
public.address
public.category
public.city
public.country
public.customer
public.film
public.film_actor
public.film_category
public.inventory
public.language
public.payment
public.rental
public.staff
public.store

--------------------------------------------------

Total number of user tables (excluding system schemas): 15

User tables (schema.table):
public.actor
public.address
public.category
public.city
public.country
public.customer
public.film
public.film_actor
public.film_category
public.inventory
public.language
public.payment
public.rental
public.staff
public.store

--------------------------------------------------

Percentage of distinct first_names in actor table: 64.00%
```

**Screenshot:**  
![SQL Query Results](images/db_connect_output.png)

---
### 2️⃣ Connect to the Database with Python

**Approach:**  
Installed `psycopg2-binary` and wrote queries in `db_connect.py` to explore the database.

**Commands:**
```bash
pip3 install psycopg2-binary
python db_connect.py
```

**Queries:**
SELECT version();
SELECT table_schema, table_name
            FROM information_schema.tables
            WHERE table_type = 'BASE TABLE'
              AND table_catalog = 'dvdrental'
              AND table_schema NOT IN ('pg_catalog', 'information_schema')
            ORDER BY table_schema, table_name
SELECT ROUND(100.0 * COUNT(DISTINCT first_name) / COUNT(first_name), 2) AS distinct_percentage
            FROM actor;

**Output:**
- PostgreSQL version detected
- Total tables: 75
- User tables: 15
- % distinct first_names in actor: 64.00%

**Screenshot:**  
![SQL Query Results](images/db_connect_output.png)

---

### 3️⃣ Debugging Operational Errors

#### 3.1 Password Authentication Failed

**Explanation:**  
Occurs when credentials are incorrect.  
**Resolution Steps:**  
- Verify username/password
- Test manual login with `psql`
- Reset password if needed
- Check connection string and `pg_hba.conf`
- Restart PostgreSQL after changes

**Sample Code:**
```python
from sqlalchemy.exc import OperationalError

try:
    with engine.connect() as conn:
        conn.execute(text("SELECT 1"))
except OperationalError as e:
    if "password authentication failed" in str(e):
        print("Authentication failed: verify username and password.")
    else:
        print("Database error:", e)
```

#### 3.2 Connection Refused

**Explanation:**  
Occurs if server/container is not running or port mapping is incorrect.  
**Resolution Steps:**  
- Start container: `docker start collate-pg`
- Check port: `netstat -plnt | grep 5432`
- Verify Docker mapping: `docker ps`

**Screenshot:**  
![Operational Error](images/operational_error.png)

---

### 4️⃣ Fixing an API (FastAPI)

**Approach:**  
Installed FastAPI and Uvicorn.  
**Bug:**  
`ITEMS = ITEMS.append(item)` returns None, breaking the list.

**Fix:**
```python
@app.post("/items")
def create_item(item: Item) -> Item:
    ITEMS.append(item)
    return item
```

**Curl Debugging:**  
Original curl was missing `-d` flag.  
**Correct Command:**
```bash
curl -X POST -H "Content-Type: application/json" -d '{"name": "Pizza", "price": 1.15}' http://localhost:1234/items
```
**Verified with GET and POST.**

**Screenshot:**  
![API Response](images/api_response.png)

---

### 5️⃣ Python SDK & APIs (Collate/OpenMetadata)

**Approach:**  
Installed OpenMetadata SDK, generated PAT token, and fetched table details for the required FQN.

**Sample Code:**
```python
from metadata.ingestion.ometa.ometa_api import OpenMetadata
from metadata.generated.schema.entity.services.connections.metadata.openMetadataConnection import (
    OpenMetadataConnection, AuthProvider,
)
from metadata.generated.schema.security.client.openMetadataJWTClientConfig import OpenMetadataJWTClientConfig

server_config = OpenMetadataConnection(
    hostPort="https://collate-hiring.free-1.getcollate.cloud/api",
    authProvider=AuthProvider.openmetadata,
    securityConfig=OpenMetadataJWTClientConfig(
        jwtToken="<YOUR-ACCESS-TOKEN>",
    ),
)
metadata = OpenMetadata(server_config)
```
**Output:**  
Table ID: `b58e3c0d-dc4f-401a-8b51-9dcfbfc567b0`  
Detailed info printed in terminal.

**Screenshot:**  
![Table Details](images/table_details.png)

---

## 🔑 Summary of Approach

- **Key Decisions:** Used Docker for DB, Python for queries, FastAPI for API, OpenMetadata SDK for Collate.
- **Challenges:** Docker password env, API bug, connection errors.
- **Resolutions:** Troubleshooting, code fixes, correct curl usage, verified outputs.

---

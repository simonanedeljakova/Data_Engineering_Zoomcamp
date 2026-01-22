# DOCKER-WORKSHOP
Workshop Codespaces

## Workshop file
[01-docker-terraform/docker-sql](https://github.com/DataTalksClub/data-engineering-zoomcamp/tree/d8854a70547c60a49233c7c124bb7fc9e919cb61/01-docker-terraform/docker-sql)

## 1. INTRODUCTION
1. Create repository
   - README - on
   - public
   - gitignore - Python
2. Code -> create codespace on main
   - [my codespace]([https://probable-chainsaw-v69xxxvvvq74fxx7j.github.dev])
3. Code history
```bash
   history | tail
```

## Basic Docker commands
Python version (inside docker):
```bash
  python -V
```
  
Persistent minimal prompt: 
```bash
   > touch ~/.bashrc
   > echo 'PS1="> "' > ~/.bashrc
   > cat ~/.bash
   .bash_history  .bash_logout   .bashrc        
   > cat ~/.bashrc
   PS1="> "
```
   
Docker:
-    gives isolated container

Docker version check
```bash
docker --version
```

Python version check
```bash
python3 --version
```

```bash
docker run -it ubuntu
python3 -V
```

## Stateless containers
- when you exit the container and use it again, the changes are gone
- but, this is not _completely_ correct. The state is saved somewhere. We can see stopped containers:
```bash
docker ps -a
```

How to delete
```bash
docker rm `docker ps -aq`
```

## Managing containers
- for smaller container:
```bash
python:3.13.11.-slim
```

- rewrite to bash:
```bash
docker run -it --entrypoint=bash python:3.13.11-slim
```

- create python script (in EXPLORER) + test/list_files.py
```bash
python script.py
```

## Volumes
- we know that with docker we can restore any container to its initial state in a reproducible manner. But what about data? A common way to do so is with volumes:
-    create data in test
-    create a simple script
-    mapit to a Python container
-    inside container, run

## 2. VIRTUAL ENVIRONMENT
- data pipeline - s a service that receives data as input and outputs more data. For example, reading a CSV file, transforming the data somehow and storing it as a table in a PostgreSQL database.

We'll build pipelines that:
- Download CSV data from the web
- Transform and clean the data with pandas
- Load it into PostgreSQL for querying
- Process data in chunks to handle large files

## Creating pipeline
First create directory 'pipeline' and inside, create a file pipeline.py
```bash
import sys
print("arguments", sys.argv)

day = int(sys.argv[1])
print(f"Running pipeline for day {day}")
```

Then add pandas
```bash
import pandas as pd

df = pd.DataFrame({"A": [1, 2], "B": [3, 4]})
print(df.head())

df.to_parquet(f"output_day_{sys.argv[1]}.parquet")
```

Add Pandas
```bash
import pandas as pd

df = pd.DataFrame({"A": [1, 2], "B": [3, 4]})
print(df.head())

df.to_parquet(f"output_day_{sys.argv[1]}.parquet")
```

Do not forget to install Pandas
```bash
pip install pandas
```

## Virtual environments
Instal Python extension pyarrow
```bash
pip install pandas pyarrow
```

## Using uv - modern Python package manager
- much faster than pip and handles
```bash
pip install uv
```

Initialize s Python project with uv
```bash
uv init --python=3.13
```
- this creates a pyproject.toml file for managing dependencies and a .python-version file

Comparing Python versions
Addind dependencies
Running the Pipeline
Git Configuration

```bash
> git add .
> cd ..
> git add .
> git status
> git commit -m 'pipeline'
```

## 3. DOCKERIZING PIPELINE
## Create Dockerfile
- create Dockerfile inside of Pipeline folder

```bash
# base Docker image that we will build on
FROM python:3.13.11-slim

# set up our image by installing prerequisites; pandas in this case
RUN pip install pandas pyarrow

# set up the working directory inside the container
WORKDIR /app
# copy the script to the container. 1st name is source file, 2nd is destination
COPY pipeline.py pipeline.py

# define what to do first when the container runs
# in this example, we will just run the script
ENTRYPOINT ["python", "pipeline.py"]
```

Explanation:
FROM: Base image (Python 3.13)
RUN: Execute commands during build
WORKDIR: Set working directory
COPY: Copy files into the image
ENTRYPOINT: Default command to run

Note:
```bash
> rm Dockerfile
> cat > Dockerfile << 'EOF'
FROM python:3.12-slim
RUN pip install pandas pyarrow
WORKDIR /code
COPY pipeline.py .
ENTRYPOINT ["python", "pipeline.py"]
EOF
> cat Dockerfile
FROM python:3.12-slim
RUN pip install pandas pyarrow
WORKDIR /code
COPY pipeline.py .
ENTRYPOINT ["python", "pipeline.py"]
> docker run --rm test:pandas
> docker run -it --rm --entrypoint=bash test:pandas
```

## Build and run
```bash
docker build -t test:pandas .
```
- Note: these instructions assume that pipeline.py and Dockerfile are in the same directory. The Docker commands should also be run from the same directory as these files.

## Docker with uv
- instead of using pip
```bash
# Start with slim Python 3.13 image
FROM python:3.13.10-slim

# Copy uv binary from official uv image (multi-stage build pattern)
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

# Set working directory
WORKDIR /app

# Add virtual environment to PATH so we can use installed packages
ENV PATH="/app/.venv/bin:$PATH"

# Copy dependency files first (better layer caching)
COPY "pyproject.toml" "uv.lock" ".python-version" ./
# Install dependencies from lock file (ensures reproducible builds)
RUN uv sync --locked

# Copy application code
COPY pipeline.py pipeline.py

# Set entry point
ENTRYPOINT ["uv", "run", "python", "pipeline.py"]
```

## 4. POSTGRESQL WITH DOCKER
- real data engineering

## Running PostgreSQL in a container
- we will use the example folder ny_taxi_postgres_data
```bash
docker run -it --rm \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v ny_taxi_postgres_data:/var/lib/postgresql \
  -p 5432:5432 \
  postgres:18
```

Explanation of Parameters:
- -e sets environment variables (user, password, database name)
- -v ny_taxi_postgres_data:/var/lib/postgresql creates a named volume
      - Docker manages this volume automatically
      - Data persists even after container is removed
      - Volume is stored in Docker's internal storage
- -p 5432:5432 maps port 5432 from container to host
- postgres:18 uses PostgreSQL version 18 (latest as of Dec 2025)

Alternative Approach - Bind Mount
  - create the directory, then map it:

  ```bash
  mkdir ny_taxi_postgres_data

docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql \
  -p 5432:5432 \
  postgres:18
  ```

Add path:
```bash
> source /workspaces/Data_Engineering_Zoomcamp/pipeline/.venv/bin/activate
(pipeline) > ls
(pipeline) > cd ./pipeline/
(pipeline) > uv add --dev pgcli
```

Named Volume vs Bind Mount
- Named volume (name:/path): Managed by Docker, easier
- Bind mount (/host/path:/container/path): Direct mapping to host filesystem, more control

## Connecting to PostgreSQL
- Once the container is running, we can log into our database with pgcli

```bash
uv add --dev pgcli
```

Use it to connect to Postgres:
```bash
uv run pgcli -h localhost -p 5432 -u root -d ny_taxi
```

Note: password is root (not visible in command)

Check tables:

```bash
\dt
```

## Basic SQL Commands
```bash
-- List tables
\dt

-- Create a test table
CREATE TABLE test (id INTEGER, name VARCHAR(50));

-- Insert data
INSERT INTO test VALUES (1, 'Hello Docker');

-- Query data
SELECT * FROM test;

-- Exit
\q
```

## 5. DATA INGESTION - NYC TAXI DATASET
- We will now create a Jupyter Notebook notebook.ipynb file which we will use to read a CSV file and export it to Postgres.

## Setting up Jupyter
```bash
uv add --dev jupyter
uv run jupyter notebook
```

## Nest steps
- go to the PORT
- open 8888
- copy token from results (e371ec08f1268779d28207866a7478731ab4bd8206839c37)
- add token in the website (password token)
- you will be redirected to Jupyter
- create NEW - Python 3 file
- rename to notebok
- show - remowe header
- path to Jupyter: df = pd.read_csv('https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz)

## Explore data
Jupyter notebook:
```bash
import pandas as pd

prefix = 'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow'
url = f'{prefix}/yellow_tripdata_2021-01.csv.gz'
url

ADD INSTRUCTION PART TO AVOID WARNING

df = pd.read_csv(url)

df.head()

df.dtypes

df.shape
```

## Handling data types
We have a warning: /tmp/ipykernel_25483/2933316018.py:1: DtypeWarning: Columns (6) have mixed types. Specify dtype option on import or set low_memory=False.

Instructions:
```bash
dtype = {
    "VendorID": "Int64",
    "passenger_count": "Int64",
    "trip_distance": "float64",
    "RatecodeID": "Int64",
    "store_and_fwd_flag": "string",
    "PULocationID": "Int64",
    "DOLocationID": "Int64",
    "payment_type": "Int64",
    "fare_amount": "float64",
    "extra": "float64",
    "mta_tax": "float64",
    "tip_amount": "float64",
    "tolls_amount": "float64",
    "improvement_surcharge": "float64",
    "total_amount": "float64",
    "congestion_surcharge": "float64"
}

parse_dates = [
    "tpep_pickup_datetime",
    "tpep_dropoff_datetime"
]

df = pd.read_csv(
    url,
    nrows=100,
    dtype=dtype,
    parse_dates=parse_dates
)
```

## Ingesting data into Postrges
In the Jupyter notebook, we create code to:
1. Download the CSV file
2. Read it in chunks with pandas
3. Convert datetime columns
4. Insert data into PostgreSQL using SQLAlchemy

Exact steps:
- install SQLAlchemy
- create database connection
- get DDL schema
   - output
- create table

Go back to Github pyproject.toml
```bash
> ls
README.md  pipeline  test
> cd ./pipeline/
> uv run pgcli -h localhost -p 5432 -u root -d ny_taxi
Password for root: 
Using local time zone Etc/UTC (server uses Etc/UTC)
Use `set time zone <TZ>` to override, or set `use_local_timezone = False` in the config
Server: PostgreSQL 18.1 (Debian 18.1-1.pgdg13+2)
Version: 4.4.0
Home: http://pgcli.com
root@localhost:ny_taxi> \dt
+--------+------------------+-------+-------+
| Schema | Name             | Type  | Owner |
|--------+------------------+-------+-------|
| public | test             | table | root  |
| public | yellow_taxi_data | table | root  |
+--------+------------------+-------+-------+
SELECT 2
Time: 0.014s
root@localhost:ny_taxi>
```
## Ingesting data in chunks
- we don't want to insert all the data at once. Let's do it in batches and use an iterator for that

```bash
df_iter = pd.read_csv(
    dtype=dtype,
    parse_dates=parse_dates,
    iterator=True,
    chunksize=100000
)
```
- iterate over chunks
- inserting data
- complete ingestion loop
- alternative approach (without first flag)
- adding progress bar
- verify the data

## 6. INGESTION SCRIPT
- let's convert the notebook to a Python script

## Convert notebook to script (Jupyter)
```bash
uv run jupyter nbconvert --to=script notebook.ipynb
mv notebook.py ingest_data.py
```

## Complete ingestion script
- See the pipeline/ directory for the complete script with click integration. Here's the core structure:
```bash
import pandas as pd
from sqlalchemy import create_engine
from tqdm.auto import tqdm

dtype = {
    "VendorID": "Int64",
    "passenger_count": "Int64",
    "trip_distance": "float64",
    "RatecodeID": "Int64",
    "store_and_fwd_flag": "string",
    "PULocationID": "Int64",
    "DOLocationID": "Int64",
    "payment_type": "Int64",
    "fare_amount": "float64",
    "extra": "float64",
    "mta_tax": "float64",
    "tip_amount": "float64",
    "tolls_amount": "float64",
    "improvement_surcharge": "float64",
    "total_amount": "float64",
    "congestion_surcharge": "float64"
}

parse_dates = [
    "tpep_pickup_datetime",
    "tpep_dropoff_datetime"
]
```

## Click integration
- the script uses click for command-line argument parsing:
```bash
import click

@click.command()
@click.option('--pg-user', default='root', help='PostgreSQL user')
@click.option('--pg-pass', default='root', help='PostgreSQL password')
@click.option('--pg-host', default='localhost', help='PostgreSQL host')
@click.option('--pg-port', default=5432, type=int, help='PostgreSQL port')
@click.option('--pg-db', default='ny_taxi', help='PostgreSQL database name')
@click.option('--target-table', default='yellow_taxi_data', help='Target table name')
def run(pg_user, pg_pass, pg_host, pg_port, pg_db, target_table):
    # Ingestion logic here
    pass
```

## Running the script
- the script reads data in chunks (100,000 rows at a time) to handle large files efficiently without running out of memory.

Example usage:
```bash
uv run python ingest_data.py \
  --pg-user=root \
  --pg-pass=root \
  --pg-host=localhost \
  --pg-port=5432 \
  --pg-db=ny_taxi \
  --target-table=yellow_taxi_trips
```

# DOCKER-WORKSHOP
Workshop Codespaces

## Workshop file
[01-docker-terraform/docker-sql](https://github.com/DataTalksClub/data-engineering-zoomcamp/tree/d8854a70547c60a49233c7c124bb7fc9e919cb61/01-docker-terraform/docker-sql)

## 1. INTRODUCTION
1. create repository
   - README - on
   - public
   - gitignore - Python
2. code -> create codespace on main
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

But, this is not _completely_ correct. The state is saved somewhere. We can see stopped containers:
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

NOTE:
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


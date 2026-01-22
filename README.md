# DOCKER-WORKSHOP
Workshop Codespaces

## Workshop file
[01-docker-terraform/docker-sql](https://github.com/DataTalksClub/data-engineering-zoomcamp/tree/d8854a70547c60a49233c7c124bb7fc9e919cb61/01-docker-terraform/docker-sql)

## 01. INTRODUCTION
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

## 02. VIRTUAL ENVIRONMENT
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


  

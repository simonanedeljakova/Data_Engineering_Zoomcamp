## Workshop file
[01-docker-terraform/docker-sql](https://github.com/DataTalksClub/data-engineering-zoomcamp/tree/d8854a70547c60a49233c7c124bb7fc9e919cb61/01-docker-terraform/docker-sql)

## Introduction
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


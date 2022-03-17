# Docker.git-annex
Docker container with git-annex and a tutorial built in.

These are my initial notes based on [notes](http://lists.nmglug.org/pipermail/nmglug-nmglug.org/2022-March/005881.html) from [Art Barnes](https://github.com/bluejuniper).

Eventually, I'd like to get these into a Jupyter notebook, which is more suited for a step-by-step tutorial.

## Create a Docker image
```bash

# == Create the container
( docker run \
  --rm \
  --name gitannex \
  ubuntu sleep inf & sleep 1
)

# == Configure the container
( cd ~/Downloads &&
  docker container cp git-annex-tut.sh gitannex:/tmp/
)

docker exec -i gitannex /bin/bash <<'eof1'
  export DEBIAN_FRONTEND=noninteractive
  apt-get update && apt-get install -y git-annex tree vim curl less

  ## customize environment
  mkdir -p ~/src
  cd ~/src
  git clone https://github.com/rwcitek/bash_customization.git
  cat <<'eof' > ~/.bash_aliases
  for i in $( find ~/src/bash_customization/ -type f -name '*.bash_*' | sort ) ; do
    source $i
  done
  export HISTCONTROL=ignoredups:ignorespace;
  export HISTFILESIZE=50000;
  export HISTSIZE=50000;
  export HISTTIMEFORMAT='%t%F %T%t';
  export PAGER='less -iX ';
  export IGNOREEOF=20;
  export PS1='\u@\h: \w\n\$ ' ;
  [ -d ~/bin ] && export PATH=~/bin:${PATH}
eof
eof1

docker image rm gitannex &&
docker container commit gitannex gitannex &&
docker container stop gitannex

```
Eventually, the above will get converted into a Dockerfile.

## Run a Docker Container
```bash
# create gitannex container
( docker run \
  --rm \
  --name gitannex \
  gitannex sleep inf & sleep 1
)

# == Run git annex tutorial in interactive container
docker exec -it gitannex /bin/bash
```

## Tutorial
```bash
# == Run git annex tutorial in container
# configure git
git config --global user.email "demo@example.com"
git config --global user.name "demo"

# create environment
tmpdir=/tmp/git-annex-demo
mkdir -p ${tmpdir}
cd ${tmpdir}
mkdir -p wallpapers wallpapers-bare

# On the laptop
cd ${tmpdir}/wallpapers

## initialize git and git annex
git init
git annex init

## create sample files
echo " world" > world.txt
echo -n hello, > hello.txt
ls -l
cat hello.txt world.txt 

## work with annex
git status

git annex add . # by default files are moved into .git/annex and replaced by symlinks
git status
ls -l
cat hello.txt  world.txt 

git commit -am 'add files'
git status
git ls-files

git annex unlock .  # replace symlinks with 2nd copy of files
git status
ls -l
cat hello.txt  world.txt 

git annex lock . # replace files with symlinks again
git status
ls -l
cat hello.txt  world.txt 


# On the pretend bare server
cd ${tmpdir}/wallpapers-bare
git init --bare
ls -l


# On the laptop
cd ${tmpdir}/wallpapers
git branch -a
git remote show | xargs -n1 -r git remote show

git remote add server-bare ${tmpdir}/wallpapers-bare
git branch -a
git remote show | xargs -n1 -r git remote show

git annex sync
git branch -a
git remote show | xargs -n1 -r git remote show


# On the pretend bare server
cd ${tmpdir}/wallpapers-bare
git branch -a
git remote show | xargs -n1 -r git remote show


# Clone to server2
cd ${tmpdir}/
git clone wallpapers-bare wallpapers-server
cd ${tmpdir}/wallpapers-server
ls -l

git annex init
ls -l


# On the laptop
cd ${tmpdir}/wallpapers

git branch -a
git remote remove server-bare
git branch -a
git remote show | xargs -n1 -r git remote show

git remote add server ${tmpdir}/wallpapers-server
git branch -a
git remote show | xargs -n1 -r git remote show

git annex sync # git push server master will also work 
git branch -a
ls -l
cat hello.txt  world.txt 

## create some sample files, webp and jpg
for i in {a..d} ; do echo ${i} > ${i}.webp ; done
for i in {e..h} ; do echo ${i} > ${i}.jpeg ; done
ls -l

git annex copy -t server *.webp # copy webp files to the server

git annex drop *.webp # this will remove all webp files from .git/annex

git annex drop *.jpg # this will fail as jpg files have not been copied

git annex get . # copy back the deleted files


# Clone to a flash drive
cd ${tmpdir}/
git clone wallpapers-server wallpapers-flash-drive  # create a copy on the same machine, such as a flash drive

cd ${tmpdir}/wallpapers-flash-drive
git branch -a
git annex init
git branch -a

git remote add laptop ${tmpdir}/wallpapers
git branch -a
git remote show | xargs -n1 -r git remote show

# On laptop
cd ${tmpdir}/wallpapers
git branch -a
git remote show | xargs -n1 -r git remote show

git remote add flash-drive ${tmpdir}/wallpapers-flash-drive
git branch -a
git remote show | xargs -n1 -r git remote show

git annex sync
git branch -a

# On flash drive
cd ${tmpdir}/wallpapers-flash-drive
git annex get . # copy all files from the original repo

exit
```

## Cleanup
```bash
# == Stop and remove the container
docker container stop gitannex 
```





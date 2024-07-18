# Archivebox Setup on Reldo

Reldo is running off the dev branch of ArchiveBox via pm2. This version is not a stable release and requires a few patches to make it work. Once a stable version with the REST API comes out, we should move to that.

## Prereqs

DO NOT USE ROOT ACCOUNT. There's an `archivebox` user on the Reldo box.
ArchiveBox runs on python. I was unable to get it working with Python 3.12, but was able with 3.9.6 and 3.11.9. YMMV.
Virtual environment for python3. Highly recommend since we are installing archivebox via pip.

## Instructions to rebuild

```
# Might need to get pyenv if you don't have it on the box. See the repo for instructions https://github.com/pyenv/pyenv

# Needed to install these before installing 3.11.9, otherwise archivebox version would fail on sqlite3 not existing
sudo apt install zlib1g zlib1g-dev libssl-dev libbz2-dev libsqlite3-dev # pyenv common issues https://github.com/pyenv/pyenv/wiki/Common-build-problems

# Setup a venv via pyenv and activate
cd ~
pyenv install 3.11.9
pyenv virtualenv 3.11.9 reldo
pyenv local reldo
pyenv activate

# Clone the repo
git clone https://github.com/weirdgloop/ArchiveBox.git
cd ArchiveBox
git checkout wiki-branch # I (andmcadams) made some changes on here that you will probably want.

# Needed to run this to get python-ldap to build properly
sudo apt-get install libsasl2-dev python-dev-is-python3 libldap2-dev libssl-dev

# Install requirements and subrequirements of Archivebox
pip install -r requirements.txt
git submodule update --init --recursive # Had to downgrade pocket to 0.3.6 since 0.3.7 doesnt seem to exist, you may not need to do this
git pull --recurse-submodules
pip install -r  archivebox/vendor/requirements.txt 

# Install archivebox itself as a module
pip install .

# From the archivebox install guide
python3 -m archivebox version


# Setting up an archive
mkdir -p ~/test_archive/data && cd ~/test_archive/data   # for example, name can be anything
python3 -m archivebox init --setup   # instantialize a new collection


# MAKING CODE CHANGES #
# Need to make changes to the code on the root account
# git pull/checkout or change code directly

# Then you need to install the updated package on the archivebox user
su - archivebox
cd ~/reldo-archivebox
pip install ArchiveBox
cd ~/archive
# pm2 start ./start_archivebox_server
```

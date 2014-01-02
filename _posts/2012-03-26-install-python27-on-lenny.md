---
layout: post
title: Install python2.7 on lenny
tags:
- python
---

debian lenny (or ubuntu lucid for that matter) does not ship with python2.7, so you have to install it from its source. Run everything as root.


``` sh
aptitude install libsqlite3-dev zlib1g-dev libssl-dev libbz2-dev libncurses5-dev libreadline6-dev
#for tkinter support also install tcl-dev libgdbm-dev
cd /opt
wget -O- http://python.org/ftp/python/2.7.2/Python-2.7.2.tgz | tar xz
cd Python-2.7.2
./configure --with-threads --enable-shared --prefix=/opt/python2.7
make
make altinstall
ln -s /opt/python2.7/lib/libpython2.7.so.1.0 /usr/lib64/
ln -s /opt/python2.7/lib/libpython2.7.so /usr/
cd /opt/python2.7
wget http://peak.telecommunity.com/dist/ez_setup.py
/opt/python2.7/bin/python2.7 ez_setup.py
wget -O- http://pypi.python.org/packages/source/p/pip/pip-1.0.tar.gz | tar xz
cd pip-1.0
/opt/python2.7/bin/python2.7 setup.py install
/opt/python2.7/bin/pip install virtualenv
```

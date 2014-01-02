---
layout: post
title: "Installing the environment for the PyCon2012 pandas tutorial"
tags:
- pandas
- pycon 2012
- python
---

This post shows you how to setup the required environment for the pandas tutorial, given at the PyCon 2012
Information for the talk/tutorial: [https://us.pycon.org/2012/schedule/presentation/427/](https://us.pycon.org/2012/schedule/presentation/427/)
The talk is also available at [YouTube](http://www.youtube.com/watch?v=w26x-z-BdWQ).

*This is written against a 64bit linux/ubuntu*


1.  You need a bunch of packages installed via apt(itude|-get)

    ``` sh
    sudo aptitude install build-essential libzmq-dev python2.7-dev libblas-dev liblapack-dev libsuitesparse-dev swig gfortran libfreetype6-dev libpng12-dev
    ```

2.  If you don't mind having all development headers installed, you can also use

    ``` sh
    sudo aptitude build-dep python-matplotlib python-scipy python-numpy
    ```

3.  Create a new virtualenv (I'll use virtualenvwrapper)

    ``` sh
    mkvirtualenv pandastutorial --no-site-packages
    workon pandastutorial
    ```

4.  install the required python packages (will take a while)

    ``` sh
    pip install ipython numpy pyzmq tornado
    pip install scipy matplotlib pandas
    ```

5.  Grab the tutorial package and run the server

    ``` sh
    mkdir -p ~/tmp/pandastutorial
    cd ~/tmp/pandastutorial
    wget http://dl.dropbox.com/u/11102422/PandasTutorialFiles.zip
    unzip PandasTutorialFiles.zip
    ```

6.  Verify you setup

    ``` sh
    ipython --pylab -c 'import pandas; print pandas.__version__'
    ```

    This must not show any errors and the pandas version has to be \>= 0.7

7.  Start the notebook app

    ``` sh
    ipython notebook --pylab=inline
    ```

    This should open you browser an 127.0.0.1:8888 (or on the first free port after 8888)

If you're not using a virtual environment, you can substitute the steps 1.-3. with the following commands:

``` sh
sudo aptitude install ipython python-numpy python-zmq python-tornado python-scipy python-matplotlib python-pip
sudo pip install pandas
```



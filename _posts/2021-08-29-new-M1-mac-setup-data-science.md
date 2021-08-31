---
layout: post
categories: "en"
title: "M1 MacBook setup for Data Science in 2021"
date: 2021-08-29
tags: blog
desc: "My to-do list to setup my new mac, and the issue I encounter"
comment_id: 9

---

I'm in the ongoing process of setting up my new MacBook Air (M1, 2020) for my data analytics projects plus grad-school homework, so I need a very *average* set up of Homebrew, Python, Pyenv, Jupyterlab, R; on top of some simple things for my blog like Jekyll and Ruby. While doing this, I've been encountering issues - mainly because of incompatibilities with python libraries that are not yet updated. I'll be logging these issues and hopefully whatever workaround I found to solve them.

<br>

### Steps to set up an M1 for Data Science

1. Install Xcode and Command Line tools first, as this takes the longest.

2. Install Homebrew:

    <pre>
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)" </pre>

    Add Homebrew to PATH:

    <pre>
    echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/camilaburne/.zprofile
    eval "$(/opt/homebrew/bin/brew shellenv)"</pre>

    Then check if it can run with:

    <pre>
    brew —version </pre>


3. Install GIT:
    <pre>
    brew install git </pre>

    Then, [authenticate via SSH to Github](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) so you can connect to your repos.

4. Unhide files with shift+command+dot and sudo chflags nohidden, and open my bash profile file just to check it out :)

    <pre>
    sudo chflags nohidden /Library/ ~/Library/

    user@Camilas-Air ~ % touch . bash_profile
    user@Camilas-Air ~ % open . bash_profile </pre>


5. Install Python3 [the "right" way](https://opensource.com/article/19/5/python-3-default-mac), without removing python 2.7 because that's a mistake I've made in the past 🤦🏼‍♀️

6. Install [pyenv](https://github.com/pyenv/pyenv#understanding-path), pipx, [pipenv](https://pipenv.pypa.io/en/latest/install/).

    pyenv instructions from its github repo
    <pre>
    brew update
    brew install pyenv </pre>

    pipenv instructions from its website
    <pre>
    pip install --user pipx
    pipx install pipenv </pre>

7. Install [jupyterlab with pyenv](
https://jupyterlab.readthedocs.io/en/stable/getting_started/installation.html).

    jupyterlab read the docs:
    <pre>
    pipenv install jupyterlab
    pipenv shell
    </pre>

    Some help to manage pyenv and jupyter, because I haven't got to handle this successfully:    
      - [Pyenv, virtualenv and using them with Jupyter](https://albertauyeung.github.io/2020/08/17/pyenv-jupyter.html)
      - [Jupyter with Pyenv](https://brandonrozek.com/blog/jupyterwithpyenv/)


    <br>

8. Install Ruby and Jekyll, [following this repo](https://github.com/monfresh/install-ruby-on-macos).

9. Download [R for macOS](https://cran.r-project.org/bin/macosx/) and [R Studio](https://www.rstudio.com/products/rstudio/download/#download).


### Issues encountered:

- I think I made a mistake when installing my Jupyter notebook (which I installed at the same time as Jup Lab), I get [this error](https://github.com/scipy/scipy/issues/13409), which I temporarily solved with:

    <pre>
    brew install openblas
    pip3 install --upgrade pip
    pip3 install --upgrade pip3 setuptools wheel
    </pre>

- I have another issue downloading the library hdbcli to install drivers for Sap Hana. Apparently it's not compatible with anything I have.

<br>

When I see those issue I realize I have no clue what I'm doing 🥲

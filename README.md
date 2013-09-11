# The Problem

When developing multiple Python projects concurrently, each having its own separate set of modules/packages which might conflict, and sometimes even its own version of Python, it is very hard to keep track of which Python interpreter and which set of libraries go with each project.

As for Python selection, an excellent tool called [Pythonz](https://github.com/saghul/pythonz) exists to simplify the installation, usage, and reinstallation of Python versions.

As for the package sets, another industry-standard tool called [virtualenv](http://pypi.python.org/pypi/virtualenv) enables you to use "virtual pythons" on top of your existing installation to isolate environments. To simplify the workflow, a [wrapper exists](http://www.doughellmann.com/projects/virtualenvwrapper/) that enables you to quickly switch between virtual environments.

The typical scenario (at least for some developers) is working on several projects concurrently - some may be for work, some personal, etc. Each set of projects needs its own *virtualenv*. However, the way virtualenv works, you must remember to **activate** the "right" environment before working on a project.

This process may be familiar to you as this sequence of actions:

1. start a shell
2. `workon` your favorite virtualenv
3. `python setup.py develop` your project and start working on it.

Although *virtualenvwrapper* and *virtualenv* work quite well, the above sequence has several annoying problems when switching projects and/or shell sessions.

## Problem no. 1: Wrong Environments

Running the wrong python can be a painful mistake. You can install your project in the global python installation (which means a lot of dependencies creeping in). Also one project being developed in one environment (via `setup.py develop`) can be a dependency in another environment, which can get you wasting a whole lot of time trying to regain control over your working environment. 

A quick look at the above set of actions reveals a great potential for getting the wrong environment by mistake. It gets even worse when using "console scripts" -- for instance, if you have `nose` installed in your global Python installation, then the `nosetests` script will be in your $PATH somewhere. However, if you try to use it to test a project being developed in a virtual environment, it will fail importing stuff.

## Problem no. 2: IDEs

Even if you got to activating your "right" environment, that only solves your problems for the current shell session. Once you start a new shell you're forced to deal with the same issue again. 

It gets even worse when you try to integrate various scripts and tools into your editors/IDEs. If you want to run a script from your project with `emacs`, for instance, you can't just run it as-is, because it relies on the virtual python installation that it was installed on. This gets you to mess around with your configuration schemes on a per-project basis hoping to get it right, and is generally a headache.

## Problem no. 3: Responsiveness

Many solutions involve a shell mode of some sort to switch your default python interpreter (like `pb switch` and `virtualenvwrapper.sh`). They are great, but sometimes they take some time to execute. This causes your `.bashrc` or `.zshrc` to take an extra second or so, and sometimes even more. I don't know about you, but my workflow includes opening and closing many tabs and panes, so this can get pretty annoying pretty fast.

# Using `pytoggle`

`pytoggle` is a very simple utility meant to be used alongside other tools like `pythonz`. It borrows the idea of [direnv](https://github.com/zimbatm/direnv) and adapts it to the problems discussed above. It is a "smart shortcut" that deduces the version of python you'd like to run and runs it for you.

## Step 1: Clone the Project

That's an easy one (NB the path used for the repo):
   
   $ git clone https://github.com/vmalloc/pytoggle.git ~/.pytoggle

Make sure that the shebang (`#!`) on top of `~/.pytoggle/pytoggle` makes sense in your environment. `pytoggle` itself is python-based, so it needs a concrete Python interpreter to run. Once you get that right, just type:

    $ ~/.pytoggle/pytoggle install

## Step 2: Fix your `$PATH`

Include this in your shell profile file (`.bashrc` for bash, `.zshrc` for zsh, etc.):

    export PATH=~/.pytoggle/aliases:$PATH

## Step 3: Specifying your configuration

`pytoggle` works best served with `pythonz`. Let's say your project resides in `~/projects/proj1`. You can install a version of python to act as your default, and create a virtualenv for your project:

    $ ~/.pythonz/pythons/CPython-2.7.5/bin/virtualenv proj1_venv

The above will normally create a virtualenv under `proj1_venv/`. Now edit a file named `.pytoggle.cfg` and put it anywhere above your project directory (for instance, in `~/projects/proj1`, or `~/projects/`):

    [bin_dir]
    path = /path/to/proj1_venv/bin

That's it! Now when you run python from within the `proj1` dir, you'll get the "right" python for you:

    $ cd ~/projects/proj1
    $ python
    Python 2.7.3 (default, Jul 23 2012, 21:34:02) 
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import sys
    >>> sys.executable
    '/path/to/proj1_venv/bin/python'
    
Also, when you run a python script in your project directory, the right python interpreter will be used, even if you're not in that directory:

    $ cd /tmp
    $ python ~/projects/proj1/script.py # this will run the proj1 venv python

## Step 4 (optional): Create More Aliases

By default, the `python`, `easy_install` and `pip` aliases are created for you. If you would like to alias more commands to be toggled between environments, just run `pytoggle alias`:

    $ pytoggle alias nosetests

The above will make `nosetests` choose the "right" virtual environment as well.




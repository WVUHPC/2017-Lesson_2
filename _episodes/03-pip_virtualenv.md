---
title: "Installing python packages with pip and virtualenv"
teaching: 45
exercises: 15
questions:
- "Key question"
objectives:
- "First objective."
keypoints:
- "First key point."
---

## pip

pip is a package manager for python packages.
You can install software locally using your home folder to store it.

### Installing packages

The basic command to install packages is

~~~
$ pip install --user PACKAGE_NAME
~~~
{: .source}

However, pip allow you to do much more, you can for example install packages
with specific versions

~~~
pip install PACKAGE_NAME            # latest version
pip install PACKAGE_NAME==1.0.4     # specific version
pip install 'PACKAGE_NAME>=1.0.4'     # minimum version
~~~
{: .source}

### Listing

To list just the packages you have installed locally execute:

~~~
pip list --user
~~~
{: .source}

You can also see the packages outdated

~~~
pip list --user --outdated
~~~
{: .source}

Show details about packages

~~~
pip show scipy
~~~
{: .source}

### Uninstalling

~~~
pip uninstall PACKAGE_NAME
~~~
{: .source}



## virtualenv

virtualenv is a tool to create isolated Python environments.
You can use virtualenv to control the libraries needed by one application.
If an application works, any change in its libraries or the versions
of those libraries can break the application.

The use is very simple.
First you need to create the environment.

~~~
$ virtualenv ENV
~~~
{: .source}

To activate it use:

~~~
$ source bin/activate
~~~
{: .source}

To deactivate the environment, just use

~~~
$ deactivate
~~~
{: .source}



{% include links.md %}

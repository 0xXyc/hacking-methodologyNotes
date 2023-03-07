# Virtual Environments (venv)

## Introduction

venv for Python3 and virtualenv for python2 allows you to manage seperate package installations for different projects.

They essentially allow you to create a "virtual" isolated Python installation and install packages into that virtual installation.

* A folder structure that gives you everything you need to run a lightweight yet isolated Python environment

{% embed url="https://realpython.com/python-virtual-environments-a-primer/" %}

## How-to

1. Create a directory: mkdir test
2. Change into directory: cd test
3. python3 -m venv venv
4. Activate venv: source venv/bin/activate
5. Run application with python normally: python3 app.py
6. Exit the venv: deactivate

**Make sure all packages are installed within the venv or you will run into some issues!**

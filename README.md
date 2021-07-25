# pydocs

[![Documentation Status](https://readthedocs.org/projects/python-learning-docs/badge/?version=latest)](https://python-learning-docs.readthedocs.io/en/latest/?badge=latest)

Python website where I document my learning materials to share with others.

 - Documentation[RTD](https://python-learning-docs.readthedocs.io/en/latest/)

# Building locally

 - pip install tox
 - python -m venv .venv
 - source .venv/bin/activate (unix) || .venv\Scripts\activate (windows)
 - tox -e docs
 - Access the docs via /docs/build/

# Editing docs with hot reloading

 - pip install tox
 - python -m venv .venv
 - source .venv/bin/activate (unix) || .venv\Scripts\activate (windows)
 - tox -e hotreload
 - Edit /docs and the browser will be auto reloaded!
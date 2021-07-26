# pydocs

[![Documentation Status](https://readthedocs.org/projects/symonk-pydocs/badge/?version=latest)](https://symonk-pydocs.readthedocs.io/en/latest/?badge=latest)

Python website where I document my learning materials to share with others.

 - [Click here for Docs](https://symonk-pydocs.readthedocs.io/en/latest/)

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
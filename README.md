# pydocs

Python website where I document my learning materials to share with others.

 - https://python-learning-docs.readthedocs.io/en/latest/

# Building locally

 - pip install tox
 - python -m venv .venv
 - source .venv/bin/activate (unix) || .venv\Scripts\activate (windows)
 - tox -e docs
 - Access the docs via /docs/build/

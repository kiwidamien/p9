---
title: "Publishing Python Packages"
author: "Damien Martin"
date: "2024-05-27 12:00"
categories: [python, package-management]
description: "How to push packages to artifactory with either setup.py or pyproject.toml"
---

Suppose you have a company python package index with the name `our-user-libs` (the default is to publish to PYPI).

There are two ways to publish to a package index:

* The old way: If you have `setup.py`
* The new way: If you have a `pyproject.toml` instead

## Old way (setup.py)

```shell
python setup.py sdist upload -r our-user-libs
```

## New way (pyproject.toml)

```shell
python -m twine upload -r our-user-libs dist/
```

You'll need to install the `twine` package first.

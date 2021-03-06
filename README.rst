======================
pipenv-to-requirements
======================

.. image:: https://travis-ci.org/gsemet/pipenv-to-requirements.svg?branch=master
    :target: https://travis-ci.org/gsemet/pipenv-to-requirements
.. image:: https://badge.fury.io/py/pipenv-to-requirements.svg
   :target: https://pypi.python.org/pypi/pipenv-to-requirements/
   :alt: Pypi package
.. image:: https://img.shields.io/badge/license-MIT-blue.svg
   :target: ./LICENSE
   :alt: MIT licensed

Generate ``requirements[-dev].txt`` from ``Pipfile`` (using ``pipenv``).
Used in my `modern Python module project cookiecutter <https://github.com/gsemet/python-module-cookiecutter>`_.
template.

* Free software: MIT
* Documentation: https://pipenv-to-requirements.readthedocs.org/en/latest/
* Source: https://github.com/gsemet/pipenv-to-requirements

Rational
--------

``Pipfile`` and its sibling ``Pipfile.lock`` are clearly superior tools defining clear dependencies
or a package. ``Pipfile`` is to be maintained by the package's developer while ``Pipfile.lock``
represent a clear image of what is currently installed on the current system, guarantying full
reproductibility of the setup. See more information about `Pipfile format here
<https://github.com/pypa/pipfile>`_. Most of the time, ``Pipfile.lock`` should be ignored (ie, not
tracked in your git) for packages published on Pypi.

`pipenv <https://github.com/kennethreitz/pipenv>`_ is a great tool to maintain ``Pipfile``, but
developers might be stuck with backward compatibility issues for tools and services that still use
`requirements.txt` and does not know how to handle ``Pipfile`` or ``Pipfile.lock`` yet.

For examples:

- `Read the Docs <https://github.com/rtfd/readthedocs.org/issues/3181>`_
- `Pyup <https://github.com/pyupio/pyup/issues/197>`_ (experimental support is
  `arriving <https://github.com/pyupio/pyup/issues/197>`_ )
- Any library that uses `PBR <https://docs.openstack.org/pbr/latest/>`_ (*)
- ``pip install`` (if you install a package with ``pip`` that does not have a ``requirements.txt``,
  its dependencies won't be installed, even if you use ``Pipfile``)

(*): for the moment, I recommend to generate at least ``requirements.txt`` (without version
freeze) for the libraries using PBR that you publish on Pypi, and commit this file into your git
history.
Remember PBR automatically synchronize the content of `requirements.txt` found at the root of your
package with the `install_requires` section of your package's `setup.py`.
This allows the automatic installation of the all production dependencies when "pip-installing"
your package .

Without this file, your package would still be installed by ``pip``, but without its own dependencies.
Support in PBR may be added in the future (see this
`this patch <https://review.openstack.org/#/c/524436/>`_ ).

For build reproductibility, I also recommend to **check in** your lock file even for libraries,
so that your CI won't fail when new packages is published on Pypi.

Usage
-----

Just before building source/binary/wheel package of your python module, only of the following
commands:

- To generate requirements files (ie, dependencies are described eventually by range), typically
  for **libraries**:

    .. code-block:: bash

        pipenv run pipenv_to_requirements

- To generate frozen requirements (ie, all dependencies have their version frozen), typically for
  **applications**:

    .. code-block:: bash

        pipenv run pipenv_to_requirements -f

It will generate ``requirements.txt`` and, if applicable, ``requirements-dev.txt``, in the current
directory.

- Also possible:

    .. code-block:: bash

        pipenv run pipenv_to_requirements -d requirements-dev-custom.txt
        pipenv run pipenv_to_requirements -d requirements-dev-custom.txt -f
        pipenv run pipenv_to_requirements -o requirements-custom.txt
        pipenv run pipenv_to_requirements -o requirements-custom.txt -f
        pipenv run pipenv_to_requirements  -d requirements-dev-custom.txt -o requirements-custom.txt -f

Example using a Makefile::

    dev:
        pipenv install --dev
        pipenv run pip install -e .

    dists: requirements sdist bdist wheels

    requirements:
        # For a library, use:
        pipenv run pipenv_to_requirements
        # For an application, use:
        # pipenv run pipenv_to_requirements -f

    sdist: requirements
        pipenv run python setup.py sdist

    bdist: requirements
        pipenv run python setup.py bdist

    wheels: requirements
        pipenv run python setup.py bdist_wheel

Just use `make requirements` to refresh the `requirements.txt`.

Read the Docs
-------------

Simply commit these files in your tree so that Read the Rocs, and ensure they are synchronized each
time you change your ``Pipfile``. Do not forget to ask Read the Docs to use ``requirements-dev.txt``
when building the documentation.


Contributing
------------

This package has been bootstrapped with Gsemet's
[Python-module-cookiecutter](https://github.com/gsemet/python-module-cookiecutter).

Create your development environment with

.. code-block:: bash

    $ make dev

Execute unit tests:

.. code-block:: bash

    $ make test

Code formatter:

.. code-block:: bash

    $ make style

Code Style Checks:

.. code-block:: bash

    $ make check

Build distribution packages with

.. code-block:: bash

    $ make dists

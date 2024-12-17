Installation
============

The SimScale SDK is hosted and published in GitHub. Following are the instructions for
the installation of the Python and C# libraries.

Python SDK
----------

The SDK library requires Python version >= 3.6. The installation can be done with pip,
directly from the repository, or via setuptools from source.

When installing python packages, it is recommended that you make use of virtual environments.

To install with pip, you can run the command:

.. code-block:: shell

    pip install git+https://github.com/SimScaleGmbH/simscale-python-sdk.git

If you want to install an specifiy version, you can run:

.. code-block:: shell

    pip install git+https://github.com/SimScaleGmbH/simscale-python-sdk.git@10.0.0

To install from source, first clone the repository:

.. code-block:: shell

    git clone https://github.com/SimScaleGmbH/simscale-python-sdk.git

Then you can use setuptools to perform the installation:

.. code-block:: shell

    cd simscale-python-sdk

    python setup.py isntall --user

To test the installation, open a python prompt and import the library:

.. code-bloc:: shell

    python

.. code-block:: python

    >>> import simscale_sdk

If you don't see a 'ModuleNotFoundError', then you should be good to go.


C# SDK
------

To Do...
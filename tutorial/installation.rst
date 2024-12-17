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

.. code-block:: console

    pip install git+https://github.com/SimScaleGmbH/simscale-python-sdk.git

If you want to install an specifiy version, you can run:

.. code-block:: console

    pip install git+https://github.com/SimScaleGmbH/simscale-python-sdk.git@10.0.0

To install from source, first clone the repository:

.. code-block:: console

    git clone https://github.com/SimScaleGmbH/simscale-python-sdk.git

Then you can use setuptools to perform the installation:

.. code-block:: console

    cd simscale-python-sdk

    python setup.py isntall --user

To test the installation, open a python prompt and import the library:

.. code-block:: console

    python

.. code-block:: python

    import simscale_sdk

If you don't receive a `ModuleNotFoundError`, then you should be good to go.


C# SDK
------

The C# SDK supports the following frameworks:

* .NET Core >= 1.0
* .NET Framework >= 4.6
* Mono/Xamarin >= vNext

The following dependencies must also be satisfied:

* RestSharp 106.10.1
* Json.NET 12.0.1
* JsonSubTypes

The DLLs included in the package are probably not the latest available version.
It is recommended to use NuGet to install the latest versions:

.. code-block:: console

    Install-Package RestSharp

    Install-Package Newtonsoft.Json

    Install-Package JsonSubTypes

In order to perform the installation, first clone the reposotiry:


.. code-block:: console

    git clone https://github.com/SimScaleGmbH/simscale-csharp-sdk.git

    cd simscale-csharp-sdk

Then generate the DLL using your preferred tool

.. code-block:: console

    dotnet build

The DLL can be found under the bin folder. You can link it in your project, then
include the relevant namespaces:

.. code-block:: csharp

    using SimScale.Sdk.Api;
    using SimScale.Sdk.Client;
    using SimScale.Sdk.Model;

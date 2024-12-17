API Keys
========

How to get an API Key
---------------------

Your personal API Key is generated in the SimScale platform. Follow these steps:

1. Sign in to the SimScale platform in `www.simscale.com <https://www.simscale.com/>`_
2. Open account menu at the top left
3. Go to 'Manage Account' or 'User Preferences', depending on wether you are in the Workbench
4. Open the 'API Keys' tab
5. Use the 'Generate Key' button
6. Enter a description for this new key
7. Copy the key to a safe location, for instance to your ``.env`` file

If doing so, your ``.env`` file should contain a line such as:

.. code-block:: python

    SIMSCALE_API_KEY=(generated api key)


API Client
----------

User authentication with the SimScale API is performed using the ``X-API-KEY`` request header.
In your code, you must first initialize and configure a ``simscale_sdk.ApiClient`` object, which
holds the data for the host, identification header and API Key. This object is later used to
setup the different Api clients specific to each platform action.

The following snippet shows an example of the creation and setup of the ``ApiClient`` object:

.. code-block:: python

    # Python

    import os
    from dotenv import load_dotenv()

    import simscale_sdk as sim

    load_dotenv()

    API_KEY = os.getenv('SIMSCALE_API_KEY')
    API_HOST_URL = "https://api.simscale.com/v0"

    configuration = sim.Configuration()
    configuration.host = API_HOST_URL
    configuration.api_key = {
        "X-API-KEY": API_KEY
    }

    api_client = sim.ApiClient(configuration)


.. code-block:: csharp
    
    // C#
    
    using System;
    using SimScale.Sdk.Api;
    using SimScale.Sdk.Client;
    using SimScale.Sdk.Model;
    using dotenv.net;

    DotEnv.Load();

    var API_KEY = System.Environment.GetEnvironmentVariable("SIMSCALE_API_KEY")
    var API_HOST_URL = "https://api.simscale.com/v0"

    Configuration configuration = new Configuration();
    
    configuration.BasePath = API_HOST_URL
    configuration.ApiKey.Add("X-API-KEY", API_KEY)

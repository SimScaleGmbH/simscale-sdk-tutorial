Simulation Project
==================

To Do:

* Explain what a simulation project is and what it contains
* How to create a simulation project with the SDK

A simulation project is a container  for the elements needed to setup a simulation, the two most
important of them being the CAD geometry and the simulation or physics setup. We can visualize
the hierarchy with a tree structure:

.. code-block::

    - Simulation project
    |
    |- Geometries
    | |
    | |- CAD 1
    | |- CAD 2
    |
    |- Simulations
      |
      |- Simulation 1
      |- Simulation 2


Eeach simulation is linked to one geometry, and each simulation setup contains the physics definition.
More details to follow in the specific steps.

In order to create and manipulate simulation projects, the SimScale SDK provides a Projects API client:

.. code-block:: python

    import simscale_sdk as sim

    projects_api = sim.ProjectsApi(api_client)


.. code-block:: csharp

    ...


We can see that the projects api object is initialized with the ``api_client``, which we previously
initialized with our Api key and host URL.

The project data is hosted in a ``Project`` object, which is then included as the argument of the
actual Api call to create the project:

.. code-block:: python

    import simscale_sdk as sim

    project_data = sim.Project(
        name="My first simulation project",
        description="Created via SDK",
        measurement_system="SI"
    )

    project = projects_api.create_project(project_data)

    project_id = project.project_id

    print(f"{project_id=}")


.. code-block:: csharp

    ...


You can see that the ``ProjectsApi.create_project`` method returns a ``Project`` object containing
the data of our original object, plus some additional data added after the project is created in the platform.
For instance, the ``project_id`` is a unique identification string for our newly created project.

The ``ProjectsApi`` object has many other useful methods to deal with simulations project data, such as:

* To get the data for an existing project ``ProjectsApi.get_project(project_id)``
* To update the data for an existing proejct ``ProjectsApi.update_project(project_id, project_data)``

More details on the Projects Api can be found in the 
`documentation <https://simscalegmbh.github.io/simscale-python-sdk/simscale_sdk.api.html#module-simscale_sdk.api.projects_api>`_

Geometry Upload and Import
==========================

All simulations in the SimScale platform are built on top of a geometry model, and physics concepts
such as material models, boundary conditions and interactions are assigned to geometrical elements
such as volumes and faces. 

The geometrical model is defined via a CAD file. Some of the CAD formats supported in the platform
include:

* Parasolid
* Solidworks
* Inventor
* Rinho
* CATIA
* ACIS
* STEP
* IGES
* STL

The full list can be found in the `documentation <https://www.simscale.com/docs/cad-preparation/>`_

For a CAD geometry to be available for simulation use via the SDK, a two-step process
is performed:

1. Upload the CAD file to the platform storage
2. Import the stored file into the project

After the geometry is imported to the project, it can be used to build a simulation on top of it.


Uploading the CAD File
----------------------

In order to upload a CAD file to the platform, a ``storage`` is first created, by making use of
the ``StorageApi`` module:


.. code-block:: python

    import simscale_sdk as sim

    storage_api = sim.StorageApi(api_client)

    storage = storage_api.create_storage()


Then we can read the CAD file stored locally and upload it with a ``PUT`` request. The ``URL``
for the request is available in the ``storage`` object:


.. code-block:: python

    with open("cad_file.x_t", "rb") as file:
        
        api_client.rest_client.PUT(
            url=storage.url,
            headers={
                "Content-Type": "application/octet-stream"
            },
            body=file.read()
        )


After the request is completed, the file is available in the SimScale's cloud storage and ready to
be imported into a simulation project. For instance, the storage is identified by an ``UUID``:


.. code-block:: python

    storage_id = storage.storage_id

    print(f"{storage_id=}")


Importing the Geometry into the Project
---------------------------------------

In order to deal with geometry data and the related operations, the SDK offers the 
``GeometryImportRequest`` object and the ``GeometryImportsApi`` module. We first define the 
relevant data for the CAD file, because the platform still doesn't know about the format, 
name, import options, etc., of the file, and then use the ``GeometryImportsApi.import_geometry`` 
function call to start the import operation:


.. code-block:: python

    import simscale_sdk as sim

    geometry_import_api = sim.GeometryImportsApi(api_client)

    geometry_import_req = sim.GeometryImportRequest(
        name="Geometry",
        location=sim.GeometryImportRequestLocation(storage_id),
        format="PARASOLID",
        input_unit="m",
        options=sim.GeometryImportRequestOptions(
            facet_split=False,
            sewing=False,
            improve=True,
            optimize_for_lbm_solver=False
        ),
    )

    geometry_import = geometry_import_api.import_geometry(project_id, geometry_import_req)

    geometry_import_id = geometry_import.geometry_import_id


The ``GeometryImportsApi.import_geometry`` method takes some time to complete its work, and is a 
non-blocking call because the action happens in the platform. In order to sync our code with 
the execution of the task, we create a loop to check the status of the operation at a given
frequency, which is every 10 seconds in this example:


.. code-block:: python

    import time

    while geometry_import.status not in ("FINISHED", "CANCELED", "FAILED"):

        geometry_import = geometry_import_api.get_geometry_import(project_id, geometry_import_id)

        time.sleep(10)


Notice how the loop executes while the status is finsihed (successfuly), canceled or failed. 
An improved version of this snippet also adds a time-out check:


.. code-block:: python

    import time

    GEOMETRY_IMPORT_TIMEOUT = 900

    import_start = time.time()

    while geometry_import.status not in ("FINISHED", "CANCELED", "FAILED"):

        if time.time() > import_start + GEOMETRY_IMPORT_TIMEOUT:
            raise TimeoutError()

        geometry_import = geometry_import_api.get_geometry_import(project_id, geometry_import_id)

        time.sleep(10)


When the loop exits, because the operation reaches one of the expected status, we can process 
the result, such as getting the id for the imported geometry:


.. code-block:: python

    if geometry_import.status != "FINISHED":
        raise Exception("Geometry import operation was canceled or failed.")

    geometry_id = geometry_import.geometry_id

    print(f"{geometry_id=}")


This is a common pattern that we will encounter on non-blocking operations that are launched 
with the API, but that we need to sync with because the results are to be used in suqsequent 
operations. Such cases would include mesh computation, simulation run execution, etc.

Also, this loop is a great opportunity for async execution break points. If you are running 
multiple such operations in a parallel asyncio loop, instead of waiting some seconds on a blocking 
``time.sleep()`` call, you can mark the hypervisor to switch tasks at this point. For instance, 
take a look at the following snippet:


.. code-block:: python

    import asyncio

    async def async_import_geometry(geoemetry_import_req, project_id):

        # Do the preparation tasks and launch the import

        while geometry_import.status not in ("FINISHED", "CANCELED", "FAILED"):

            if time.time() > import_start + GEOMETRY_IMPORT_TIMEOUT:
                raise TimeoutError()

            geometry_import = geometry_import_api.get_geometry_import(project_id, geometry_import_id)

            await asyncio.sleep(10)

        return geometry_import.geometry_id

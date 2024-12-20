Meshing
=======

The CFD numerical method used for the incompressible setup requires
the discretization of the fluid domain, a process also known as meshing.
In the Simscale platform, a mesh is an individual object that needs a model
to specify its properties, such as sizing. The mesh also has to be computed 
before the simulation can be run.

Mesh Model
----------

Similar to a simulation, we first need to specify the mesh parameters
with a model object. It can be done as follows:


.. code-block:: python

    import simscale_sdk as sim

    mesh_model = sim.SimmetrixMeshingFluid(
        sizing=sim.AutomaticMeshSizingSimmetrix(
            fineness=5,
            curvature=sim.AutomaticCurvature(),
        ),
        automatic_layer_settings=sim.AutomaticLayerOn(
            number_of_layers=3,
            total_relative_thickness=0.4,
            growth_rate=1.5
        ),
        pysics_based_meshing=True,
        hex_core=True,
    )


Here we make use of the automatic sizing with fineness level 5. Also, physics-based
meshing allows to take into account boundary conditions for layer creation. boundary
layer settings are entered manually with 3 layers on walls. Finally, the hex_core
is entered to improve the orthogonality of the mesh.

Mesh Operation
--------------

Now that we have a proper mesh model, we can create the mesh operation, which links
the project, the geometry and the model:


.. code-block:: python

    import simscale_sdk as sim

    mesh_operation_api = sim.MeshOperationsApi(api_client)

    mesh_operation = mesh_operation_api.create_mesh_operation(
        project_id,
        sim.MeshOperation(
            name="Pipe junction mesh",
            geometry_id=geometry_id,
            model=mesh_model,
        )
    )


The mesh operation object is used to launch the computation and track its progress:


.. code-block:: python

    mesh_operation_api.start_mesh_operation(
        project_id,
        mesh_operation.mesh_operation_id,
        simulation_id=simulation_id,
    )

    while mesh_operation.status not in ("FINISHED", "CANCELED", "FAILED"):

        mesh_operation = mesh_operation_api.get_mesh_operation(
            project_id,
            mesh_operation.mesh_operation_id,
        )

        time.sleep(30)

    print(f"Mesh with id={mesh_operation.mesh_id} was completed with status {mesh_operation.status}")


You should recognize this pattern from the geometry import process. Of course, it is
possible to improve the loop, as was done there, to include a time limit or use 
parallel async computation of multiple meshes.

Link Mesh to simulation
-----------------------

Finally, when the mesh is succesfully computed, we can link it to our simulation spec
so the platform knows that it should use it as part of the simulation computation:


.. code-block:: python

    # Might not be needed if the simulation_spec object is updated
    simulation_spec = simulations_api.get_simulation(project_id, simulation_id)

    simulation_spec.mesh_id = mesh_operation.mesh_id

    simulations_api.update_simulation(project_id, simulation_id, simulation_spec)


Now our simulation is ready for computation, with its physics model and a mesh
completely defined.

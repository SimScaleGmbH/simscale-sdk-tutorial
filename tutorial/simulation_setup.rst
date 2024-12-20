Simulation Setup
================

Having the geometry model already available in the project, the next step to 
setup a simulation is to specify the physics model. This is performed via a simulation 
``Spec`` (short for specification), an object that contains the following data:

* Name of the simulation
* Geometry ID
* Physical model

A ``SimulationSpec`` object is initialized with this data, then passed to the
``SimulationsApi.create_simulation()`` method:


.. code-block:: python

    import simscale_sdk as sim

    simulations_api = sim.SimulationsApi(api_client)

    simulation_spec = sim.SimulationSpec(
        name="Incompressible",
        geometry_id=geometry_id,
        model=model
    )

    simulation = simulation_api.create_simulation(project_id, simulation_spec)


Now we need to delve deeper into the physics ``model``.

Model
-----

The ``model`` object completely defines the characteristics and physical conditions 
of the system to be simulated, alongside the numerics properties and simulation control. 
For instance, the following data is captured in the model:

* Materials models
* Initial conditions
* Boundary conditions
* Interactions between multiple components (contacts)
* Numerical methods and parameters
* Simulated time and time stepping
* Computed output fields and derived data

Notice that this is a rather long amount of data, and as such it will be broken
part by part. Also, not all of this data is required to setup a simulation, and
the specifics will depend on the particular physics of the model.

For the sake of this tutorial, we will cover the ``Incompressible`` model as
an example:

.. code-block:: python

    import simscale_sdk as sim

    tutorial_model = sim.Incompressible(
        model=sim.FluidModel(),
        initial_conditions=sim.FluidInitialConditions(),
        advanced_concepts=sim.AdvancedConcepts(),
        materials=sim.IncompressibleFluidMaterials(
            ...
        ),
        numerics=sim.FluidNumerics(
            relaxation_factors=sim.RelaxationFactor(),
            pressure_reference_vale=sim.DimensionalPressure(value=0, unit="Pa"),
            residual_controls=sim.ResidualControls(
                velocity=sim.Tolerance(),
                pressure=sim.Tolerance(),
                turbulent_kinetic_energy=sim.Tolerance(),
                omega_dissipation_rate=sim.Tolerance(),
            ),
            solvers=sim.FluidSolvers(),
            schemes=sim.Schemes(
                time_differentiation=sim.TimeDifferentiationSchemes(),
                gradient=sim.GradientSchemes(),
                divergence=sim.DivergenceSchemes(),
                laplacian=sim.LaplacianSchemes(),
                interpolation=sim.InterpolationSchemes(),
                surface_normal_gradient=sim.SurfaceNormalGradientSchemes(),
            ),
        ),
        boundary_conditions=[
            ...
        ],
        simulation_control=sim.FluidSimulationControl(
            end_time=sim.DimensionalTime(value=100, unit="s"),
            delta_t=sim.DimensionalTime(value=1, unit="s"),
            write_control=sim.TimeStepWriteControl(write_interval=20),
            max_run_time=sim.DimensionalTime(value=10000, unit="s"),
            decompose_altorithm=sim.ScotchDecomposeAlgorithm(),
        )
        result_control=sim.FluidResultControls()
    )


In this example we used mostly the default values for each object parameter. Of course,
each one of these objects has its own parameters to finely tune the behavior of the 
simulation. If you need to look at the detail of each object and its parameters, please 
check the SDK documentation:

`Python SDK Documentation <https://simscalegmbh.github.io/simscale-python-sdk/simscale_sdk.api.html>`_

We will focus, as an example, on the most basic setup for a fluid simulation: the material 
model and the boundary conditions.

Material
~~~~~~~~

Being a single fluid phase simulation, there is only one material that we need to setup.
Following we will enter the properties for water at ambient temperature, and assign it
to the only volumetric region in the geometry:

.. code-block:: python

    materials=sim.IncompressibleFluidMaterials(
        fluids=[
            sim.IncompressibleMaterial(
                name="Water",
                type="INCOMPRESSIBLE",
                viscosity_model=sim.NewtonianViscosityModel(
                    type="NEWTONIAN",
                    kinematic_viscosity=sim.DimensionalKinematicViscosity(
                        value=9.3379E-7,
                        unit="m²/s",
                    ),
                ),
                density=sim.DimensionalDensity(
                    value=997.33,
                    unit="kg/m³",
                ),
                topological_reference=sim.TopologicalReference(
                    entities=[
                        "B1_TE39",
                    ],
                    sets=[],
                ),
            ),
        ],
    ),


Boundary conditions
~~~~~~~~~~~~~~~~~~~

In this simulation we need three boundary conditions:

1. Velocity inlet 1, at 1.5 m/s
2. Velocity inlet 2, at 1 m/s
3. Pressure outlet, at 0 Pa

The boundary conditions are setup with the following code:

.. code-block:: python

    boundary_conditions=[
        sim.VelocityInletBC(
            name="Velocity inlet 1",
            velocity=sim.FixedValueVBC(
                value=sim.DimensionalVectorFunctionSpeed(
                    value=sim.ComponentVectorFunction(
                        x=sim.ConstantFunction(
                            value=0,
                        ),
                        y=sim.ConstantFunction(
                            value=0,
                        ),
                        z=sim.ConstantFunction(
                            value=-1.5,
                        ),
                    ),
                    unit="m/s",
                ),
            ),
            topological_reference=sim.TopologicalReference(
                entities=[
                    "B1_TE3",
                ],
            ),
        ),
        VelocityInletBC(
            name="Velocity inlet 2",
            velocity=sim.FixedValueVBC(
                value=sim.DimensionalVectorFunctionSpeed(
                    value=sim.ComponentVectorFunction(
                        x=sim.ConstantFunction(
                            value=0,
                        ),
                        y=sim.ConstantFunction(
                            value=-1,
                        ),
                        z=sim.ConstantFunction(
                            value=0,
                        ),
                    ),
                    unit="m/s",
                ),
            ),
            topological_reference=sim.TopologicalReference(
                entities=[
                    "B1_TE30",
                ],
            ),
        ),
        PressureOutletBC(
            name="Pressure outlet",
            gauge_pressure=sim.FixedValuePBC(
                value=sim.DimensionalFunctionPressure(
                    value=sim.ConstantFunction(
                        value=0,
                    ),
                    unit="Pa",
                ),
            ),
            topological_reference=sim.TopologicalReference(
                entities=[
                    "B1_TE37",
                ],
            ),
        ),
    ],


Generating SDK Code
-------------------

It might be a little difficult to navigate the documentation and reference pages to create a 
simulation spec from scratch. Some of the reasons would be:

* How to find out the internal entity name for my part or face?
* How to know for sure the appropriate objects for the parameters?
* How to know the units?

An alternative route to get there is to setup the simulation in the Workbench,
then use the automatic code generator provided by the SDK. For this you need
the project id and the simulation id. You can obtain the project id from the 
workbench URL, looking for the ``pid=`` parameter. Then you can query the project
for the available simulations. For example:


.. code-block:: python

    print(simulations_api.get_simulations(project_id))


Will print something like the following:


.. code-block:: python

    {'embedded': [{'name': 'Incompressible',
                'simulation_id': '326cf56d-d72c-4672-8686-45f46e229792'}],
    'links': {'_self': {'href': '/projects/1562209390575198452/simulations?page=1&limit=100'},
            'first': {'href': '/projects/1562209390575198452/simulations?page=1&limit=100'},
            'last': {'href': '/projects/1562209390575198452/simulations?page=1&limit=100'},
            'next': {'href': '/projects/1562209390575198452/simulations?page=1&limit=100'},
            'prev': {'href': '/projects/1562209390575198452/simulations?page=1&limit=100'}},
    'meta': {'total': 1}}


We can identify that we want the simulation named 'Incompressible', and copy its ``simulation_id``.

Now, to generate the SDK code for this simulation model, we can do as follows:


.. code-block:: python

    sdk_code = simulations_api.get_simulation_sdk_code(project_id, simulation_id)

    with open("sim_code.py", "w") as f:
        f.write(str(sdk_code))


Then all of the code to define the simulation model object will be found in the ``sim_code.py`` file.

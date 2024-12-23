Code Summary
============

For reference, following is a copy of the full code of the tutorial 
in a single go:


.. code-block:: python

    import time

    import simscale_sdk as sim

    API_KEY = "your_api_key_here"
    API_HOST_URL = "https://api.simscale.com/v0"

    configuration = sim.Configuration()
    configuration.host = API_HOST_URL
    configuration.api_key = {
        "X-API-KEY": API_KEY
    }

    api_client = sim.ApiClient(configuration)

    projects_api = sim.ProjectsApi(api_client)

    project_data = sim.Project(
        name="My first simulation project",
        description="Created via SDK",
        measurement_system="SI"
    )

    project = projects_api.create_project(project_data)

    project_id = project.project_id

    print(f"{project_id=}")

    storage_api = sim.StorageApi(api_client)

    storage = storage_api.create_storage()

    with open("pipe_junction_model_tutorial.x_t", "rb") as file:

        api_client.rest_client.PUT(
            url=storage.url,
            headers={
                "Content-Type": "application/octet-stream"
            },
            body=file.read()
        )

    storage_id = storage.storage_id

    print(f"{storage_id=}")

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

    while geometry_import.status not in ("FINISHED", "CANCELED", "FAILED"):

        geometry_import = geometry_import_api.get_geometry_import(project_id, geometry_import_id)

        time.sleep(10)

    if geometry_import.status != "FINISHED":
        raise Exception("Geometry import operation was canceled or failed.")

    geometry_id = geometry_import.geometry_id

    print(f"{geometry_id=}")

    tutorial_model = sim.Incompressible(
        model=sim.FluidModel(),
        initial_conditions=sim.FluidInitialConditions(),
        advanced_concepts=sim.AdvancedConcepts(),
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
        numerics=sim.FluidNumerics(
            relaxation_factor=sim.RelaxationFactor(),
            pressure_reference_value=sim.DimensionalPressure(value=0, unit="Pa"),
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
            sim.VelocityInletBC(
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
            sim.PressureOutletBC(
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
        simulation_control=sim.FluidSimulationControl(
            end_time=sim.DimensionalTime(value=100, unit="s"),
            delta_t=sim.DimensionalTime(value=1, unit="s"),
            write_control=sim.TimeStepWriteControl(write_interval=20),
            max_run_time=sim.DimensionalTime(value=10000, unit="s"),
            decompose_algorithm=sim.ScotchDecomposeAlgorithm(),
        ),
        result_control=sim.FluidResultControls(),
    )

    simulations_api = sim.SimulationsApi(api_client)

    simulation_spec = sim.SimulationSpec(
        name="Incompressible",
        geometry_id=geometry_id,
        model=tutorial_model
    )

    simulation = simulations_api.create_simulation(project_id, simulation_spec)

    mesh_model = sim.SimmetrixMeshingFluid(
        sizing=sim.AutomaticMeshSizingSimmetrix(
            fineness=5,
            curvature=sim.AutomaticCurvature(),
        ),
        automatic_layer_settings=sim.AutomaticLayerOn(
            layer_type=sim.FractionalHeight2(
                number_of_layers=3,
                total_relative_thickness=0.4,
                growth_rate=1.5
            ),
        ),
        physics_based_meshing=True,
        hex_core=True,
    )

    mesh_operation_api = sim.MeshOperationsApi(api_client)

    mesh_operation = mesh_operation_api.create_mesh_operation(
        project_id,
        sim.MeshOperation(
            name="Pipe junction mesh",
            geometry_id=geometry_id,
            model=mesh_model,
        )
    )

    mesh_operation_api.start_mesh_operation(
        project_id,
        mesh_operation.mesh_operation_id,
        simulation_id=simulation.simulation_id,
    )

    while mesh_operation.status not in ("FINISHED", "CANCELED", "FAILED"):

        mesh_operation = mesh_operation_api.get_mesh_operation(
            project_id,
            mesh_operation.mesh_operation_id,
        )

        time.sleep(30)

    print(f"Mesh with id={mesh_operation.mesh_id} was completed with status {mesh_operation.status}")

    simulation_spec = simulations_api.get_simulation(project_id, simulation.simulation_id)

    simulation_spec.mesh_id = mesh_operation.mesh_id

    simulations_api.update_simulation(project_id, simulation.simulation_id, simulation_spec)

    simulation_run_api = sim.SimulationRunsApi(api_client)

    simulation_run = simulation_run_api.create_simulation_run(
        project_id,
        simulation.simulation_id,
        sim.SimulationRun(
            name="Run 1"
        ),
    )

    simulation_run_api.start_simulation_run(
        project_id,
        simulation.simulation_id,
        simulation_run.run_id,
    )

    while simulation_run.status not in ("FINISHED", "CANCELED", "FAILED"):

        simualtion_run = simulation_run_api.get_simulation_run(
            project_id,
            simulation.simulation_id,
            simulation_run.run_id,
        )

        print(f"Status = {simulation_run.status}")

        time.sleep(30)

    print(f"Simulation run with id={simulation_run.run_id} finished with status {simulation_run.status}")

    solution_fields_result = simulation_run_api.get_simulation_run_results(
        project_id,
        simulation.simulation_id,
        simulation_run.run_id,
        category="SOLUTION"
    )

    print(f"{solution_fields_result=}")

    solution_info = solution_fields_result.embedded[0]

    scalar_field = sim.ScalarField(
        field_name="Velocity",
        component="Magnitude",
        data_type="CELL"
    )

    model_settings = sim.ModelSettings(
        parts=[],
        scalar_field=scalar_field,
    )

    cutting_plane = sim.CuttingPlane(
        name="velocity-plane",
        scalar_field=scalar_field,
        center=sim.Vector3D(x=0, y=0, z=0),
        normal=sim.Vector3D(x=1, y=0, z=0),
        opacity=1,
        clipping=True,
        render_mode=sim.RenderMode.SURFACES,
    )

    filters = sim.Filters(
        cutting_planes=[cutting_plane],
    )

    camera_settings = sim.TopViewPredefinedCameraSettings(
        projection_type=sim.ProjectionType.ORTHOGONAL,
        direction_specifier="X_POSITIVE",
    )

    output_settings = sim.TimeStepAnimationOutputSettings(
        type="TIME_STEP",
        name="Output 1",
        format="MP4",
        resolution=sim.ResolutionInfo(x=1440, y=1080),
        from_frame_index=0,
        to_frame_index=5,
        skip_frames=0,
        show_legend=True,
        show_cube=False,
    )

    report_properties = sim.AnimationReportProperties(
        model_settings=model_settings,
        filters=filters,
        camera_settings=camera_settings,
        output_settings=output_settings,
    )

    report_req = sim.ReportRequest(
        name="Report 1",
        description="Simulation report",
        result_ids=[solution_info.result_id],
        report_properties=report_properties,
    )

    reports_api = sim.ReportsApi(api_client)

    create_report_res = reports_api.create_report(
        project_id,
        report_req,
    )

    report_id = create_report_res.report_id

    report_job = reports_api.start_report_job(
        project_id,
        report_id,
    )

    report = reports_api.get_report(
        project_id,
        report_id,
    )

    while report.status not in ("FINISHED", "CANCELED", "FAILED"):

        time.sleep(30)

        report = reports_api.get_report(
            project_id,
            report_id,
        )

    print(f"Report creation finished with status {report.status}")

    report_res = api_client.rest_client.GET(
        url=report.download.url,
        headers={"X-API-KEY": API_KEY},
    )

    file_name = f"report.{report.download.format}"

    with open(file_name, "wb") as file:
        file.write(report_res.data)


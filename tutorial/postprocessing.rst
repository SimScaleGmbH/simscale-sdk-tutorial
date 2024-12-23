Postprocessing
==============

Having the numerical results computed by the simulation run,
the next step is to generate some plots to interpretate the results.
This task is generally referred to as 'post-processing'.

In the SimScale platform, the results are stored in the cloud, and the SDK
provides the methods to query and download the results. In this example we
will generate a report from the results, including a cutting-plane view 
of the 3D fields.

To begin, let's obtain the meta-data from the solution fields:


.. code-block:: python

    solution_fields_result = simulation_run_api.get_simulation_run_results(
        project_id,
        simulation_id,
        simulation_run_id,
        category="SOLUTION"
    )

    print(f"{solution_fields_result=}")

If we examine this object, we find that it contains the property ``embedded``
with a list of results. We will use the first value in the list, and download
the actual results data for later use:


.. code-block:: python

    solution_info = solution_fields_result.embedded[0]


Now we can use the data in ``solution_info`` to perform our tasks.

Create an Animation
-------------------

We will create a simulation report using the SDK, which will contain an
animation of the velocity field results. In order to do so, we need to first
create a series of configuration objects:


.. code-block:: python

    scalar_field = sim.ScalarField(
        field_name="Velocity",
        component="Magnitude",
        data_type="CELL"
    )

    model_settings = sim.ModelSettings(
        parts=[],
        sacalar_field=scalar_field,
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


You can see how each object controls one part of the animation:

* ``ScalarField`` selects which result field is to be displayed
* ``CuttingPlane`` specifies a cutting plane to look inside the volume
* ``CameraSettings`` specify the view direction
* ``TimeStepAnimationOutputSettings`` control the animation
* ``AnimationReportProperties`` links all of the other configurations

Then we can request the creation of the animation and launch the job, which 
happens in the platform:


.. code-block:: python

    report_req = sim.ReportRequest(
        name="Report 1",
        description="Simulation report",
        result_ids=[solution_info.result_id],
        report_properties=report_properties,
    )

    reports_api = sim.ReportsApi(api_client)

    create_report_res = reports_api.create_report(
        project_id,
        report_request,
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


Now that the report is successfuly created in the platform, we can 
download and store it locally:


.. code-block:: python

    report_res = api_client.rest_client.GET(
        url=report.download.url,
        headers={"X-API-KEY": API_KEY},
    )

    file_name = f"report.{report.download.format}"

    with open(file_name, "wb") as file:
        file.write(report_res.data)


You should be able to open the video file with your preferred player.


Download 3D Fields
------------------

Another option to use the result fields is to get the full numerical fields,
and process them locally. As the results are in VTK format, the pocessing 
can be performed with a library such as `PyVista <https://pyvista.org/>`_ 
or with a GUI application like `Paraview <https://www.paraview.org/>`_.

In order to download the results we can do as follows:


.. code-block:: python

    solution_res = api_client.rest_client.GET(
        url=solution_info.download.url,
        headers={"X-API-KEY": API_KEY},
    )

    with open("solution.zip", "wb") as file:
        file.write(solution_res.data)


The results are stored in a zip archive, and can be extracted manually with
the file explorer or programatically with the ``zip`` library.

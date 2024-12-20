Simulation Run
==============

With a completed simulation spec and a computed mesh, we are ready to 
launch the solution computation. This task is known as a simualtion run
in the Simscale platform.

The ``SimulationRun`` object alongside the ``SimulationRunsApi`` are 
used to launch the task and check its progress, just like a geometry 
import or mesh computation operations:


.. code-block:: python

    import simscale_sdk as sim

    simulation_run_api = sim.SimulationRunsApi(api_client)

    simulation_run = simulation_run_api.create_simulation_run(
        project_id,
        simulation_id,
        sim.SimulationRun(
            name="Run 1"
        ),
    )

    simulation_run_api.start_simulation_run(
        project_id,
        simulation_id,
        simulation_run.run_id,
    )

    while simulation_run.status not in ("FINSIHED", "CANCELED", "FAILED"):

        simualtion_run = simulation_run_api.get_simulation_run(
            project_id,
            simulation_id,
            simulation_run.run_id,
        )

        time.sleep(30)

    print(f"Simulation run with id={simulation_run.run_id} finished with status {simulation_run.status}")


After this point, if the simualtion was successful, we can proceed to the
postprocessing of the results.

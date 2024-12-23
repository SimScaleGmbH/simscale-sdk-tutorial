.. SimScale SDK Tutorial documentation master file, created by
   sphinx-quickstart on Mon Dec 16 10:27:42 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

SimScale SDK Tutorial
=====================

*Welcome to the tutorial for the SimScale SDK!*

The goal of this tutorial is to provide a first-touch introduction to the SimScale SDK,
with a hands-on approach and practical examples.

The SimScale SDK offers API access to the SimScale simulation platform, with 
`Python <https://github.com/SimScaleGmbH/simscale-python-sdk>`_ and 
`C# <https://github.com/SimScaleGmbH/simscale-csharp-sdk>`_ libraries available. 

With the SDK library, the user can automate tasks to create, run and postprocess physics 
simulations with FEA, CFD and LBM methods.

For more information about the SimScale platform, please visit the 
`official web page <https://www.simscale.com/>`_

Target Audience
---------------

This tutorial is aimed towards users of the SimScale platform, who already have a notion
of the simulation workflow, and want to lear how to automate their simualtions with the API. 
If you are a new user to SimScale, please first get a grasp of the workflow
and capabilites of the platform, for instance by following the introductory and advaced tutorials

`Introductory Platform Tutorials <https://www.simscale.com/tutorials>`_ 

`Advanced Platform Tutorials <https://www.simscale.com/docs/tutorials>`_

In fact, the SDK driven simulation that will be developed in this guide corresponds with the
"Fluid Flow Simulation" step-by-step tutorial from the introductory tutorials.

Also, this tutorial assumes that the user is already experienced in the python or C#
programming languages and knowledgeable in basic to intermediate programming concepts. Most 
of the presented code is in the form of snippets and will not run on its own. For instance, 
variables created in previous pages might be used without additional notice or context.

Although the individual pages can serve as reference, following the tutorial step by step 
might be required to have a full understanding of the API workflow.



Summary
-------

.. toctree::
   :maxdepth: 2

   tutorial/installation
   tutorial/api_keys
   tutorial/create_project
   tutorial/geometry_upload_and_import
   tutorial/simulation_setup
   tutorial/meshing
   tutorial/simulation_run
   tutorial/postprocessing

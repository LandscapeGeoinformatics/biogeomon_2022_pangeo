Test that everything works
--------------------------

If you don't have the Anaconda Prompt open, please open it. ``Anaconda Prompt`` can be found by clicking on the Windows start menu button and start typing ``Anaconda Prompt``.
Also always make sure you have activated your ``biogeomon2022`` environment.

.. code::

    (C:\dev\conda3)  activate biogeomon2022

    (biogeomon2022)

You can test that the installations have worked by running following commands in a Python console.
At first start the Python console:

.. code::

    (biogeomon2022) python

    Type "help", "copyright", "credits" or "license" for more information.
    >>>

.. code:: python


    # Processing
    import pandas as pd
    import numpy as np
    import xarray as xr
    import rioxarray # for CRS management in Xarray

    # Data loading
    import intake
    import intake_xarray # Xarray wrapper for Intake
    import h5netcdf # for NetCDF reading

    # Plotting
    import matplotlib
    import matplotlib.pyplot as plt

    # Trend analysis
    import pymannkendall as mk


If you don't receive any errors, everything should be working!

In order to close the Python interpreter type ``exit()`` or press **Ctrl+Z** plus Return to exit.

.. code::

    (biogeomon2022) >>> exit()


Setting up Jupyter Notebook
---------------------------

The Jupyter Notebook server is an open-source web application that allows you to create and share documents that contain live code,
equations, visualizations and narrative text. Uses include: data cleaning and transformation, numerical simulation, statistical modeling,
data visualization, machine learning, and much more. Jupyter Lab is an extended version that allows to have several notebooks open at the same time. We will be mostly working with Jupyter Lab.

Before we start Python coding we will make our newly created conda Python environment known to the Jupyter notebook system by installing the kernel, basically the execution engine link from Jupyter web notebook to our Python environment:

Make sure you are in your dedicated working directory, your "biogeomon2022" folder, on the Command line window (Anaconda prompt). And make sure the ``biogeomon2022`` conda environment is activated:

.. code::

    (C:\dev\conda3)  activate biogeomon2022

    (biogeomon2022) python -m ipykernel install --user --name biogeomon2022

That should be it. You should now be able to start the Jupyter notebook server:

.. code::

    (biogeomon2022) jupyter lab

This should open a webpage in your default webbrowser.

.. Note::

    Don't close the console window nor the browser window where the notebooks opened. Always use the "File" -> "Save and Checkpoint" and then -> "Close and Halt" menu options in the notebooks, and the "Quit" button on the main notebook server webpage.

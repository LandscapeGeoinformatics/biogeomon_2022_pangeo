Alternative installation method for Conda environments
------------------------------------------------------

In the previous section we installed our specific conda Python environment with a pre-defined environment configuration file. In this section we show an alternative variant how you can install conda environments in a more flexible way if you need to.


.. admonition:: BEWARE:

    Please DO NOT follow these steps, if you already have the environment installed via the ``environment.yml`` configuration above.

The other variant is typically more widely used in exploratory setups. It is a step-by-step procedure. Here we are making sure that Python in version 3.8 will be installed and that we want to explicitly use the additional package channel `conda-forge`.
And we give the environment a name (-n).

If you would create your environment manually, it would go like that:
Open the ``Anaconda prompt`` from the Start Menu and type the command below.

.. code::

    (C:\dev\conda3) conda create -n biogeomon2022alt python=3.9 -c conda-forge

Ok, now that we have installed a Python working environment with the name ``biogeomon2022alt`` with our desired library packages, we can check installed environments just to be sure.
In order to show all environments that have already been created you can ask conda to list these:

.. code::

    (C:\dev\conda3)  conda env list

Now we want to activate that environment, install additional packages and start working with it:

.. code::

    (C:\dev\conda3)  activate biogeomon2022alt

    (biogeomon2022alt)


Install GIS related packages with conda by running in command prompt following commands (in the same order as they are listed).
Make sure you are in the correct environment (don't install into ``base``, install new packages ideally only into your designated created environments)

.. code::

    (biogeomon2022alt) conda install -c conda-forge pandas numpy xarray dask hdf4 netcdf4 hdf5 h5netcdf

    # Install matplotlib and Jupyter Lab/Notebook
    (biogeomon2022alt) conda install -c conda-forge matplotlib jupyter notebook jupyterlab ipywidgets

    # Install intake and some dependencies
    (biogeomon2022alt) conda install -c conda-forge aiohttp intake intake-xarray

    # Install rasterio and some raster libraries link for xarray
    (biogeomon2022alt) conda install -c conda-forge rasterio rioxarry

    # Install pymannkendall
    (biogeomon2022alt) conda install -c conda-forge pymannkendall



In the next step we will verify the installation of our conda Python environment and configure Jupyter Notebooks.

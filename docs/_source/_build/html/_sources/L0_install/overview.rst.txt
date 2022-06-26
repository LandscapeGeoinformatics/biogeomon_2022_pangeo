The Pangeo Python Ecosystem
---------------------------

PANGEO - A community platform for Big Data geoscience

.. figure:: ../_static/pangeo_icon.png
   :height: 100

   https://pangeo.io/

Pangeo is first and foremost a community promoting open, reproducible, and scalable science. This community provides documentation, develops and maintains software, and deploys computing infrastructure to make scientific research and programming easier. The Pangeo software ecosystem involves open source tools such as xarray, iris, dask, jupyter, and many other packages. There is no single software package called “pangeo”; rather, the Pangeo project serves as a coordination point between scientists, software, and computing infrastructure.

1. Foster collaboration around the open source scientific python ecosystem for ocean / atmosphere / land / climate science.

2. Support the development with domain-specific geoscience packages.

3. Improve scalability of these tools to handle petabyte-scale datasets on HPC and cloud platforms.


Jupyter Notebook and Jupyter Lab
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. figure:: https://jupyter.org/assets/homepage/main-logo.svg
   :height: 100

   https://jupyter.org/

JupyterLab is the latest web-based interactive development environment for notebooks, code, and data. Its flexible interface allows users to configure and arrange workflows in data science, scientific computing, computational journalism, and machine learning.



xarray: N-D labeled arrays and datasets in Python
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. figure:: https://docs.xarray.dev/en/stable/_static/dataset-diagram-logo.png
   :height: 100

   https://docs.xarray.dev/en/stable/

xarray (formerly xray) is an open source project and Python package that makes working with labelled multi-dimensional arrays simple, efficient, and fun!

Xarray introduces labels in the form of dimensions, coordinates and attributes on top of raw NumPy-like arrays, which allows for a more intuitive, more concise, and less error-prone developer experience. The package includes a large and growing library of domain-agnostic functions for advanced analytics and visualization with these data structures.

Xarray is inspired by and borrows heavily from pandas, the popular data analysis package focused on labelled tabular data. It is particularly tailored to working with netCDF files, which were the source of xarray’s data model, and integrates tightly with dask for parallel computing.



Intake: a Python library for accessing data in a simple and uniform way
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. figure:: https://intake.readthedocs.io/en/latest/_static/images/logo.png
   :height: 100

   https://intake.readthedocs.io/en/latest/

Intake is a lightweight package for finding, investigating, loading and disseminating data. It will appeal to different groups for some of the reasons below, but is useful for all and acts as a common platform that everyone can use to smooth the progression of data from developers and providers to users.



Next steps
~~~~~~~~~~


After this lesson you should:

- setup a Python environment and install required packages with the conda tool
- know what a Jupyter notebook is and how to start it in the right place

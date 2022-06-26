Installation and setup for Python with Miniconda
------------------------------------------------

.. note::

    In the University computer lab you **DO NOT** have to install Anaconda or Miniconda or any other Python distribution.
    If you are attending the course in the University computer lab, please jump to `Creating environments and install packages <Installing_Miniconda_GIS.html#id1>`_ to create the working environment.
    This section only for general info if you want to install Python on your own computer, and for remote students.

**How to start doing GIS with Python on a computer?**

Well, first you need to install Python and necessary Python modules that are used to perform various GIS-tasks. The purpose of this page is to help you
out installing Python and all those modules into your own computer. Even though it is possible to install Python from their `homepage <https://www.python.org/>`_,
**we highly recommend using** `Miniconda <https://docs.conda.io/en/latest/miniconda.html>`_ or `Anaconda <https://www.anaconda.com/distribution/>`_ which is an open source distribution of the Python and R programming
languages for large-scale data processing, predictive analytics, and scientific computing, that aims to simplify package management and deployment. In short,
it makes life much easier when installing new tools on your Python to play with.

.. note::

    **Miniconda** is an encapsulated versatile virtual python environment installer,
    that works under the hood of the big Anaconda python distribution.
    Miniconda is basically a mini version of Anaconda that includes only the conda package manager and its dependencies!


https://conda.io/miniconda.html

Following steps have been tested to work on Windows 7 and 10 with Anaconda/Miniconda 64 bit.

`Download Miniconda installer (64 bit) <https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x86_64.exe>`_ a Python 3.8/3.9, 64-bit (exe installer) for Windows.

.. admonition:: BEWARE:

    * To install miniconda SYSTEM-WIDE for ALL users, this does require administrator permissions;
      every users can then create their own environments with the conda tool.
    * Please do NOT make Conda the default python for the system if you don't want it to interfere with other Python installations you might have,
      eg. Pythons of ArcGIS and QGis etc

Install Miniconda on your computer by double clicking the installer and install it into a directory you want.

Install it to **all users** and use default settings.

Additional install information:
https://conda.io/projects/conda/en/latest/user-guide/index.html

Verifying the installation
--------------------------

.. note::

    As a convention, whenever I demonstrate Python codes or using commands on the the shell/cmd commandline,
    using the ``#`` symbol implies a ``comment``. This line, respectively everything after that symbol is NOT to be executed.


In order to test that the ``conda`` package manager works we have to go through a few more steps:
After successful installation you should have a menu entry in the Windows Start Menu:

``Anaconda Prompt``

This is a Windows CMD (Commandline window, that "knows" about, where your Miniconda/Anaconda installation lies, and where to find the ``conda`` tool (without interfering other Python installations on your computer).
After it opens it should display somehow like so:

.. code::

    (base) C:\Users\Your_Account>

    or

    (C:\dev\conda3) C:\Users\Your_Account>

On the command line type command ``conda --version`` in order to see if the command is successful, it should show the version of the conda tool.

.. code::

    (base) C:\Users\Your_Account> conda --version
    conda 4.9.2


Creating environments and install packages
------------------------------------------

It is important to understand, that you are always "residing" somewhere in some folder. This will in particular important to make sure that you find your scripts, notebooks and the working data, which ideally reside under the same directory hierarchy.

In order to be well organised throughout the course, please create a working folder/directory, where you will put your scripts and data files for the lab sessions and homeworks.
On Windows you have your local user folder, typically ``C:\Users\Your_Account``. Open with the Windows File Explorer and create a new folder in this folder, with the name ``biogeomon2022``.

.. admonition:: BEWARE:

    Make sure you navigate explicitly into your correct working folder "biogeomon2022". As we will always start or environment from the command line, here are two very simple practical steps to navigate on the commandline.

Open the Anaconda/conda command prompt from the Start menu, and type the commands shown below.

.. code::

    c:
    cd C:\Users\Your_Account\biogeomon2022

You can see which files are inside this folder by using the ``dir`` command. (On Mac and Linux it is ``ls``) and it will print information and files of your current folder.

.. code::

    c:
    cd C:\Users\Your_Account\biogeomon2022

    dir

    ... output below

    Volume in drive C is Windows
    Volume Serial Number is 5E4C-FED5

    Directory of C:\Users\Your_Account\biogeomon2022

    29.09.2021  15:00    <DIR>          .
    29.09.2021  15:00    <DIR>          ..


``conda`` basically represents a typical Python virtualenv command. You can create a several distinct environments, with different Python version, and with different packages to be installed.
This will come in very handy to *try out* new libraries/packages/tools, without breaking you working installation. How to use the conda command?

https://docs.conda.io/projects/conda/en/latest/user-guide/cheatsheet.html

We want to use a modern Python version 3.8, but some packages might not be fully available or tested in Python 3.9. Here it becomes obvious how practical virtual environments can be.
They help you to keep various Python versions around without messing up your system, and at the same time, keep several working environments with different, possibly conflicting versions of different Python packages.

There are two main ways of creating a ``conda`` environment.

The more professional/reproducible way is to use a so called ``environment.yml`` file. In the file the environment name, the package channels, Python version and all desired packages are declared.
This way, you should be able to install the same environment in different computers, therfore improving the reliability of having the same packages etc installed. You can even go so far as to specify the exact package version.

Make sure you are using the correct folder on the University computers. In the Windows File Explorer, go via ``this PC``, ``c:\Users`` to ``your account name`` folder.

.. admonition:: NB:

   Now, please download the prepared `environment.yml <https://raw.githubusercontent.com/LandscapeGeoinformatics/biogeomon_2022_pangeo/main/environment.yml>`_ file and save it to your newly created working folder.


To check, in the commandline window (Anaconda prompt) navigate to your working folder again and list the contents of the folder with ``dir``:

.. code::

    (C:\dev\conda3) c:

    (C:\dev\conda3) cd C:\Users\Your_Account\biogeomon2022

    (C:\dev\conda3) dir

    ... output below

    Volume in drive C is Windows
    Volume Serial Number is 5E4C-FED5

    Directory of C:\Users\Your_Account\biogeomon2022

    29.10.2021  15:00    <DIR>          .
    29.10.2021  15:00    <DIR>          ..
    29.10.2021  08:51 AM            693 environment.yml


You should see the newly downloaded file in your folder.
Now let's install the enviroment with conda:

.. code::

    (C:\dev\conda3) conda env create -f environment.yml


This will take some time.

In the `next steps we will verify the installation and test that everything works <test_installation.html>`_ in our new ``biogeomon2022`` conda Python environment and configure Jupyter Notebooks.

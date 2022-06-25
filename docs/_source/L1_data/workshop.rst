Spatio-temporal trend analysis of temperature using Python
----------------------------------------------------------

**Tutors:** Alexander Kmoch1, Holger Virro1, and Evelyn Uuemaa1

1\ `Landscape Geoinformatics
Lab <https://landscape-geoinformatics.ut.ee/>`__, Chair of
Geoinformatics, `Department of
Geography <https://www.geograafia.ut.ee/en>`__, University of Tartu


Introduction
~~~~~~~~~~~~

Under the climate change threat, we need to quantify the spatio-temporal
trend of temperature and rainfall patterns to understand and evaluate
the potential impacts of climate change on ecosystem services, energy
fluxes and biogeochemical processes. There are wide range of global or
regional scale climate data freely available in a gridded format that
can be used for spatio-temporal analysis. These data can be derived
directly from satellite remote sensing data or based on reanalysis of
weather station time series data. Additionally, some gridded data are
also developed by combining time series data from weather stations with
remote sensing data.

Aim of the workshop
~~~~~~~~~~~~~~~~~~~

The goal of this session is to teach the participants how to handle
**NetCDF** datasets, apply the `Mann-Kendall
(MK) <https://www.statisticshowto.com/wp-content/uploads/2016/08/Mann-Kendall-Analysis-1.pdf>`__
test and calculate **Sen’s slope (SS)** values on a gridded climate
dataset (please, check additional info below).

We will use the gridded daily mean temperature product from **European
Climate Assessment and Dataset** (E-OBS version v23.1e) (Cornes et al.,
2018).
`E-OBS <https://surfobs.climate.copernicus.eu/dataaccess/access_eobs.php>`__
is developed by interpolating climate data from weather stations
provided by the national meteorological agencies of the European
countries. The product is available at a grid resolution of 0.1° or
0.25° and covers the period since 1950 to present. For the workshop, we
will use the **0.1° resolution** dataset (which represents a resolution
of about 11 km at the equator) from **January 2001 until December 2020** already
aggregated into **monthly mean temperature (°C)**. The figure below
illustrates how we will calculate MK and SS at pixel scale on monthly
basis.



Mann-Kendall test for trend analysis
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Different methodologies have combined spatial data sets and
nonparametric statistical methods, such as the **Mann-Kendall (MK)**
(Mann, 1945; Kendall, 1957) test, to infer about the temporal trends of
climatic variables time series (Atta-ur-Rahman and Dawood, 2017; Jaagus
et al., 2017; Silva Junior et al., 2018). As MK is a nonparametric test,
it does not require normal distribution of the data, but it does assume
no autocorrelation in the time series.

The MK test has as null hypothesis that the time series has no trend,
and the alternative hypothesis is that it does have a trend (upward or
downward). The test first calculates the differences between all pairs
of earlier and later data points in the time series. After that, it
calculates the variance of these differences, which is posteriorly used
to calculate the **Z test** value. Finally, the statistical significance
of the trend is quantified by calculating the probability associated
with the normalized Z test. If :math:`Z>0`, it indicates an increasing
trend and if :math:`Z<0` it indicates a decreasing trend. Usually, the
trends are defined as significant using 95% confidence level.

In addition to the trend calculation, it is also possible to quantify
the magnitude of the trends. The magnitudes can be estimated by using
the nonparametric Sen statistic, more specifically, the **Theil–Sen
estimator** (Sen Pranab Kumar, 1968), which is given by the median of
the slopes of each pair of points. To calculate the Sen’s slope, the
times series data is ordered accordingly to the time (as function of
time) and a confidence interval is provided for each slope value (same
as in the MK test).

NetCDF files
~~~~~~~~~~~~

**NetCDF** is a file format that allows the storage of multidimensional
data into a single file archive and has been widely used. Check more
info about NetCDF and Python by following this `YouTube
tutorial <https://youtu.be/K1_8EqCJlwo>`__ produced by `Copernicus
Marine Service <https://marine.copernicus.eu/>`__.

Reading the NetCDF file with Xarray
-----------------------------------

We will start by importing all Python libraries used in this workshop.
Data loading will be handled by
`Intake <https://intake.readthedocs.io/en/latest/>`__. For data
processing and wrangling we are going to use the
`Xarray <http://xarray.pydata.org/en/stable/>`__,
`Pandas <https://pandas.pydata.org/>`__ and
`NumPy <https://numpy.org/>`__ libraries.
`Matplotlib <https://matplotlib.org/>`__ will be used for plotting
purposes. Trend analysis will be carried out using the
`pyMannKendall <https://pypi.org/project/pymannkendall/>`__ library.

.. code:: ipython3

    # Import libraries

    # Data loading
    import intake
    import intake_xarray # Xarray wrapper for Intake
    import h5netcdf # for NetCDF reading

    # Processing
    import xarray as xr
    import pandas as pd
    import numpy as np
    import rioxarray # for CRS management in Xarray

    # Trend analysis
    import pymannkendall as mk

    # Plotting
    import matplotlib
    import matplotlib.pyplot as plt
    import matplotlib.colors as mcolors
    import collections
    import matplotlib.patches as mpatches
    import calendar

Our temperature NetCDF is located in an
`ownCloud <https://owncloud.com/>`__ folder and the file’s metadata is
given in ``sc58_catalog.yaml`` located in the workshop’s GitHub
repository.

.. code:: ipython3

    print('https://raw.githubusercontent.com/LandscapeGeoinformatics/biogeomon_2022_pangeo/main/resources/workshop_catalog.yaml')


.. parsed-literal::

    https://raw.githubusercontent.com/LandscapeGeoinformatics/biogeomon_2022_pangeo/main/resources/workshop_catalog.yaml


Let us open the catalog file and list its content. We will use the
`Intake <https://intake.readthedocs.io/en/latest/>`__ library for
loading the data. It allows for convenient data import from external
locations.

.. code:: ipython3

    # Open catalog file
    cat = intake.open_catalog('https://raw.githubusercontent.com/LandscapeGeoinformatics/biogeomon_2022_pangeo/main/resources/workshop_catalog.yaml')
    list(cat)




.. parsed-literal::

    ['monthly_precip_baltics', 'monthly_temp_baltics']



We are going to use only the temperature file named
``monthly_temp_baltics.nc`` in this workshop. First, let us see the
metadata of this NetCDF.

.. code:: ipython3

    # Read metadata
    temp_data = cat.monthly_temp_baltics
    temp_data



.. parsed-literal::

    monthly_temp_baltics:
      args:
        urlpath: https://github.com/LandscapeGeoinformatics/biogeomon_2022_pangeo/raw/main/resources/monthly_temp_baltics.nc
      description: Monthly temperature data for the Baltics
      driver: intake_xarray.netcdf.NetCDFSource
      metadata:
        catalog_dir: https://raw.githubusercontent.com/LandscapeGeoinformatics/biogeomon_2022_pangeo/main/resources



The metadata includes the parameters \* ``urlpath`` showing the location
of the NetCDF \* the ``description`` of the data \* ``driver`` to be
used when importing the data \* ``catalog_dir`` showing the catalog file
location

Now we can read the NetCDF with monthly temperature data for the Baltic
states in the period 2001–2020. We are going to read the data as an
Xarray dataset. The Xarray library allows for convenient manipulation of
multidimensional arrays and interaction with the two main data
objects—DataArray and Dataset—borrows elements from the Rasterio, Pandas
and NumPy Python libraries.

Xarray is a part of the `Pangeo <https://pangeo.io/index.html>`__ big
data geoscience environment and is particularly useful for working with
large datasets. As our dataset is not quite that big, we will only use
some of the main Xarray functionalities for our analysis.

As the driver used for reading the file is already specified in the
catalog, we can simply use ``read`` to import the temperature data.

.. code:: ipython3

    # Import temperature data as an Xarray dataset
    temp_ds = temp_data.read()
    temp_ds




.. raw:: html

    <div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
    <defs>
    <symbol id="icon-database" viewBox="0 0 32 32">
    <path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
    <path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
    <path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
    </symbol>
    <symbol id="icon-file-text2" viewBox="0 0 32 32">
    <path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
    <path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    <path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    <path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    </symbol>
    </defs>
    </svg>
    <style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
     *
     */

    :root {
      --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
      --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
      --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
      --xr-border-color: var(--jp-border-color2, #e0e0e0);
      --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
      --xr-background-color: var(--jp-layout-color0, white);
      --xr-background-color-row-even: var(--jp-layout-color1, white);
      --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
    }

    html[theme=dark],
    body.vscode-dark {
      --xr-font-color0: rgba(255, 255, 255, 1);
      --xr-font-color2: rgba(255, 255, 255, 0.54);
      --xr-font-color3: rgba(255, 255, 255, 0.38);
      --xr-border-color: #1F1F1F;
      --xr-disabled-color: #515151;
      --xr-background-color: #111111;
      --xr-background-color-row-even: #111111;
      --xr-background-color-row-odd: #313131;
    }

    .xr-wrap {
      display: block !important;
      min-width: 300px;
      max-width: 700px;
    }

    .xr-text-repr-fallback {
      /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
      display: none;
    }

    .xr-header {
      padding-top: 6px;
      padding-bottom: 6px;
      margin-bottom: 4px;
      border-bottom: solid 1px var(--xr-border-color);
    }

    .xr-header > div,
    .xr-header > ul {
      display: inline;
      margin-top: 0;
      margin-bottom: 0;
    }

    .xr-obj-type,
    .xr-array-name {
      margin-left: 2px;
      margin-right: 10px;
    }

    .xr-obj-type {
      color: var(--xr-font-color2);
    }

    .xr-sections {
      padding-left: 0 !important;
      display: grid;
      grid-template-columns: 150px auto auto 1fr 20px 20px;
    }

    .xr-section-item {
      display: contents;
    }

    .xr-section-item input {
      display: none;
    }

    .xr-section-item input + label {
      color: var(--xr-disabled-color);
    }

    .xr-section-item input:enabled + label {
      cursor: pointer;
      color: var(--xr-font-color2);
    }

    .xr-section-item input:enabled + label:hover {
      color: var(--xr-font-color0);
    }

    .xr-section-summary {
      grid-column: 1;
      color: var(--xr-font-color2);
      font-weight: 500;
    }

    .xr-section-summary > span {
      display: inline-block;
      padding-left: 0.5em;
    }

    .xr-section-summary-in:disabled + label {
      color: var(--xr-font-color2);
    }

    .xr-section-summary-in + label:before {
      display: inline-block;
      content: '►';
      font-size: 11px;
      width: 15px;
      text-align: center;
    }

    .xr-section-summary-in:disabled + label:before {
      color: var(--xr-disabled-color);
    }

    .xr-section-summary-in:checked + label:before {
      content: '▼';
    }

    .xr-section-summary-in:checked + label > span {
      display: none;
    }

    .xr-section-summary,
    .xr-section-inline-details {
      padding-top: 4px;
      padding-bottom: 4px;
    }

    .xr-section-inline-details {
      grid-column: 2 / -1;
    }

    .xr-section-details {
      display: none;
      grid-column: 1 / -1;
      margin-bottom: 5px;
    }

    .xr-section-summary-in:checked ~ .xr-section-details {
      display: contents;
    }

    .xr-array-wrap {
      grid-column: 1 / -1;
      display: grid;
      grid-template-columns: 20px auto;
    }

    .xr-array-wrap > label {
      grid-column: 1;
      vertical-align: top;
    }

    .xr-preview {
      color: var(--xr-font-color3);
    }

    .xr-array-preview,
    .xr-array-data {
      padding: 0 5px !important;
      grid-column: 2;
    }

    .xr-array-data,
    .xr-array-in:checked ~ .xr-array-preview {
      display: none;
    }

    .xr-array-in:checked ~ .xr-array-data,
    .xr-array-preview {
      display: inline-block;
    }

    .xr-dim-list {
      display: inline-block !important;
      list-style: none;
      padding: 0 !important;
      margin: 0;
    }

    .xr-dim-list li {
      display: inline-block;
      padding: 0;
      margin: 0;
    }

    .xr-dim-list:before {
      content: '(';
    }

    .xr-dim-list:after {
      content: ')';
    }

    .xr-dim-list li:not(:last-child):after {
      content: ',';
      padding-right: 5px;
    }

    .xr-has-index {
      font-weight: bold;
    }

    .xr-var-list,
    .xr-var-item {
      display: contents;
    }

    .xr-var-item > div,
    .xr-var-item label,
    .xr-var-item > .xr-var-name span {
      background-color: var(--xr-background-color-row-even);
      margin-bottom: 0;
    }

    .xr-var-item > .xr-var-name:hover span {
      padding-right: 5px;
    }

    .xr-var-list > li:nth-child(odd) > div,
    .xr-var-list > li:nth-child(odd) > label,
    .xr-var-list > li:nth-child(odd) > .xr-var-name span {
      background-color: var(--xr-background-color-row-odd);
    }

    .xr-var-name {
      grid-column: 1;
    }

    .xr-var-dims {
      grid-column: 2;
    }

    .xr-var-dtype {
      grid-column: 3;
      text-align: right;
      color: var(--xr-font-color2);
    }

    .xr-var-preview {
      grid-column: 4;
    }

    .xr-var-name,
    .xr-var-dims,
    .xr-var-dtype,
    .xr-preview,
    .xr-attrs dt {
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      padding-right: 10px;
    }

    .xr-var-name:hover,
    .xr-var-dims:hover,
    .xr-var-dtype:hover,
    .xr-attrs dt:hover {
      overflow: visible;
      width: auto;
      z-index: 1;
    }

    .xr-var-attrs,
    .xr-var-data {
      display: none;
      background-color: var(--xr-background-color) !important;
      padding-bottom: 5px !important;
    }

    .xr-var-attrs-in:checked ~ .xr-var-attrs,
    .xr-var-data-in:checked ~ .xr-var-data {
      display: block;
    }

    .xr-var-data > table {
      float: right;
    }

    .xr-var-name span,
    .xr-var-data,
    .xr-attrs {
      padding-left: 25px !important;
    }

    .xr-attrs,
    .xr-var-attrs,
    .xr-var-data {
      grid-column: 1 / -1;
    }

    dl.xr-attrs {
      padding: 0;
      margin: 0;
      display: grid;
      grid-template-columns: 125px auto;
    }

    .xr-attrs dt,
    .xr-attrs dd {
      padding: 0;
      margin: 0;
      float: left;
      padding-right: 10px;
      width: auto;
    }

    .xr-attrs dt {
      font-weight: normal;
      grid-column: 1;
    }

    .xr-attrs dt:hover span {
      display: inline-block;
      background: var(--xr-background-color);
      padding-right: 10px;
    }

    .xr-attrs dd {
      grid-column: 2;
      white-space: pre-wrap;
      word-break: break-all;
    }

    .xr-icon-database,
    .xr-icon-file-text2 {
      display: inline-block;
      vertical-align: middle;
      width: 1em;
      height: 1.5em !important;
      stroke-width: 0;
      stroke: currentColor;
      fill: currentColor;
    }
    </style><pre class='xr-text-repr-fallback'>&lt;xarray.Dataset&gt;
    Dimensions:      (latitude: 57, longitude: 72, ym: 240)
    Coordinates:
      * latitude     (latitude) float64 53.95 54.05 54.15 ... 59.35 59.45 59.55
      * longitude    (longitude) float64 21.05 21.15 21.25 ... 27.95 28.05 28.15
      * ym           (ym) object &#x27;2001-01&#x27; &#x27;2001-02&#x27; ... &#x27;2020-11&#x27; &#x27;2020-12&#x27;
        spatial_ref  int32 0
    Data variables:
        temp_mean    (latitude, longitude, ym) float32 nan nan nan ... nan nan nan
    Attributes:
        grid_mapping:  spatial_ref</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.Dataset</div></div><ul class='xr-sections'><li class='xr-section-item'><input id='section-25d5631e-b05b-41e0-b042-f4d22f32be9f' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-25d5631e-b05b-41e0-b042-f4d22f32be9f' class='xr-section-summary'  title='Expand/collapse section'>Dimensions:</label><div class='xr-section-inline-details'><ul class='xr-dim-list'><li><span class='xr-has-index'>latitude</span>: 57</li><li><span class='xr-has-index'>longitude</span>: 72</li><li><span class='xr-has-index'>ym</span>: 240</li></ul></div><div class='xr-section-details'></div></li><li class='xr-section-item'><input id='section-b6695a26-c2ef-4600-b4d1-219860333249' class='xr-section-summary-in' type='checkbox'  checked><label for='section-b6695a26-c2ef-4600-b4d1-219860333249' class='xr-section-summary' >Coordinates: <span>(4)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>latitude</span></div><div class='xr-var-dims'>(latitude)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>53.95 54.05 54.15 ... 59.45 59.55</div><input id='attrs-779f615b-01af-45ab-9d36-e6cd7b2973d5' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-779f615b-01af-45ab-9d36-e6cd7b2973d5' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-19d5f146-947d-42cf-b752-670fd34c0073' class='xr-var-data-in' type='checkbox'><label for='data-19d5f146-947d-42cf-b752-670fd34c0073' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([53.949861, 54.049861, 54.149861, 54.249861, 54.349861, 54.449861,
           54.549861, 54.649861, 54.749861, 54.849861, 54.949861, 55.049861,
           55.149861, 55.249861, 55.349861, 55.449861, 55.549861, 55.649861,
           55.749861, 55.849861, 55.949861, 56.049861, 56.149861, 56.249861,
           56.349861, 56.449861, 56.549861, 56.649861, 56.749861, 56.849861,
           56.949861, 57.049861, 57.149861, 57.249861, 57.349861, 57.449861,
           57.549861, 57.649861, 57.749861, 57.849861, 57.949861, 58.049861,
           58.149861, 58.249861, 58.349861, 58.449861, 58.549861, 58.649861,
           58.749861, 58.849861, 58.949861, 59.049861, 59.149861, 59.249861,
           59.349861, 59.449861, 59.549861])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>longitude</span></div><div class='xr-var-dims'>(longitude)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>21.05 21.15 21.25 ... 28.05 28.15</div><input id='attrs-87698f36-1c05-46e5-acdf-b59383dec149' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-87698f36-1c05-46e5-acdf-b59383dec149' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b3031833-f857-4fd0-935d-e3e1c6c3b4bd' class='xr-var-data-in' type='checkbox'><label for='data-b3031833-f857-4fd0-935d-e3e1c6c3b4bd' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([21.04986, 21.14986, 21.24986, 21.34986, 21.44986, 21.54986, 21.64986,
           21.74986, 21.84986, 21.94986, 22.04986, 22.14986, 22.24986, 22.34986,
           22.44986, 22.54986, 22.64986, 22.74986, 22.84986, 22.94986, 23.04986,
           23.14986, 23.24986, 23.34986, 23.44986, 23.54986, 23.64986, 23.74986,
           23.84986, 23.94986, 24.04986, 24.14986, 24.24986, 24.34986, 24.44986,
           24.54986, 24.64986, 24.74986, 24.84986, 24.94986, 25.04986, 25.14986,
           25.24986, 25.34986, 25.44986, 25.54986, 25.64986, 25.74986, 25.84986,
           25.94986, 26.04986, 26.14986, 26.24986, 26.34986, 26.44986, 26.54986,
           26.64986, 26.74986, 26.84986, 26.94986, 27.04986, 27.14986, 27.24986,
           27.34986, 27.44986, 27.54986, 27.64986, 27.74986, 27.84986, 27.94986,
           28.04986, 28.14986])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>ym</span></div><div class='xr-var-dims'>(ym)</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>&#x27;2001-01&#x27; &#x27;2001-02&#x27; ... &#x27;2020-12&#x27;</div><input id='attrs-3218a3a8-ad39-4247-99b1-8310e058f739' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-3218a3a8-ad39-4247-99b1-8310e058f739' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-46d733c4-b115-43f5-9f4e-9317b24d3c31' class='xr-var-data-in' type='checkbox'><label for='data-46d733c4-b115-43f5-9f4e-9317b24d3c31' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([&#x27;2001-01&#x27;, &#x27;2001-02&#x27;, &#x27;2001-03&#x27;, ..., &#x27;2020-10&#x27;, &#x27;2020-11&#x27;, &#x27;2020-12&#x27;],
          dtype=object)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>spatial_ref</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>int32</div><div class='xr-var-preview xr-preview'>0</div><input id='attrs-a40a59b6-62d6-4fa6-ac8d-b189d956cfc8' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-a40a59b6-62d6-4fa6-ac8d-b189d956cfc8' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-32ad7895-e74e-43ed-b5b1-eeaa5af38652' class='xr-var-data-in' type='checkbox'><label for='data-32ad7895-e74e-43ed-b5b1-eeaa5af38652' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>crs_wkt :</span></dt><dd>GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AXIS[&quot;Latitude&quot;,NORTH],AXIS[&quot;Longitude&quot;,EAST],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]]</dd><dt><span>semi_major_axis :</span></dt><dd>6378137.0</dd><dt><span>semi_minor_axis :</span></dt><dd>6356752.314245179</dd><dt><span>inverse_flattening :</span></dt><dd>298.257223563</dd><dt><span>reference_ellipsoid_name :</span></dt><dd>WGS 84</dd><dt><span>longitude_of_prime_meridian :</span></dt><dd>0.0</dd><dt><span>prime_meridian_name :</span></dt><dd>Greenwich</dd><dt><span>geographic_crs_name :</span></dt><dd>WGS 84</dd><dt><span>grid_mapping_name :</span></dt><dd>latitude_longitude</dd><dt><span>spatial_ref :</span></dt><dd>GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AXIS[&quot;Latitude&quot;,NORTH],AXIS[&quot;Longitude&quot;,EAST],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]]</dd></dl></div><div class='xr-var-data'><pre>array(0)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-8ef9126b-bc48-464d-bebe-a84ec155bb89' class='xr-section-summary-in' type='checkbox'  checked><label for='section-8ef9126b-bc48-464d-bebe-a84ec155bb89' class='xr-section-summary' >Data variables: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>temp_mean</span></div><div class='xr-var-dims'>(latitude, longitude, ym)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>nan nan nan nan ... nan nan nan nan</div><input id='attrs-e01386d6-daa6-4dc1-9b8d-79cf0de67436' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-e01386d6-daa6-4dc1-9b8d-79cf0de67436' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-28786c9a-58a6-4717-93bb-39531432bdc8' class='xr-var-data-in' type='checkbox'><label for='data-28786c9a-58a6-4717-93bb-39531432bdc8' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>grid_mapping :</span></dt><dd>spatial_ref</dd></dl></div><div class='xr-var-data'><pre>array([[[        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            ...,
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan]],

           [[        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
    ...
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [-1.7364515 , -6.0203576 , -3.3258066 , ...,  8.616452  ,
              3.806333  , -0.85096776],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan]],

           [[        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            ...,
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan]]], dtype=float32)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-faea86c6-97e0-46b9-a427-aea0a98620b7' class='xr-section-summary-in' type='checkbox'  checked><label for='section-faea86c6-97e0-46b9-a427-aea0a98620b7' class='xr-section-summary' >Attributes: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>grid_mapping :</span></dt><dd>spatial_ref</dd></dl></div></li></ul></div></div>



Our dataset has three dimensions, each of which has their own set of
coordinates. The spatial dimension is made up of 57 ``latitude`` and 72
``longitude`` coordinates, meaning that there are a total of
:math:`57\times72=4104` unique locations with temperature information in
the grid. Third dimension ``ym`` is an array of all 240 year–month pairs
(e.g. 2001-01) covering the 20 year study period.

The fourth coordinate ``spatial_ref`` is not a separate dimension, but
an indicator that a coordinate reference system (CRS) has been assigned
to the data. The ``grid_mapping`` attribute of this fourth coordinate
holds information about the CRS of the dataset. If we display the
``attrs`` property then we can see that the CRS in this case is WGS84.

.. code:: ipython3

    # Display CRS information
    display(temp_ds['spatial_ref'].attrs)



.. parsed-literal::

    {'crs_wkt': 'GEOGCS["WGS 84",DATUM["WGS_1984",SPHEROID["WGS 84",6378137,298.257223563,AUTHORITY["EPSG","7030"]],AUTHORITY["EPSG","6326"]],PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],UNIT["degree",0.0174532925199433,AUTHORITY["EPSG","9122"]],AXIS["Latitude",NORTH],AXIS["Longitude",EAST],AUTHORITY["EPSG","4326"]]',
     'semi_major_axis': 6378137.0,
     'semi_minor_axis': 6356752.314245179,
     'inverse_flattening': 298.257223563,
     'reference_ellipsoid_name': 'WGS 84',
     'longitude_of_prime_meridian': 0.0,
     'prime_meridian_name': 'Greenwich',
     'geographic_crs_name': 'WGS 84',
     'grid_mapping_name': 'latitude_longitude',
     'spatial_ref': 'GEOGCS["WGS 84",DATUM["WGS_1984",SPHEROID["WGS 84",6378137,298.257223563,AUTHORITY["EPSG","7030"]],AUTHORITY["EPSG","6326"]],PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],UNIT["degree",0.0174532925199433,AUTHORITY["EPSG","9122"]],AXIS["Latitude",NORTH],AXIS["Longitude",EAST],AUTHORITY["EPSG","4326"]]'}


Data variable ``temp_mean`` contains the monthly temperature amounts of
the Baltics in the period 2001–2020. Since each unique location has a
time series length of 240 months, the variable contains a total of
:math:`57\times72\times240=984960` values.

For a quick visualization of our data, let us plot the temperature
amount for January 2001. We can use the ``sel`` function and specify the
specific month by using the corresponding coordinate ``ym``. We also
need to include the data variable ``temp_mean`` to our plotting call.

.. code:: ipython3

    # Select a specific month
    temp_ds.sel(ym='2001-01')




.. raw:: html

    <div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
    <defs>
    <symbol id="icon-database" viewBox="0 0 32 32">
    <path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
    <path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
    <path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
    </symbol>
    <symbol id="icon-file-text2" viewBox="0 0 32 32">
    <path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
    <path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    <path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    <path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    </symbol>
    </defs>
    </svg>
    <style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
     *
     */

    :root {
      --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
      --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
      --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
      --xr-border-color: var(--jp-border-color2, #e0e0e0);
      --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
      --xr-background-color: var(--jp-layout-color0, white);
      --xr-background-color-row-even: var(--jp-layout-color1, white);
      --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
    }

    html[theme=dark],
    body.vscode-dark {
      --xr-font-color0: rgba(255, 255, 255, 1);
      --xr-font-color2: rgba(255, 255, 255, 0.54);
      --xr-font-color3: rgba(255, 255, 255, 0.38);
      --xr-border-color: #1F1F1F;
      --xr-disabled-color: #515151;
      --xr-background-color: #111111;
      --xr-background-color-row-even: #111111;
      --xr-background-color-row-odd: #313131;
    }

    .xr-wrap {
      display: block !important;
      min-width: 300px;
      max-width: 700px;
    }

    .xr-text-repr-fallback {
      /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
      display: none;
    }

    .xr-header {
      padding-top: 6px;
      padding-bottom: 6px;
      margin-bottom: 4px;
      border-bottom: solid 1px var(--xr-border-color);
    }

    .xr-header > div,
    .xr-header > ul {
      display: inline;
      margin-top: 0;
      margin-bottom: 0;
    }

    .xr-obj-type,
    .xr-array-name {
      margin-left: 2px;
      margin-right: 10px;
    }

    .xr-obj-type {
      color: var(--xr-font-color2);
    }

    .xr-sections {
      padding-left: 0 !important;
      display: grid;
      grid-template-columns: 150px auto auto 1fr 20px 20px;
    }

    .xr-section-item {
      display: contents;
    }

    .xr-section-item input {
      display: none;
    }

    .xr-section-item input + label {
      color: var(--xr-disabled-color);
    }

    .xr-section-item input:enabled + label {
      cursor: pointer;
      color: var(--xr-font-color2);
    }

    .xr-section-item input:enabled + label:hover {
      color: var(--xr-font-color0);
    }

    .xr-section-summary {
      grid-column: 1;
      color: var(--xr-font-color2);
      font-weight: 500;
    }

    .xr-section-summary > span {
      display: inline-block;
      padding-left: 0.5em;
    }

    .xr-section-summary-in:disabled + label {
      color: var(--xr-font-color2);
    }

    .xr-section-summary-in + label:before {
      display: inline-block;
      content: '►';
      font-size: 11px;
      width: 15px;
      text-align: center;
    }

    .xr-section-summary-in:disabled + label:before {
      color: var(--xr-disabled-color);
    }

    .xr-section-summary-in:checked + label:before {
      content: '▼';
    }

    .xr-section-summary-in:checked + label > span {
      display: none;
    }

    .xr-section-summary,
    .xr-section-inline-details {
      padding-top: 4px;
      padding-bottom: 4px;
    }

    .xr-section-inline-details {
      grid-column: 2 / -1;
    }

    .xr-section-details {
      display: none;
      grid-column: 1 / -1;
      margin-bottom: 5px;
    }

    .xr-section-summary-in:checked ~ .xr-section-details {
      display: contents;
    }

    .xr-array-wrap {
      grid-column: 1 / -1;
      display: grid;
      grid-template-columns: 20px auto;
    }

    .xr-array-wrap > label {
      grid-column: 1;
      vertical-align: top;
    }

    .xr-preview {
      color: var(--xr-font-color3);
    }

    .xr-array-preview,
    .xr-array-data {
      padding: 0 5px !important;
      grid-column: 2;
    }

    .xr-array-data,
    .xr-array-in:checked ~ .xr-array-preview {
      display: none;
    }

    .xr-array-in:checked ~ .xr-array-data,
    .xr-array-preview {
      display: inline-block;
    }

    .xr-dim-list {
      display: inline-block !important;
      list-style: none;
      padding: 0 !important;
      margin: 0;
    }

    .xr-dim-list li {
      display: inline-block;
      padding: 0;
      margin: 0;
    }

    .xr-dim-list:before {
      content: '(';
    }

    .xr-dim-list:after {
      content: ')';
    }

    .xr-dim-list li:not(:last-child):after {
      content: ',';
      padding-right: 5px;
    }

    .xr-has-index {
      font-weight: bold;
    }

    .xr-var-list,
    .xr-var-item {
      display: contents;
    }

    .xr-var-item > div,
    .xr-var-item label,
    .xr-var-item > .xr-var-name span {
      background-color: var(--xr-background-color-row-even);
      margin-bottom: 0;
    }

    .xr-var-item > .xr-var-name:hover span {
      padding-right: 5px;
    }

    .xr-var-list > li:nth-child(odd) > div,
    .xr-var-list > li:nth-child(odd) > label,
    .xr-var-list > li:nth-child(odd) > .xr-var-name span {
      background-color: var(--xr-background-color-row-odd);
    }

    .xr-var-name {
      grid-column: 1;
    }

    .xr-var-dims {
      grid-column: 2;
    }

    .xr-var-dtype {
      grid-column: 3;
      text-align: right;
      color: var(--xr-font-color2);
    }

    .xr-var-preview {
      grid-column: 4;
    }

    .xr-var-name,
    .xr-var-dims,
    .xr-var-dtype,
    .xr-preview,
    .xr-attrs dt {
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      padding-right: 10px;
    }

    .xr-var-name:hover,
    .xr-var-dims:hover,
    .xr-var-dtype:hover,
    .xr-attrs dt:hover {
      overflow: visible;
      width: auto;
      z-index: 1;
    }

    .xr-var-attrs,
    .xr-var-data {
      display: none;
      background-color: var(--xr-background-color) !important;
      padding-bottom: 5px !important;
    }

    .xr-var-attrs-in:checked ~ .xr-var-attrs,
    .xr-var-data-in:checked ~ .xr-var-data {
      display: block;
    }

    .xr-var-data > table {
      float: right;
    }

    .xr-var-name span,
    .xr-var-data,
    .xr-attrs {
      padding-left: 25px !important;
    }

    .xr-attrs,
    .xr-var-attrs,
    .xr-var-data {
      grid-column: 1 / -1;
    }

    dl.xr-attrs {
      padding: 0;
      margin: 0;
      display: grid;
      grid-template-columns: 125px auto;
    }

    .xr-attrs dt,
    .xr-attrs dd {
      padding: 0;
      margin: 0;
      float: left;
      padding-right: 10px;
      width: auto;
    }

    .xr-attrs dt {
      font-weight: normal;
      grid-column: 1;
    }

    .xr-attrs dt:hover span {
      display: inline-block;
      background: var(--xr-background-color);
      padding-right: 10px;
    }

    .xr-attrs dd {
      grid-column: 2;
      white-space: pre-wrap;
      word-break: break-all;
    }

    .xr-icon-database,
    .xr-icon-file-text2 {
      display: inline-block;
      vertical-align: middle;
      width: 1em;
      height: 1.5em !important;
      stroke-width: 0;
      stroke: currentColor;
      fill: currentColor;
    }
    </style><pre class='xr-text-repr-fallback'>&lt;xarray.Dataset&gt;
    Dimensions:      (latitude: 57, longitude: 72)
    Coordinates:
      * latitude     (latitude) float64 53.95 54.05 54.15 ... 59.35 59.45 59.55
      * longitude    (longitude) float64 21.05 21.15 21.25 ... 27.95 28.05 28.15
        ym           &lt;U7 &#x27;2001-01&#x27;
        spatial_ref  int32 0
    Data variables:
        temp_mean    (latitude, longitude) float32 nan nan nan nan ... nan nan nan
    Attributes:
        grid_mapping:  spatial_ref</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.Dataset</div></div><ul class='xr-sections'><li class='xr-section-item'><input id='section-ddb683fb-7887-48c9-ab0c-7d2d7e9153bf' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-ddb683fb-7887-48c9-ab0c-7d2d7e9153bf' class='xr-section-summary'  title='Expand/collapse section'>Dimensions:</label><div class='xr-section-inline-details'><ul class='xr-dim-list'><li><span class='xr-has-index'>latitude</span>: 57</li><li><span class='xr-has-index'>longitude</span>: 72</li></ul></div><div class='xr-section-details'></div></li><li class='xr-section-item'><input id='section-722a8b63-863c-4430-af24-a91c70007582' class='xr-section-summary-in' type='checkbox'  checked><label for='section-722a8b63-863c-4430-af24-a91c70007582' class='xr-section-summary' >Coordinates: <span>(4)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>latitude</span></div><div class='xr-var-dims'>(latitude)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>53.95 54.05 54.15 ... 59.45 59.55</div><input id='attrs-82a59da2-105e-43dc-941e-081e61096fdc' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-82a59da2-105e-43dc-941e-081e61096fdc' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-0e96450c-9041-47cb-b66b-2e8fcaf03241' class='xr-var-data-in' type='checkbox'><label for='data-0e96450c-9041-47cb-b66b-2e8fcaf03241' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([53.949861, 54.049861, 54.149861, 54.249861, 54.349861, 54.449861,
           54.549861, 54.649861, 54.749861, 54.849861, 54.949861, 55.049861,
           55.149861, 55.249861, 55.349861, 55.449861, 55.549861, 55.649861,
           55.749861, 55.849861, 55.949861, 56.049861, 56.149861, 56.249861,
           56.349861, 56.449861, 56.549861, 56.649861, 56.749861, 56.849861,
           56.949861, 57.049861, 57.149861, 57.249861, 57.349861, 57.449861,
           57.549861, 57.649861, 57.749861, 57.849861, 57.949861, 58.049861,
           58.149861, 58.249861, 58.349861, 58.449861, 58.549861, 58.649861,
           58.749861, 58.849861, 58.949861, 59.049861, 59.149861, 59.249861,
           59.349861, 59.449861, 59.549861])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>longitude</span></div><div class='xr-var-dims'>(longitude)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>21.05 21.15 21.25 ... 28.05 28.15</div><input id='attrs-023a4137-1342-4527-8bf5-82220ffda312' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-023a4137-1342-4527-8bf5-82220ffda312' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-34a66210-36df-47d6-b0fa-b2f83b5a94e4' class='xr-var-data-in' type='checkbox'><label for='data-34a66210-36df-47d6-b0fa-b2f83b5a94e4' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([21.04986, 21.14986, 21.24986, 21.34986, 21.44986, 21.54986, 21.64986,
           21.74986, 21.84986, 21.94986, 22.04986, 22.14986, 22.24986, 22.34986,
           22.44986, 22.54986, 22.64986, 22.74986, 22.84986, 22.94986, 23.04986,
           23.14986, 23.24986, 23.34986, 23.44986, 23.54986, 23.64986, 23.74986,
           23.84986, 23.94986, 24.04986, 24.14986, 24.24986, 24.34986, 24.44986,
           24.54986, 24.64986, 24.74986, 24.84986, 24.94986, 25.04986, 25.14986,
           25.24986, 25.34986, 25.44986, 25.54986, 25.64986, 25.74986, 25.84986,
           25.94986, 26.04986, 26.14986, 26.24986, 26.34986, 26.44986, 26.54986,
           26.64986, 26.74986, 26.84986, 26.94986, 27.04986, 27.14986, 27.24986,
           27.34986, 27.44986, 27.54986, 27.64986, 27.74986, 27.84986, 27.94986,
           28.04986, 28.14986])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>ym</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>&lt;U7</div><div class='xr-var-preview xr-preview'>&#x27;2001-01&#x27;</div><input id='attrs-fc8e0284-bd4c-43ca-ab7f-24a584fba689' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-fc8e0284-bd4c-43ca-ab7f-24a584fba689' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-99f78c34-aa40-48ec-9276-21951a364d6d' class='xr-var-data-in' type='checkbox'><label for='data-99f78c34-aa40-48ec-9276-21951a364d6d' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array(&#x27;2001-01&#x27;, dtype=&#x27;&lt;U7&#x27;)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>spatial_ref</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>int32</div><div class='xr-var-preview xr-preview'>0</div><input id='attrs-1195dfdf-0fef-4c17-96c1-5407a95c078c' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-1195dfdf-0fef-4c17-96c1-5407a95c078c' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-86ce82e1-79a3-4f56-8f0f-513de580a898' class='xr-var-data-in' type='checkbox'><label for='data-86ce82e1-79a3-4f56-8f0f-513de580a898' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>crs_wkt :</span></dt><dd>GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AXIS[&quot;Latitude&quot;,NORTH],AXIS[&quot;Longitude&quot;,EAST],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]]</dd><dt><span>semi_major_axis :</span></dt><dd>6378137.0</dd><dt><span>semi_minor_axis :</span></dt><dd>6356752.314245179</dd><dt><span>inverse_flattening :</span></dt><dd>298.257223563</dd><dt><span>reference_ellipsoid_name :</span></dt><dd>WGS 84</dd><dt><span>longitude_of_prime_meridian :</span></dt><dd>0.0</dd><dt><span>prime_meridian_name :</span></dt><dd>Greenwich</dd><dt><span>geographic_crs_name :</span></dt><dd>WGS 84</dd><dt><span>grid_mapping_name :</span></dt><dd>latitude_longitude</dd><dt><span>spatial_ref :</span></dt><dd>GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AXIS[&quot;Latitude&quot;,NORTH],AXIS[&quot;Longitude&quot;,EAST],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]]</dd></dl></div><div class='xr-var-data'><pre>array(0)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-fab27242-487f-446c-878f-a90cb3753f3e' class='xr-section-summary-in' type='checkbox'  checked><label for='section-fab27242-487f-446c-878f-a90cb3753f3e' class='xr-section-summary' >Data variables: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>temp_mean</span></div><div class='xr-var-dims'>(latitude, longitude)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>nan nan nan nan ... nan nan nan nan</div><input id='attrs-414b483a-4dc0-42aa-af1d-7110c0573338' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-414b483a-4dc0-42aa-af1d-7110c0573338' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-79872e7f-9226-4864-98cd-558b9f4fca10' class='xr-var-data-in' type='checkbox'><label for='data-79872e7f-9226-4864-98cd-558b9f4fca10' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>grid_mapping :</span></dt><dd>spatial_ref</dd></dl></div><div class='xr-var-data'><pre>array([[       nan,        nan,        nan, ...,        nan,        nan,
                   nan],
           [       nan,        nan,        nan, ...,        nan,        nan,
                   nan],
           [       nan,        nan,        nan, ...,        nan,        nan,
                   nan],
           ...,
           [       nan,        nan,        nan, ..., -1.7796775, -1.7503226,
            -1.7583872],
           [       nan,        nan,        nan, ...,        nan, -1.7364515,
                   nan],
           [       nan,        nan,        nan, ...,        nan,        nan,
                   nan]], dtype=float32)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-e98e0ab8-5109-434d-a3fe-26eafd464770' class='xr-section-summary-in' type='checkbox'  checked><label for='section-e98e0ab8-5109-434d-a3fe-26eafd464770' class='xr-section-summary' >Attributes: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>grid_mapping :</span></dt><dd>spatial_ref</dd></dl></div></li></ul></div></div>



.. code:: ipython3

    # Select and plot a specific month
    temp_ds.sel(ym='2001-01')['temp_mean'].plot()




.. parsed-literal::

    <matplotlib.collections.QuadMesh at 0x19f3be9ce80>




.. image:: workshop_files/workshop_19_1.png


Getting data ready for trend analysis
-------------------------------------

Now that we have familiarized ourselves with the dataset, let us prepare
it for trend analysis.

This preprocessing phase will consist of \* Converting the Xarray
Dataset into a Pandas DataFrame \* Grouping the data by location (grid
cell) and month \* Extracting the grouped temperature values as a NumPy
array

We will start by converting the dataset into a Pandas DataFrame using
the corresponding function ``to_dataframe``. Using ``reset_index`` will
flatten the DataFrame, so that we have a separate row for each of our
984960 monthly temperature records.

.. code:: ipython3

    # Convert the xarray to Pandas DF
    temp_df = temp_ds.to_dataframe().reset_index() # flatten
    display(temp_df)



.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }

        .dataframe tbody tr th {
            vertical-align: top;
        }

        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>latitude</th>
          <th>longitude</th>
          <th>ym</th>
          <th>temp_mean</th>
          <th>spatial_ref</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>53.949861</td>
          <td>21.04986</td>
          <td>2001-01</td>
          <td>NaN</td>
          <td>0</td>
        </tr>
        <tr>
          <th>1</th>
          <td>53.949861</td>
          <td>21.04986</td>
          <td>2001-02</td>
          <td>NaN</td>
          <td>0</td>
        </tr>
        <tr>
          <th>2</th>
          <td>53.949861</td>
          <td>21.04986</td>
          <td>2001-03</td>
          <td>NaN</td>
          <td>0</td>
        </tr>
        <tr>
          <th>3</th>
          <td>53.949861</td>
          <td>21.04986</td>
          <td>2001-04</td>
          <td>NaN</td>
          <td>0</td>
        </tr>
        <tr>
          <th>4</th>
          <td>53.949861</td>
          <td>21.04986</td>
          <td>2001-05</td>
          <td>NaN</td>
          <td>0</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>984955</th>
          <td>59.549861</td>
          <td>28.14986</td>
          <td>2020-08</td>
          <td>NaN</td>
          <td>0</td>
        </tr>
        <tr>
          <th>984956</th>
          <td>59.549861</td>
          <td>28.14986</td>
          <td>2020-09</td>
          <td>NaN</td>
          <td>0</td>
        </tr>
        <tr>
          <th>984957</th>
          <td>59.549861</td>
          <td>28.14986</td>
          <td>2020-10</td>
          <td>NaN</td>
          <td>0</td>
        </tr>
        <tr>
          <th>984958</th>
          <td>59.549861</td>
          <td>28.14986</td>
          <td>2020-11</td>
          <td>NaN</td>
          <td>0</td>
        </tr>
        <tr>
          <th>984959</th>
          <td>59.549861</td>
          <td>28.14986</td>
          <td>2020-12</td>
          <td>NaN</td>
          <td>0</td>
        </tr>
      </tbody>
    </table>
    <p>984960 rows × 5 columns</p>
    </div>


The input data includes a large number of missing values, indicated by
``NaN`` in the ``temp_mean`` column. These are grid cells located
outside the study area (i.e. the Baltic Sea, neighboring countries).

We should remove them with ``dropna`` before moving on.

.. code:: ipython3

    # Drop rows with missing temperature values
    temp_df = temp_df.dropna().reset_index()
    temp_df




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }

        .dataframe tbody tr th {
            vertical-align: top;
        }

        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>index</th>
          <th>latitude</th>
          <th>longitude</th>
          <th>ym</th>
          <th>temp_mean</th>
          <th>spatial_ref</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>6000</td>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2001-01</td>
          <td>-1.155484</td>
          <td>0</td>
        </tr>
        <tr>
          <th>1</th>
          <td>6001</td>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2001-02</td>
          <td>-2.777500</td>
          <td>0</td>
        </tr>
        <tr>
          <th>2</th>
          <td>6002</td>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2001-03</td>
          <td>0.088065</td>
          <td>0</td>
        </tr>
        <tr>
          <th>3</th>
          <td>6003</td>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2001-04</td>
          <td>8.223666</td>
          <td>0</td>
        </tr>
        <tr>
          <th>4</th>
          <td>6004</td>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2001-05</td>
          <td>13.019676</td>
          <td>0</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>611275</th>
          <td>980635</td>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>2020-08</td>
          <td>16.812258</td>
          <td>0</td>
        </tr>
        <tr>
          <th>611276</th>
          <td>980636</td>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>2020-09</td>
          <td>13.711668</td>
          <td>0</td>
        </tr>
        <tr>
          <th>611277</th>
          <td>980637</td>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>2020-10</td>
          <td>9.299999</td>
          <td>0</td>
        </tr>
        <tr>
          <th>611278</th>
          <td>980638</td>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>2020-11</td>
          <td>5.165666</td>
          <td>0</td>
        </tr>
        <tr>
          <th>611279</th>
          <td>980639</td>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>2020-12</td>
          <td>0.163548</td>
          <td>0</td>
        </tr>
      </tbody>
    </table>
    <p>611280 rows × 6 columns</p>
    </div>



Let us also split the ``ym`` column into separate ``year`` and ``month``
columns for more convenient grouping later on. We can use the function
``split`` for that and specify the delimiter in the original column.

.. code:: ipython3

    # Extract year and month from the observation date as new columns
    temp_df[['year', 'month']] = temp_df['ym'].str.split('-', expand=True) # expand=True tells Pandas that there are two output columns
    temp_df['year'] = temp_df['year'].astype(int) # astype(int) will convert string column into integer
    temp_df['month'] = temp_df['month'].astype(int)
    display(temp_df)



.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }

        .dataframe tbody tr th {
            vertical-align: top;
        }

        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>index</th>
          <th>latitude</th>
          <th>longitude</th>
          <th>ym</th>
          <th>temp_mean</th>
          <th>spatial_ref</th>
          <th>year</th>
          <th>month</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>6000</td>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2001-01</td>
          <td>-1.155484</td>
          <td>0</td>
          <td>2001</td>
          <td>1</td>
        </tr>
        <tr>
          <th>1</th>
          <td>6001</td>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2001-02</td>
          <td>-2.777500</td>
          <td>0</td>
          <td>2001</td>
          <td>2</td>
        </tr>
        <tr>
          <th>2</th>
          <td>6002</td>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2001-03</td>
          <td>0.088065</td>
          <td>0</td>
          <td>2001</td>
          <td>3</td>
        </tr>
        <tr>
          <th>3</th>
          <td>6003</td>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2001-04</td>
          <td>8.223666</td>
          <td>0</td>
          <td>2001</td>
          <td>4</td>
        </tr>
        <tr>
          <th>4</th>
          <td>6004</td>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2001-05</td>
          <td>13.019676</td>
          <td>0</td>
          <td>2001</td>
          <td>5</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>611275</th>
          <td>980635</td>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>2020-08</td>
          <td>16.812258</td>
          <td>0</td>
          <td>2020</td>
          <td>8</td>
        </tr>
        <tr>
          <th>611276</th>
          <td>980636</td>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>2020-09</td>
          <td>13.711668</td>
          <td>0</td>
          <td>2020</td>
          <td>9</td>
        </tr>
        <tr>
          <th>611277</th>
          <td>980637</td>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>2020-10</td>
          <td>9.299999</td>
          <td>0</td>
          <td>2020</td>
          <td>10</td>
        </tr>
        <tr>
          <th>611278</th>
          <td>980638</td>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>2020-11</td>
          <td>5.165666</td>
          <td>0</td>
          <td>2020</td>
          <td>11</td>
        </tr>
        <tr>
          <th>611279</th>
          <td>980639</td>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>2020-12</td>
          <td>0.163548</td>
          <td>0</td>
          <td>2020</td>
          <td>12</td>
        </tr>
      </tbody>
    </table>
    <p>611280 rows × 8 columns</p>
    </div>


In order to make sure that the monthly temperature values end up in the
correct order during grouping we should sort the DataFrame by year and
month. This can be done using the ``sort_values`` function.

.. code:: ipython3

    # Sort the DataFrame by year and month in ascending order
    temp_df.sort_values(['year', 'month'], ascending=[True, True], inplace=True) # inplace=True tells Pandas to modify the existing object

Now we can aggregate the DataFrame by location and month. We will use
``groupby`` to determine the columns used for grouping and assign
``temp_mean`` as the target column.

Pandas function ``apply`` can be used for applying different functions
rowwise in a DataFrame. In this case, we use parameter ``np.array`` to
tell ``apply`` to collect the grouped temperature values into a NumPy
array for each row.

.. code:: ipython3

    # Group locations by month and collect observation values to NumPy array
    temp_group_df = temp_df\
        .groupby(['latitude', 'longitude', 'month'])['temp_mean']\
        .apply(np.array)\
        .reset_index() # flatten the DataFrame
    display(temp_group_df)



.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }

        .dataframe tbody tr th {
            vertical-align: top;
        }

        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>latitude</th>
          <th>longitude</th>
          <th>month</th>
          <th>temp_mean</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>1</td>
          <td>[-1.155484, -1.569032, -5.0525813, -6.9348392,...</td>
        </tr>
        <tr>
          <th>1</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2</td>
          <td>[-2.7775, 2.2021434, -6.039285, -1.8844824, -5...</td>
        </tr>
        <tr>
          <th>2</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>3</td>
          <td>[0.08806454, 3.3738706, 0.8622579, 1.8580645, ...</td>
        </tr>
        <tr>
          <th>3</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>4</td>
          <td>[8.223666, 8.266, 5.5366664, 7.551, 7.8069997,...</td>
        </tr>
        <tr>
          <th>4</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>5</td>
          <td>[13.019676, 15.822902, 14.108708, 10.822258, 1...</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>30559</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>8</td>
          <td>[15.805161, 18.406773, 16.10516, 17.048388, 16...</td>
        </tr>
        <tr>
          <th>30560</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>9</td>
          <td>[12.335667, 11.58, 11.987667, 12.545667, 13.21...</td>
        </tr>
        <tr>
          <th>30561</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>10</td>
          <td>[8.4554825, 1.8396775, 4.703548, 6.5590324, 7....</td>
        </tr>
        <tr>
          <th>30562</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>11</td>
          <td>[0.766, -1.2219999, 2.9746668, 0.9679999, 3.94...</td>
        </tr>
        <tr>
          <th>30563</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>12</td>
          <td>[-6.6154838, -7.694516, 0.27806458, 0.46129018...</td>
        </tr>
      </tbody>
    </table>
    <p>30564 rows × 4 columns</p>
    </div>


Let us check the cell content in the array column for the first row. It
contains 20 years worth of monthly temperature mean values for January
at 53.9499 latitude and 23.5499 longitude.

We can use ``iloc`` to select a DataFrame row by index.

.. code:: ipython3

    # Display first row
    display(temp_group_df.iloc[0])

    # Display cell content in the grouped column
    display(temp_group_df.iloc[0]['temp_mean'])



.. parsed-literal::

    latitude                                             53.949861
    longitude                                             23.54986
    month                                                        1
    temp_mean    [-1.155484, -1.569032, -5.0525813, -6.9348392,...
    Name: 0, dtype: object



.. parsed-literal::

    array([ -1.155484  ,  -1.569032  ,  -5.0525813 ,  -6.9348392 ,
            -0.04483882,  -7.9774194 ,   1.5177417 ,  -0.8593547 ,
            -3.7196774 , -10.487097  ,  -2.7848382 ,  -3.04      ,
            -5.996774  ,  -5.3016133 ,  -0.33290324,  -5.7196774 ,
            -4.384839  ,  -1.638387  ,  -4.256451  ,   2.0070968 ],
          dtype=float32)


Trend analysis using the Mann-Kendall (MK) test
-----------------------------------------------

Now we have the temperature arrays in a column and can move on with the
trend analysis. We are going to use the
`pyMannKendall <https://pypi.org/project/pymannkendall/>`__ library for
that.

Although there are 13 different versions of the MK test available in
pyMannKendall, the ``original_test`` will suffice for now.

The test will return a tuple containing the named parameters \*
**trend:** tells the trend (increasing, decreasing or no trend) \*
**h:** True (if trend is present) or False (if the trend is absence) \*
**p:** p-value of the significance test \* **z:** normalized test
statistics \* **Tau:** Kendall Tau \* **s:** Mann-Kendal’s score \*
**var_s:** Variance S \* **slope:** Theil-Sen estimator/slope \*
**intercept:** intercept of Kendall-Theil Robust Line

.. code:: ipython3

    # Check the ouput of the MK test function
    array = temp_group_df.iloc[0]['temp_mean']
    result = mk.original_test(array, alpha=0.05)
    display(result)



.. parsed-literal::

    Mann_Kendall_Test(trend='no trend', h=False, p=0.9741177473553713, z=0.03244428422615251, Tau=0.010526315789473684, s=2.0, var_s=950.0, slope=0.03997344138858082, intercept=-3.759586398254384)


We will loop over the arrays in the ``temp_mean`` column, applying the
MK test each time using ``apply``. Confidence level—0.05 in our case—can
be assigned with parameter ``alpha``.

This can take some time as we need to repeat the process as many times
as there are rows, i.e. **30564** iterations in total.

.. code:: ipython3

    %%time
    # Apply MK test rowwise and collect result into new column
    temp_group_df['result'] = temp_group_df['temp_mean'].apply(lambda x: mk.original_test(x, alpha=0.05))
    display(temp_group_df)



.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }

        .dataframe tbody tr th {
            vertical-align: top;
        }

        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>latitude</th>
          <th>longitude</th>
          <th>month</th>
          <th>temp_mean</th>
          <th>result</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>1</td>
          <td>[-1.155484, -1.569032, -5.0525813, -6.9348392,...</td>
          <td>(no trend, False, 0.9741177473553713, 0.032444...</td>
        </tr>
        <tr>
          <th>1</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2</td>
          <td>[-2.7775, 2.2021434, -6.039285, -1.8844824, -5...</td>
          <td>(no trend, False, 0.2299690777236405, 1.200438...</td>
        </tr>
        <tr>
          <th>2</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>3</td>
          <td>[0.08806454, 3.3738706, 0.8622579, 1.8580645, ...</td>
          <td>(no trend, False, 0.2299690777236405, 1.200438...</td>
        </tr>
        <tr>
          <th>3</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>4</td>
          <td>[8.223666, 8.266, 5.5366664, 7.551, 7.8069997,...</td>
          <td>(no trend, False, 0.7702877264736667, 0.291998...</td>
        </tr>
        <tr>
          <th>4</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>5</td>
          <td>[13.019676, 15.822902, 14.108708, 10.822258, 1...</td>
          <td>(no trend, False, 0.9741177473553713, 0.032444...</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>30559</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>8</td>
          <td>[15.805161, 18.406773, 16.10516, 17.048388, 16...</td>
          <td>(no trend, False, 0.721176307474874, 0.3568871...</td>
        </tr>
        <tr>
          <th>30560</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>9</td>
          <td>[12.335667, 11.58, 11.987667, 12.545667, 13.21...</td>
          <td>(no trend, False, 0.07435290536856365, 1.78443...</td>
        </tr>
        <tr>
          <th>30561</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>10</td>
          <td>[8.4554825, 1.8396775, 4.703548, 6.5590324, 7....</td>
          <td>(no trend, False, 0.5376032363488861, 0.616441...</td>
        </tr>
        <tr>
          <th>30562</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>11</td>
          <td>[0.766, -1.2219999, 2.9746668, 0.9679999, 3.94...</td>
          <td>(no trend, False, 0.07435290536856365, 1.78443...</td>
        </tr>
        <tr>
          <th>30563</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>12</td>
          <td>[-6.6154838, -7.694516, 0.27806458, 0.46129018...</td>
          <td>(no trend, False, 0.18344722817247305, 1.33021...</td>
        </tr>
      </tbody>
    </table>
    <p>30564 rows × 5 columns</p>
    </div>


.. parsed-literal::

    CPU times: total: 45.2 s
    Wall time: 45.6 s


For our purposes, we will only extract parameters ``trend``, ``p`` and
``slope``, which have the indices **0**, **2** and **7**, respectively.
We can use these indices to extract the parameters into separate columns
from the MK test ``result`` column.

.. code:: ipython3

    temp_group_df['trend'] = temp_group_df['result'].apply(lambda x: x[0])
    temp_group_df['p'] = temp_group_df['result'].apply(lambda x: x[2])
    temp_group_df['slope'] = temp_group_df['result'].apply(lambda x: x[7])
    temp_group_df




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }

        .dataframe tbody tr th {
            vertical-align: top;
        }

        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>latitude</th>
          <th>longitude</th>
          <th>month</th>
          <th>temp_mean</th>
          <th>result</th>
          <th>trend</th>
          <th>p</th>
          <th>slope</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>1</td>
          <td>[-1.155484, -1.569032, -5.0525813, -6.9348392,...</td>
          <td>(no trend, False, 0.9741177473553713, 0.032444...</td>
          <td>no trend</td>
          <td>0.974118</td>
          <td>0.039973</td>
        </tr>
        <tr>
          <th>1</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2</td>
          <td>[-2.7775, 2.2021434, -6.039285, -1.8844824, -5...</td>
          <td>(no trend, False, 0.2299690777236405, 1.200438...</td>
          <td>no trend</td>
          <td>0.229969</td>
          <td>0.251141</td>
        </tr>
        <tr>
          <th>2</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>3</td>
          <td>[0.08806454, 3.3738706, 0.8622579, 1.8580645, ...</td>
          <td>(no trend, False, 0.2299690777236405, 1.200438...</td>
          <td>no trend</td>
          <td>0.229969</td>
          <td>0.135081</td>
        </tr>
        <tr>
          <th>3</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>4</td>
          <td>[8.223666, 8.266, 5.5366664, 7.551, 7.8069997,...</td>
          <td>(no trend, False, 0.7702877264736667, 0.291998...</td>
          <td>no trend</td>
          <td>0.770288</td>
          <td>0.012048</td>
        </tr>
        <tr>
          <th>4</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>5</td>
          <td>[13.019676, 15.822902, 14.108708, 10.822258, 1...</td>
          <td>(no trend, False, 0.9741177473553713, 0.032444...</td>
          <td>no trend</td>
          <td>0.974118</td>
          <td>0.000922</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>30559</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>8</td>
          <td>[15.805161, 18.406773, 16.10516, 17.048388, 16...</td>
          <td>(no trend, False, 0.721176307474874, 0.3568871...</td>
          <td>no trend</td>
          <td>0.721176</td>
          <td>0.018852</td>
        </tr>
        <tr>
          <th>30560</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>9</td>
          <td>[12.335667, 11.58, 11.987667, 12.545667, 13.21...</td>
          <td>(no trend, False, 0.07435290536856365, 1.78443...</td>
          <td>no trend</td>
          <td>0.074353</td>
          <td>0.066889</td>
        </tr>
        <tr>
          <th>30561</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>10</td>
          <td>[8.4554825, 1.8396775, 4.703548, 6.5590324, 7....</td>
          <td>(no trend, False, 0.5376032363488861, 0.616441...</td>
          <td>no trend</td>
          <td>0.537603</td>
          <td>0.042174</td>
        </tr>
        <tr>
          <th>30562</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>11</td>
          <td>[0.766, -1.2219999, 2.9746668, 0.9679999, 3.94...</td>
          <td>(no trend, False, 0.07435290536856365, 1.78443...</td>
          <td>no trend</td>
          <td>0.074353</td>
          <td>0.129285</td>
        </tr>
        <tr>
          <th>30563</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>12</td>
          <td>[-6.6154838, -7.694516, 0.27806458, 0.46129018...</td>
          <td>(no trend, False, 0.18344722817247305, 1.33021...</td>
          <td>no trend</td>
          <td>0.183447</td>
          <td>0.145511</td>
        </tr>
      </tbody>
    </table>
    <p>30564 rows × 8 columns</p>
    </div>



We can use Pandas function ``value_counts`` to display how many grid
cells ended up in each of the trend classes. We will do this for each
month, so we are going to group ``temp_group_df`` again by month and
loop through it.

.. code:: ipython3

    # Loop through months and display significant trend count
    for month, df in temp_group_df.groupby('month'):
        print('Siginificant trends in month number ' + str(month))
        display(df['trend'].value_counts())
        print('\n')


.. parsed-literal::

    Siginificant trends in month number 1



.. parsed-literal::

    no trend    2547
    Name: trend, dtype: int64


.. parsed-literal::



    Siginificant trends in month number 2



.. parsed-literal::

    no trend    2547
    Name: trend, dtype: int64


.. parsed-literal::



    Siginificant trends in month number 3



.. parsed-literal::

    no trend    2547
    Name: trend, dtype: int64


.. parsed-literal::



    Siginificant trends in month number 4



.. parsed-literal::

    no trend      2536
    increasing      11
    Name: trend, dtype: int64


.. parsed-literal::



    Siginificant trends in month number 5



.. parsed-literal::

    no trend    2547
    Name: trend, dtype: int64


.. parsed-literal::



    Siginificant trends in month number 6



.. parsed-literal::

    no trend      1531
    increasing    1016
    Name: trend, dtype: int64


.. parsed-literal::



    Siginificant trends in month number 7



.. parsed-literal::

    no trend      2436
    decreasing     111
    Name: trend, dtype: int64


.. parsed-literal::



    Siginificant trends in month number 8



.. parsed-literal::

    no trend    2547
    Name: trend, dtype: int64


.. parsed-literal::



    Siginificant trends in month number 9



.. parsed-literal::

    no trend      1589
    increasing     958
    Name: trend, dtype: int64


.. parsed-literal::



    Siginificant trends in month number 10



.. parsed-literal::

    no trend      2469
    increasing      78
    Name: trend, dtype: int64


.. parsed-literal::



    Siginificant trends in month number 11



.. parsed-literal::

    increasing    2067
    no trend       480
    Name: trend, dtype: int64


.. parsed-literal::



    Siginificant trends in month number 12



.. parsed-literal::

    no trend      2543
    increasing       4
    Name: trend, dtype: int64


.. parsed-literal::





We can see that a large majority of locations did not show a significant
trend during the study period as most cells indicated no trend.

Next we will convert the parameter ``trend`` into a new numeric column,
which is required for plotting later on.

Let us first create a function that assigns a new numeric trend class
based on ``p`` and ``slope``. Negative significant trends will be
indicated by **-1**, positive trends by **1** and **0** will be assigned
to locations where no trend was detected.

.. code:: ipython3

    # Function for numeric trend conversion
    def numeric_trend(slope, p):
        if slope < 0 and p < 0.05:
            return -1
        elif slope > 0 and p < 0.05:
            return 1
        else:
            return 0

Now we can use ``apply`` and its ``lambda`` parameter to generate the
``numeric_trend`` column. The ``lambda`` parameter is useful for
applying a function to each row.

In our case, we are taking the values of columns ``slope`` and ``p`` for
each row as input parameters for our function ``numeric_trend``, which
we created in the last step. Parameter ``axis=1`` will tell the function
that we are iterating over rows rather than columns.

Some examples of ``lambda`` use can be found at
https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.apply.html.

.. code:: ipython3

    # Create a numeric version of the trend column
    temp_group_df['numeric_trend'] = temp_group_df.apply(lambda x: numeric_trend(x['slope'], x['p']), axis=1)
    display(temp_group_df)



.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }

        .dataframe tbody tr th {
            vertical-align: top;
        }

        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>latitude</th>
          <th>longitude</th>
          <th>month</th>
          <th>temp_mean</th>
          <th>result</th>
          <th>trend</th>
          <th>p</th>
          <th>slope</th>
          <th>numeric_trend</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>1</td>
          <td>[-1.155484, -1.569032, -5.0525813, -6.9348392,...</td>
          <td>(no trend, False, 0.9741177473553713, 0.032444...</td>
          <td>no trend</td>
          <td>0.974118</td>
          <td>0.039973</td>
          <td>0</td>
        </tr>
        <tr>
          <th>1</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>2</td>
          <td>[-2.7775, 2.2021434, -6.039285, -1.8844824, -5...</td>
          <td>(no trend, False, 0.2299690777236405, 1.200438...</td>
          <td>no trend</td>
          <td>0.229969</td>
          <td>0.251141</td>
          <td>0</td>
        </tr>
        <tr>
          <th>2</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>3</td>
          <td>[0.08806454, 3.3738706, 0.8622579, 1.8580645, ...</td>
          <td>(no trend, False, 0.2299690777236405, 1.200438...</td>
          <td>no trend</td>
          <td>0.229969</td>
          <td>0.135081</td>
          <td>0</td>
        </tr>
        <tr>
          <th>3</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>4</td>
          <td>[8.223666, 8.266, 5.5366664, 7.551, 7.8069997,...</td>
          <td>(no trend, False, 0.7702877264736667, 0.291998...</td>
          <td>no trend</td>
          <td>0.770288</td>
          <td>0.012048</td>
          <td>0</td>
        </tr>
        <tr>
          <th>4</th>
          <td>53.949861</td>
          <td>23.54986</td>
          <td>5</td>
          <td>[13.019676, 15.822902, 14.108708, 10.822258, 1...</td>
          <td>(no trend, False, 0.9741177473553713, 0.032444...</td>
          <td>no trend</td>
          <td>0.974118</td>
          <td>0.000922</td>
          <td>0</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>30559</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>8</td>
          <td>[15.805161, 18.406773, 16.10516, 17.048388, 16...</td>
          <td>(no trend, False, 0.721176307474874, 0.3568871...</td>
          <td>no trend</td>
          <td>0.721176</td>
          <td>0.018852</td>
          <td>0</td>
        </tr>
        <tr>
          <th>30560</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>9</td>
          <td>[12.335667, 11.58, 11.987667, 12.545667, 13.21...</td>
          <td>(no trend, False, 0.07435290536856365, 1.78443...</td>
          <td>no trend</td>
          <td>0.074353</td>
          <td>0.066889</td>
          <td>0</td>
        </tr>
        <tr>
          <th>30561</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>10</td>
          <td>[8.4554825, 1.8396775, 4.703548, 6.5590324, 7....</td>
          <td>(no trend, False, 0.5376032363488861, 0.616441...</td>
          <td>no trend</td>
          <td>0.537603</td>
          <td>0.042174</td>
          <td>0</td>
        </tr>
        <tr>
          <th>30562</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>11</td>
          <td>[0.766, -1.2219999, 2.9746668, 0.9679999, 3.94...</td>
          <td>(no trend, False, 0.07435290536856365, 1.78443...</td>
          <td>no trend</td>
          <td>0.074353</td>
          <td>0.129285</td>
          <td>0</td>
        </tr>
        <tr>
          <th>30563</th>
          <td>59.549861</td>
          <td>26.34986</td>
          <td>12</td>
          <td>[-6.6154838, -7.694516, 0.27806458, 0.46129018...</td>
          <td>(no trend, False, 0.18344722817247305, 1.33021...</td>
          <td>no trend</td>
          <td>0.183447</td>
          <td>0.145511</td>
          <td>0</td>
        </tr>
      </tbody>
    </table>
    <p>30564 rows × 9 columns</p>
    </div>


Before plotting the trend results we need to convert our DataFrame back
into an Xarray dataset. This can be done with ``to_xarray``, but first
we need to use ``set_index`` and assign an index to the DataFrame. The
columns used for the index will end up as dimensions in the resulting
Xarray dataset.

Let us also assign a CRS with ``write_crs`` from
`rioxarray <https://corteva.github.io/rioxarray/stable/>`__, which is a
supplementary Xarray library used for spatial operations.

.. code:: ipython3

    # Convert the DataFrame back to xarray Dataset and assign CRS
    temp_group_df.set_index(['latitude', 'longitude', 'month'], inplace=True)
    temp_array = temp_group_df.to_xarray()
    temp_array.rio.write_crs('epsg:4326', inplace=True)
    display(temp_array)



.. raw:: html

    <div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
    <defs>
    <symbol id="icon-database" viewBox="0 0 32 32">
    <path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
    <path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
    <path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
    </symbol>
    <symbol id="icon-file-text2" viewBox="0 0 32 32">
    <path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
    <path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    <path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    <path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    </symbol>
    </defs>
    </svg>
    <style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
     *
     */

    :root {
      --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
      --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
      --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
      --xr-border-color: var(--jp-border-color2, #e0e0e0);
      --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
      --xr-background-color: var(--jp-layout-color0, white);
      --xr-background-color-row-even: var(--jp-layout-color1, white);
      --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
    }

    html[theme=dark],
    body.vscode-dark {
      --xr-font-color0: rgba(255, 255, 255, 1);
      --xr-font-color2: rgba(255, 255, 255, 0.54);
      --xr-font-color3: rgba(255, 255, 255, 0.38);
      --xr-border-color: #1F1F1F;
      --xr-disabled-color: #515151;
      --xr-background-color: #111111;
      --xr-background-color-row-even: #111111;
      --xr-background-color-row-odd: #313131;
    }

    .xr-wrap {
      display: block !important;
      min-width: 300px;
      max-width: 700px;
    }

    .xr-text-repr-fallback {
      /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
      display: none;
    }

    .xr-header {
      padding-top: 6px;
      padding-bottom: 6px;
      margin-bottom: 4px;
      border-bottom: solid 1px var(--xr-border-color);
    }

    .xr-header > div,
    .xr-header > ul {
      display: inline;
      margin-top: 0;
      margin-bottom: 0;
    }

    .xr-obj-type,
    .xr-array-name {
      margin-left: 2px;
      margin-right: 10px;
    }

    .xr-obj-type {
      color: var(--xr-font-color2);
    }

    .xr-sections {
      padding-left: 0 !important;
      display: grid;
      grid-template-columns: 150px auto auto 1fr 20px 20px;
    }

    .xr-section-item {
      display: contents;
    }

    .xr-section-item input {
      display: none;
    }

    .xr-section-item input + label {
      color: var(--xr-disabled-color);
    }

    .xr-section-item input:enabled + label {
      cursor: pointer;
      color: var(--xr-font-color2);
    }

    .xr-section-item input:enabled + label:hover {
      color: var(--xr-font-color0);
    }

    .xr-section-summary {
      grid-column: 1;
      color: var(--xr-font-color2);
      font-weight: 500;
    }

    .xr-section-summary > span {
      display: inline-block;
      padding-left: 0.5em;
    }

    .xr-section-summary-in:disabled + label {
      color: var(--xr-font-color2);
    }

    .xr-section-summary-in + label:before {
      display: inline-block;
      content: '►';
      font-size: 11px;
      width: 15px;
      text-align: center;
    }

    .xr-section-summary-in:disabled + label:before {
      color: var(--xr-disabled-color);
    }

    .xr-section-summary-in:checked + label:before {
      content: '▼';
    }

    .xr-section-summary-in:checked + label > span {
      display: none;
    }

    .xr-section-summary,
    .xr-section-inline-details {
      padding-top: 4px;
      padding-bottom: 4px;
    }

    .xr-section-inline-details {
      grid-column: 2 / -1;
    }

    .xr-section-details {
      display: none;
      grid-column: 1 / -1;
      margin-bottom: 5px;
    }

    .xr-section-summary-in:checked ~ .xr-section-details {
      display: contents;
    }

    .xr-array-wrap {
      grid-column: 1 / -1;
      display: grid;
      grid-template-columns: 20px auto;
    }

    .xr-array-wrap > label {
      grid-column: 1;
      vertical-align: top;
    }

    .xr-preview {
      color: var(--xr-font-color3);
    }

    .xr-array-preview,
    .xr-array-data {
      padding: 0 5px !important;
      grid-column: 2;
    }

    .xr-array-data,
    .xr-array-in:checked ~ .xr-array-preview {
      display: none;
    }

    .xr-array-in:checked ~ .xr-array-data,
    .xr-array-preview {
      display: inline-block;
    }

    .xr-dim-list {
      display: inline-block !important;
      list-style: none;
      padding: 0 !important;
      margin: 0;
    }

    .xr-dim-list li {
      display: inline-block;
      padding: 0;
      margin: 0;
    }

    .xr-dim-list:before {
      content: '(';
    }

    .xr-dim-list:after {
      content: ')';
    }

    .xr-dim-list li:not(:last-child):after {
      content: ',';
      padding-right: 5px;
    }

    .xr-has-index {
      font-weight: bold;
    }

    .xr-var-list,
    .xr-var-item {
      display: contents;
    }

    .xr-var-item > div,
    .xr-var-item label,
    .xr-var-item > .xr-var-name span {
      background-color: var(--xr-background-color-row-even);
      margin-bottom: 0;
    }

    .xr-var-item > .xr-var-name:hover span {
      padding-right: 5px;
    }

    .xr-var-list > li:nth-child(odd) > div,
    .xr-var-list > li:nth-child(odd) > label,
    .xr-var-list > li:nth-child(odd) > .xr-var-name span {
      background-color: var(--xr-background-color-row-odd);
    }

    .xr-var-name {
      grid-column: 1;
    }

    .xr-var-dims {
      grid-column: 2;
    }

    .xr-var-dtype {
      grid-column: 3;
      text-align: right;
      color: var(--xr-font-color2);
    }

    .xr-var-preview {
      grid-column: 4;
    }

    .xr-var-name,
    .xr-var-dims,
    .xr-var-dtype,
    .xr-preview,
    .xr-attrs dt {
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      padding-right: 10px;
    }

    .xr-var-name:hover,
    .xr-var-dims:hover,
    .xr-var-dtype:hover,
    .xr-attrs dt:hover {
      overflow: visible;
      width: auto;
      z-index: 1;
    }

    .xr-var-attrs,
    .xr-var-data {
      display: none;
      background-color: var(--xr-background-color) !important;
      padding-bottom: 5px !important;
    }

    .xr-var-attrs-in:checked ~ .xr-var-attrs,
    .xr-var-data-in:checked ~ .xr-var-data {
      display: block;
    }

    .xr-var-data > table {
      float: right;
    }

    .xr-var-name span,
    .xr-var-data,
    .xr-attrs {
      padding-left: 25px !important;
    }

    .xr-attrs,
    .xr-var-attrs,
    .xr-var-data {
      grid-column: 1 / -1;
    }

    dl.xr-attrs {
      padding: 0;
      margin: 0;
      display: grid;
      grid-template-columns: 125px auto;
    }

    .xr-attrs dt,
    .xr-attrs dd {
      padding: 0;
      margin: 0;
      float: left;
      padding-right: 10px;
      width: auto;
    }

    .xr-attrs dt {
      font-weight: normal;
      grid-column: 1;
    }

    .xr-attrs dt:hover span {
      display: inline-block;
      background: var(--xr-background-color);
      padding-right: 10px;
    }

    .xr-attrs dd {
      grid-column: 2;
      white-space: pre-wrap;
      word-break: break-all;
    }

    .xr-icon-database,
    .xr-icon-file-text2 {
      display: inline-block;
      vertical-align: middle;
      width: 1em;
      height: 1.5em !important;
      stroke-width: 0;
      stroke: currentColor;
      fill: currentColor;
    }
    </style><pre class='xr-text-repr-fallback'>&lt;xarray.Dataset&gt;
    Dimensions:        (latitude: 57, longitude: 72, month: 12)
    Coordinates:
      * latitude       (latitude) float64 53.95 54.05 54.15 ... 59.35 59.45 59.55
      * longitude      (longitude) float64 21.05 21.15 21.25 ... 27.95 28.05 28.15
      * month          (month) int64 1 2 3 4 5 6 7 8 9 10 11 12
        spatial_ref    int32 0
    Data variables:
        temp_mean      (latitude, longitude, month) object nan nan nan ... nan nan
        result         (latitude, longitude, month) object nan nan nan ... nan nan
        trend          (latitude, longitude, month) object nan nan nan ... nan nan
        p              (latitude, longitude, month) float64 nan nan nan ... nan nan
        slope          (latitude, longitude, month) float64 nan nan nan ... nan nan
        numeric_trend  (latitude, longitude, month) float64 nan nan nan ... nan nan</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.Dataset</div></div><ul class='xr-sections'><li class='xr-section-item'><input id='section-1e1ac0d9-02dc-40a9-baa6-3e5b2daef44b' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-1e1ac0d9-02dc-40a9-baa6-3e5b2daef44b' class='xr-section-summary'  title='Expand/collapse section'>Dimensions:</label><div class='xr-section-inline-details'><ul class='xr-dim-list'><li><span class='xr-has-index'>latitude</span>: 57</li><li><span class='xr-has-index'>longitude</span>: 72</li><li><span class='xr-has-index'>month</span>: 12</li></ul></div><div class='xr-section-details'></div></li><li class='xr-section-item'><input id='section-05685a08-0647-4caf-a5ad-b43a99b24962' class='xr-section-summary-in' type='checkbox'  checked><label for='section-05685a08-0647-4caf-a5ad-b43a99b24962' class='xr-section-summary' >Coordinates: <span>(4)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>latitude</span></div><div class='xr-var-dims'>(latitude)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>53.95 54.05 54.15 ... 59.45 59.55</div><input id='attrs-0b9e91fc-3342-48f7-bb71-92a731975c52' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-0b9e91fc-3342-48f7-bb71-92a731975c52' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-571ce859-2ea1-4b60-a49a-e73e16ce38a2' class='xr-var-data-in' type='checkbox'><label for='data-571ce859-2ea1-4b60-a49a-e73e16ce38a2' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([53.949861, 54.049861, 54.149861, 54.249861, 54.349861, 54.449861,
           54.549861, 54.649861, 54.749861, 54.849861, 54.949861, 55.049861,
           55.149861, 55.249861, 55.349861, 55.449861, 55.549861, 55.649861,
           55.749861, 55.849861, 55.949861, 56.049861, 56.149861, 56.249861,
           56.349861, 56.449861, 56.549861, 56.649861, 56.749861, 56.849861,
           56.949861, 57.049861, 57.149861, 57.249861, 57.349861, 57.449861,
           57.549861, 57.649861, 57.749861, 57.849861, 57.949861, 58.049861,
           58.149861, 58.249861, 58.349861, 58.449861, 58.549861, 58.649861,
           58.749861, 58.849861, 58.949861, 59.049861, 59.149861, 59.249861,
           59.349861, 59.449861, 59.549861])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>longitude</span></div><div class='xr-var-dims'>(longitude)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>21.05 21.15 21.25 ... 28.05 28.15</div><input id='attrs-ad6a4a91-6d49-478c-adc3-534721f527dd' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-ad6a4a91-6d49-478c-adc3-534721f527dd' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-f7aa663d-2863-44c2-adf8-eed10f71f96a' class='xr-var-data-in' type='checkbox'><label for='data-f7aa663d-2863-44c2-adf8-eed10f71f96a' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([21.04986, 21.14986, 21.24986, 21.34986, 21.44986, 21.54986, 21.64986,
           21.74986, 21.84986, 21.94986, 22.04986, 22.14986, 22.24986, 22.34986,
           22.44986, 22.54986, 22.64986, 22.74986, 22.84986, 22.94986, 23.04986,
           23.14986, 23.24986, 23.34986, 23.44986, 23.54986, 23.64986, 23.74986,
           23.84986, 23.94986, 24.04986, 24.14986, 24.24986, 24.34986, 24.44986,
           24.54986, 24.64986, 24.74986, 24.84986, 24.94986, 25.04986, 25.14986,
           25.24986, 25.34986, 25.44986, 25.54986, 25.64986, 25.74986, 25.84986,
           25.94986, 26.04986, 26.14986, 26.24986, 26.34986, 26.44986, 26.54986,
           26.64986, 26.74986, 26.84986, 26.94986, 27.04986, 27.14986, 27.24986,
           27.34986, 27.44986, 27.54986, 27.64986, 27.74986, 27.84986, 27.94986,
           28.04986, 28.14986])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>month</span></div><div class='xr-var-dims'>(month)</div><div class='xr-var-dtype'>int64</div><div class='xr-var-preview xr-preview'>1 2 3 4 5 6 7 8 9 10 11 12</div><input id='attrs-0438c531-9c4e-4e9c-bbd4-4c184f99e7dd' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-0438c531-9c4e-4e9c-bbd4-4c184f99e7dd' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-87acc037-66be-4bb6-b357-6cf9122b7854' class='xr-var-data-in' type='checkbox'><label for='data-87acc037-66be-4bb6-b357-6cf9122b7854' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([ 1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12], dtype=int64)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>spatial_ref</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>int32</div><div class='xr-var-preview xr-preview'>0</div><input id='attrs-7fb39716-2ca3-49a6-a94f-eab32166250f' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-7fb39716-2ca3-49a6-a94f-eab32166250f' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-c2af89bd-88fc-4ef9-8273-f42793b58801' class='xr-var-data-in' type='checkbox'><label for='data-c2af89bd-88fc-4ef9-8273-f42793b58801' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>crs_wkt :</span></dt><dd>GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AXIS[&quot;Latitude&quot;,NORTH],AXIS[&quot;Longitude&quot;,EAST],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]]</dd><dt><span>semi_major_axis :</span></dt><dd>6378137.0</dd><dt><span>semi_minor_axis :</span></dt><dd>6356752.314245179</dd><dt><span>inverse_flattening :</span></dt><dd>298.257223563</dd><dt><span>reference_ellipsoid_name :</span></dt><dd>WGS 84</dd><dt><span>longitude_of_prime_meridian :</span></dt><dd>0.0</dd><dt><span>prime_meridian_name :</span></dt><dd>Greenwich</dd><dt><span>geographic_crs_name :</span></dt><dd>WGS 84</dd><dt><span>grid_mapping_name :</span></dt><dd>latitude_longitude</dd><dt><span>spatial_ref :</span></dt><dd>GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AXIS[&quot;Latitude&quot;,NORTH],AXIS[&quot;Longitude&quot;,EAST],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]]</dd></dl></div><div class='xr-var-data'><pre>array(0)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-b136486f-b456-4fd6-88e0-e22cfda0d2ad' class='xr-section-summary-in' type='checkbox'  checked><label for='section-b136486f-b456-4fd6-88e0-e22cfda0d2ad' class='xr-section-summary' >Data variables: <span>(6)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>temp_mean</span></div><div class='xr-var-dims'>(latitude, longitude, month)</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>nan nan nan nan ... nan nan nan nan</div><input id='attrs-a641dc34-23ab-4620-90b6-be741665c7d7' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-a641dc34-23ab-4620-90b6-be741665c7d7' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-201df3ea-bf47-49f2-bc11-17ecab539092' class='xr-var-data-in' type='checkbox'><label for='data-201df3ea-bf47-49f2-bc11-17ecab539092' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([[[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,

             array([-2.5999896e-02, -1.6996669e+00,  2.6086669e+00,  7.7733320e-01,
                     3.4280002e+00,  2.2163332e+00,  3.3333460e-03,  3.0209994e+00,
                     2.1780002e+00,  5.3666675e-01,  4.4736657e+00,  3.0280004e+00,
                     4.4746671e+00,  4.8666665e-01,  3.6960001e+00, -1.2796664e+00,
                     2.0933332e+00,  2.6239998e+00,  1.7930000e+00,  3.8063331e+00],
                   dtype=float32)                                                   ,
             array([-7.8674183 , -9.10613   , -0.05677424, -0.13290322, -2.8580642 ,
                     3.643871  ,  1.0287098 , -0.37387088, -4.9851604 , -7.7264514 ,
                     1.9135484 , -7.466775  ,  1.2554839 , -1.3664516 ,  2.3845162 ,
                    -0.3822581 ,  0.26193547, -3.0483873 ,  1.6403228 , -0.85096776],
                   dtype=float32)                                                    ],
            [nan, nan, nan, ..., nan, nan, nan]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan]]], dtype=object)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>result</span></div><div class='xr-var-dims'>(latitude, longitude, month)</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>nan nan nan nan ... nan nan nan nan</div><input id='attrs-164fb11a-dde3-42db-9c39-6a8cff2ebd7f' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-164fb11a-dde3-42db-9c39-6a8cff2ebd7f' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-83bbfddc-4146-4cd2-bd5f-cb6e412def1e' class='xr-var-data-in' type='checkbox'><label for='data-83bbfddc-4146-4cd2-bd5f-cb6e412def1e' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([[[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],

            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,
            [nan, nan, nan, ..., nan, nan, nan],
            [Mann_Kendall_Test(trend=&#x27;no trend&#x27;, h=False, p=0.8711314915971582, z=-0.16222142113076254, Tau=-0.031578947368421054, s=-6.0, var_s=950.0, slope=-0.03012464023103901, intercept=-3.8907512329372707),
             Mann_Kendall_Test(trend=&#x27;no trend&#x27;, h=False, p=0.2561449666035265, z=1.1355499479153377, Tau=0.18947368421052632, s=36.0, var_s=950.0, slope=0.26125001907348633, intercept=-7.277405023574829),
             Mann_Kendall_Test(trend=&#x27;no trend&#x27;, h=False, p=0.18344722817247305, z=1.3302156532722529, Tau=0.22105263157894736, s=42.0, var_s=950.0, slope=0.155095394844046, intercept=-2.333728845661076),
             ...,
             Mann_Kendall_Test(trend=&#x27;no trend&#x27;, h=False, p=0.9224620671867603, z=-0.09733285267845752, Tau=-0.021052631578947368, s=-4.0, var_s=950.0, slope=-0.0028840700785319013, intercept=6.485301812489827),
             Mann_Kendall_Test(trend=&#x27;no trend&#x27;, h=False, p=0.20575410127305505, z=1.2653270848199478, Tau=0.21052631578947367, s=40.0, var_s=950.0, slope=0.09686012778963361, intercept=1.2769954672881534),
             Mann_Kendall_Test(trend=&#x27;no trend&#x27;, h=False, p=0.3467641835161843, z=0.9408842425584227, Tau=0.15789473684210525, s=30.0, var_s=950.0, slope=0.14403582943810356, intercept=-1.7464048630661433)],
            [nan, nan, nan, ..., nan, nan, nan]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],

            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan]]], dtype=object)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>trend</span></div><div class='xr-var-dims'>(latitude, longitude, month)</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>nan nan nan nan ... nan nan nan nan</div><input id='attrs-859fb994-1f09-4152-b417-8d8c22d71e14' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-859fb994-1f09-4152-b417-8d8c22d71e14' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-a8fed96d-85fd-495a-9ea3-88bc83a8986b' class='xr-var-data-in' type='checkbox'><label for='data-a8fed96d-85fd-495a-9ea3-88bc83a8986b' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([[[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],

            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],

            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,

             &#x27;no trend&#x27;, &#x27;no trend&#x27;],
            [&#x27;no trend&#x27;, &#x27;no trend&#x27;, &#x27;no trend&#x27;, ..., &#x27;no trend&#x27;,
             &#x27;no trend&#x27;, &#x27;no trend&#x27;]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,
            [nan, nan, nan, ..., nan, nan, nan],
            [&#x27;no trend&#x27;, &#x27;no trend&#x27;, &#x27;no trend&#x27;, ..., &#x27;no trend&#x27;,
             &#x27;no trend&#x27;, &#x27;no trend&#x27;],
            [nan, nan, nan, ..., nan, nan, nan]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan]]], dtype=object)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>p</span></div><div class='xr-var-dims'>(latitude, longitude, month)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>nan nan nan nan ... nan nan nan nan</div><input id='attrs-875fa28b-f5c5-4436-8d9e-0d301dfe8dbd' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-875fa28b-f5c5-4436-8d9e-0d301dfe8dbd' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-139b3830-8d0a-492f-bc94-463d4a8a2c3a' class='xr-var-data-in' type='checkbox'><label for='data-139b3830-8d0a-492f-bc94-463d4a8a2c3a' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([[[       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
            ...,
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan]],

           [[       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
    ...
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
            [0.87113149, 0.25614497, 0.18344723, ..., 0.92246207,
             0.2057541 , 0.34676418],
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan]],

           [[       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
            ...,
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan],
            [       nan,        nan,        nan, ...,        nan,
                    nan,        nan]]])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>slope</span></div><div class='xr-var-dims'>(latitude, longitude, month)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>nan nan nan nan ... nan nan nan nan</div><input id='attrs-291954a4-4a6c-4e19-ada9-ccc0d68f3a70' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-291954a4-4a6c-4e19-ada9-ccc0d68f3a70' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-f5f419b5-1273-48f9-9549-34f08313ef53' class='xr-var-data-in' type='checkbox'><label for='data-f5f419b5-1273-48f9-9549-34f08313ef53' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([[[        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            ...,
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan]],

           [[        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
    ...
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [-0.03012464,  0.26125002,  0.15509539, ..., -0.00288407,
              0.09686013,  0.14403583],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan]],

           [[        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            ...,
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan],
            [        nan,         nan,         nan, ...,         nan,
                     nan,         nan]]])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>numeric_trend</span></div><div class='xr-var-dims'>(latitude, longitude, month)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>nan nan nan nan ... nan nan nan nan</div><input id='attrs-97eda24f-69a0-40ed-9f98-e0a070378e1d' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-97eda24f-69a0-40ed-9f98-e0a070378e1d' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-134fa715-cf16-424c-a16a-084a654f90a9' class='xr-var-data-in' type='checkbox'><label for='data-134fa715-cf16-424c-a16a-084a654f90a9' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([[[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,
    ...
            ...,
            [ 0.,  0.,  0., ...,  0.,  0.,  0.],
            [ 0.,  0.,  0., ...,  0.,  0.,  0.],
            [ 0.,  0.,  0., ...,  0.,  0.,  0.]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,
            [nan, nan, nan, ..., nan, nan, nan],
            [ 0.,  0.,  0., ...,  0.,  0.,  0.],
            [nan, nan, nan, ..., nan, nan, nan]],

           [[nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            ...,
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan],
            [nan, nan, nan, ..., nan, nan, nan]]])</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-6e362325-0d52-47fb-b9ec-6a38ef143419' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-6e362325-0d52-47fb-b9ec-6a38ef143419' class='xr-section-summary'  title='Expand/collapse section'>Attributes: <span>(0)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'></dl></div></li></ul></div></div>


Plotting the trend analysis results
-----------------------------------

Plotting slopes
~~~~~~~~~~~~~~~

Finally, we can plot our results.

First, let us take a look at the spatial and temporal pattern of the
``slope`` parameter. This will give an idea where and when temperature
has changed the most. The change will be represented in °C/year.

Before plotting, let us create a list of months for the iteration.

.. code:: ipython3

    # List of months used for iterating
    months = temp_array['month']
    display(months)



.. raw:: html

    <div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
    <defs>
    <symbol id="icon-database" viewBox="0 0 32 32">
    <path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
    <path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
    <path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
    </symbol>
    <symbol id="icon-file-text2" viewBox="0 0 32 32">
    <path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
    <path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    <path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    <path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    </symbol>
    </defs>
    </svg>
    <style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
     *
     */

    :root {
      --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
      --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
      --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
      --xr-border-color: var(--jp-border-color2, #e0e0e0);
      --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
      --xr-background-color: var(--jp-layout-color0, white);
      --xr-background-color-row-even: var(--jp-layout-color1, white);
      --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
    }

    html[theme=dark],
    body.vscode-dark {
      --xr-font-color0: rgba(255, 255, 255, 1);
      --xr-font-color2: rgba(255, 255, 255, 0.54);
      --xr-font-color3: rgba(255, 255, 255, 0.38);
      --xr-border-color: #1F1F1F;
      --xr-disabled-color: #515151;
      --xr-background-color: #111111;
      --xr-background-color-row-even: #111111;
      --xr-background-color-row-odd: #313131;
    }

    .xr-wrap {
      display: block !important;
      min-width: 300px;
      max-width: 700px;
    }

    .xr-text-repr-fallback {
      /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
      display: none;
    }

    .xr-header {
      padding-top: 6px;
      padding-bottom: 6px;
      margin-bottom: 4px;
      border-bottom: solid 1px var(--xr-border-color);
    }

    .xr-header > div,
    .xr-header > ul {
      display: inline;
      margin-top: 0;
      margin-bottom: 0;
    }

    .xr-obj-type,
    .xr-array-name {
      margin-left: 2px;
      margin-right: 10px;
    }

    .xr-obj-type {
      color: var(--xr-font-color2);
    }

    .xr-sections {
      padding-left: 0 !important;
      display: grid;
      grid-template-columns: 150px auto auto 1fr 20px 20px;
    }

    .xr-section-item {
      display: contents;
    }

    .xr-section-item input {
      display: none;
    }

    .xr-section-item input + label {
      color: var(--xr-disabled-color);
    }

    .xr-section-item input:enabled + label {
      cursor: pointer;
      color: var(--xr-font-color2);
    }

    .xr-section-item input:enabled + label:hover {
      color: var(--xr-font-color0);
    }

    .xr-section-summary {
      grid-column: 1;
      color: var(--xr-font-color2);
      font-weight: 500;
    }

    .xr-section-summary > span {
      display: inline-block;
      padding-left: 0.5em;
    }

    .xr-section-summary-in:disabled + label {
      color: var(--xr-font-color2);
    }

    .xr-section-summary-in + label:before {
      display: inline-block;
      content: '►';
      font-size: 11px;
      width: 15px;
      text-align: center;
    }

    .xr-section-summary-in:disabled + label:before {
      color: var(--xr-disabled-color);
    }

    .xr-section-summary-in:checked + label:before {
      content: '▼';
    }

    .xr-section-summary-in:checked + label > span {
      display: none;
    }

    .xr-section-summary,
    .xr-section-inline-details {
      padding-top: 4px;
      padding-bottom: 4px;
    }

    .xr-section-inline-details {
      grid-column: 2 / -1;
    }

    .xr-section-details {
      display: none;
      grid-column: 1 / -1;
      margin-bottom: 5px;
    }

    .xr-section-summary-in:checked ~ .xr-section-details {
      display: contents;
    }

    .xr-array-wrap {
      grid-column: 1 / -1;
      display: grid;
      grid-template-columns: 20px auto;
    }

    .xr-array-wrap > label {
      grid-column: 1;
      vertical-align: top;
    }

    .xr-preview {
      color: var(--xr-font-color3);
    }

    .xr-array-preview,
    .xr-array-data {
      padding: 0 5px !important;
      grid-column: 2;
    }

    .xr-array-data,
    .xr-array-in:checked ~ .xr-array-preview {
      display: none;
    }

    .xr-array-in:checked ~ .xr-array-data,
    .xr-array-preview {
      display: inline-block;
    }

    .xr-dim-list {
      display: inline-block !important;
      list-style: none;
      padding: 0 !important;
      margin: 0;
    }

    .xr-dim-list li {
      display: inline-block;
      padding: 0;
      margin: 0;
    }

    .xr-dim-list:before {
      content: '(';
    }

    .xr-dim-list:after {
      content: ')';
    }

    .xr-dim-list li:not(:last-child):after {
      content: ',';
      padding-right: 5px;
    }

    .xr-has-index {
      font-weight: bold;
    }

    .xr-var-list,
    .xr-var-item {
      display: contents;
    }

    .xr-var-item > div,
    .xr-var-item label,
    .xr-var-item > .xr-var-name span {
      background-color: var(--xr-background-color-row-even);
      margin-bottom: 0;
    }

    .xr-var-item > .xr-var-name:hover span {
      padding-right: 5px;
    }

    .xr-var-list > li:nth-child(odd) > div,
    .xr-var-list > li:nth-child(odd) > label,
    .xr-var-list > li:nth-child(odd) > .xr-var-name span {
      background-color: var(--xr-background-color-row-odd);
    }

    .xr-var-name {
      grid-column: 1;
    }

    .xr-var-dims {
      grid-column: 2;
    }

    .xr-var-dtype {
      grid-column: 3;
      text-align: right;
      color: var(--xr-font-color2);
    }

    .xr-var-preview {
      grid-column: 4;
    }

    .xr-var-name,
    .xr-var-dims,
    .xr-var-dtype,
    .xr-preview,
    .xr-attrs dt {
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      padding-right: 10px;
    }

    .xr-var-name:hover,
    .xr-var-dims:hover,
    .xr-var-dtype:hover,
    .xr-attrs dt:hover {
      overflow: visible;
      width: auto;
      z-index: 1;
    }

    .xr-var-attrs,
    .xr-var-data {
      display: none;
      background-color: var(--xr-background-color) !important;
      padding-bottom: 5px !important;
    }

    .xr-var-attrs-in:checked ~ .xr-var-attrs,
    .xr-var-data-in:checked ~ .xr-var-data {
      display: block;
    }

    .xr-var-data > table {
      float: right;
    }

    .xr-var-name span,
    .xr-var-data,
    .xr-attrs {
      padding-left: 25px !important;
    }

    .xr-attrs,
    .xr-var-attrs,
    .xr-var-data {
      grid-column: 1 / -1;
    }

    dl.xr-attrs {
      padding: 0;
      margin: 0;
      display: grid;
      grid-template-columns: 125px auto;
    }

    .xr-attrs dt,
    .xr-attrs dd {
      padding: 0;
      margin: 0;
      float: left;
      padding-right: 10px;
      width: auto;
    }

    .xr-attrs dt {
      font-weight: normal;
      grid-column: 1;
    }

    .xr-attrs dt:hover span {
      display: inline-block;
      background: var(--xr-background-color);
      padding-right: 10px;
    }

    .xr-attrs dd {
      grid-column: 2;
      white-space: pre-wrap;
      word-break: break-all;
    }

    .xr-icon-database,
    .xr-icon-file-text2 {
      display: inline-block;
      vertical-align: middle;
      width: 1em;
      height: 1.5em !important;
      stroke-width: 0;
      stroke: currentColor;
      fill: currentColor;
    }
    </style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray &#x27;month&#x27; (month: 12)&gt;
    array([ 1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12], dtype=int64)
    Coordinates:
      * month        (month) int64 1 2 3 4 5 6 7 8 9 10 11 12
        spatial_ref  int32 0</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'>'month'</div><ul class='xr-dim-list'><li><span class='xr-has-index'>month</span>: 12</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-65443dd7-7ddb-40d8-9d16-c07f7cb9fa8f' class='xr-array-in' type='checkbox' checked><label for='section-65443dd7-7ddb-40d8-9d16-c07f7cb9fa8f' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>1 2 3 4 5 6 7 8 9 10 11 12</span></div><div class='xr-array-data'><pre>array([ 1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12], dtype=int64)</pre></div></div></li><li class='xr-section-item'><input id='section-8eb0c0d8-60c0-4850-86dd-0ab99f0b7f41' class='xr-section-summary-in' type='checkbox'  checked><label for='section-8eb0c0d8-60c0-4850-86dd-0ab99f0b7f41' class='xr-section-summary' >Coordinates: <span>(2)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>month</span></div><div class='xr-var-dims'>(month)</div><div class='xr-var-dtype'>int64</div><div class='xr-var-preview xr-preview'>1 2 3 4 5 6 7 8 9 10 11 12</div><input id='attrs-f80f95d9-49d7-48f1-acea-7e34fbf31fe0' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-f80f95d9-49d7-48f1-acea-7e34fbf31fe0' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-583d31a7-1b54-4b72-81c1-5e8d1d61f537' class='xr-var-data-in' type='checkbox'><label for='data-583d31a7-1b54-4b72-81c1-5e8d1d61f537' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([ 1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12], dtype=int64)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>spatial_ref</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>int32</div><div class='xr-var-preview xr-preview'>0</div><input id='attrs-411f91f7-391f-4aa1-a1b2-2ff7e4cd953f' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-411f91f7-391f-4aa1-a1b2-2ff7e4cd953f' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-4d48521a-8324-4b07-8b79-160d55b6ada4' class='xr-var-data-in' type='checkbox'><label for='data-4d48521a-8324-4b07-8b79-160d55b6ada4' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>crs_wkt :</span></dt><dd>GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AXIS[&quot;Latitude&quot;,NORTH],AXIS[&quot;Longitude&quot;,EAST],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]]</dd><dt><span>semi_major_axis :</span></dt><dd>6378137.0</dd><dt><span>semi_minor_axis :</span></dt><dd>6356752.314245179</dd><dt><span>inverse_flattening :</span></dt><dd>298.257223563</dd><dt><span>reference_ellipsoid_name :</span></dt><dd>WGS 84</dd><dt><span>longitude_of_prime_meridian :</span></dt><dd>0.0</dd><dt><span>prime_meridian_name :</span></dt><dd>Greenwich</dd><dt><span>geographic_crs_name :</span></dt><dd>WGS 84</dd><dt><span>grid_mapping_name :</span></dt><dd>latitude_longitude</dd><dt><span>spatial_ref :</span></dt><dd>GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AXIS[&quot;Latitude&quot;,NORTH],AXIS[&quot;Longitude&quot;,EAST],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]]</dd></dl></div><div class='xr-var-data'><pre>array(0)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-6acd64c3-2d7f-4e38-94c5-ae89750f7d27' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-6acd64c3-2d7f-4e38-94c5-ae89750f7d27' class='xr-section-summary'  title='Expand/collapse section'>Attributes: <span>(0)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'></dl></div></li></ul></div></div>


For plotting monthly slope values indicating the intensity of
temperature change, we will create a Matplotlib figure object ``fig``
consisting of a :math:`4\times3` grid of axes.

While iterating over each of the 12 months and their corresponding axes
we \* will select the corresponding dataset slice from ``temp_array``
using the ``sel`` command \* plot the slope values for that particular
month \* have a colorbar next to the subplots showing the magnitude of
temperature change

.. code:: ipython3

    # Degree symbol for colorbar
    degree_symbol = '$\degree$'

    # Create figure and iterate over months to plot slope values in cells
    fig, axes = plt.subplots(4, 3, figsize=(12, 10))
    for month, ax in zip(months, axes.flatten()):
        ds = temp_array.sel(month=month) # select the month
        ds['slope'].plot(ax=ax, cmap='RdBu_r', cbar_kwargs={'label': '{}C/year'.format(degree_symbol)}) # plot slope values
        month_name = calendar.month_name[int(month)] # get month name
        ax.set_title(month_name)

    # Add the title
    fig_title = 'Median annual temperature change in the Baltics during 2001-2020'
    fig.suptitle(fig_title, y=1, size=14)

    # Reduce whitespace around the plot
    fig.tight_layout()



.. image:: workshop_files/workshop_54_0.png


For setting the range of the colorbar, we need to know the full range of
slope values in the data by checking the minimum and maximum values.

.. code:: ipython3

    display(temp_array['slope'].min())
    display(temp_array['slope'].max())



.. raw:: html

    <div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
    <defs>
    <symbol id="icon-database" viewBox="0 0 32 32">
    <path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
    <path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
    <path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
    </symbol>
    <symbol id="icon-file-text2" viewBox="0 0 32 32">
    <path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
    <path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    <path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    <path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    </symbol>
    </defs>
    </svg>
    <style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
     *
     */

    :root {
      --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
      --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
      --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
      --xr-border-color: var(--jp-border-color2, #e0e0e0);
      --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
      --xr-background-color: var(--jp-layout-color0, white);
      --xr-background-color-row-even: var(--jp-layout-color1, white);
      --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
    }

    html[theme=dark],
    body.vscode-dark {
      --xr-font-color0: rgba(255, 255, 255, 1);
      --xr-font-color2: rgba(255, 255, 255, 0.54);
      --xr-font-color3: rgba(255, 255, 255, 0.38);
      --xr-border-color: #1F1F1F;
      --xr-disabled-color: #515151;
      --xr-background-color: #111111;
      --xr-background-color-row-even: #111111;
      --xr-background-color-row-odd: #313131;
    }

    .xr-wrap {
      display: block !important;
      min-width: 300px;
      max-width: 700px;
    }

    .xr-text-repr-fallback {
      /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
      display: none;
    }

    .xr-header {
      padding-top: 6px;
      padding-bottom: 6px;
      margin-bottom: 4px;
      border-bottom: solid 1px var(--xr-border-color);
    }

    .xr-header > div,
    .xr-header > ul {
      display: inline;
      margin-top: 0;
      margin-bottom: 0;
    }

    .xr-obj-type,
    .xr-array-name {
      margin-left: 2px;
      margin-right: 10px;
    }

    .xr-obj-type {
      color: var(--xr-font-color2);
    }

    .xr-sections {
      padding-left: 0 !important;
      display: grid;
      grid-template-columns: 150px auto auto 1fr 20px 20px;
    }

    .xr-section-item {
      display: contents;
    }

    .xr-section-item input {
      display: none;
    }

    .xr-section-item input + label {
      color: var(--xr-disabled-color);
    }

    .xr-section-item input:enabled + label {
      cursor: pointer;
      color: var(--xr-font-color2);
    }

    .xr-section-item input:enabled + label:hover {
      color: var(--xr-font-color0);
    }

    .xr-section-summary {
      grid-column: 1;
      color: var(--xr-font-color2);
      font-weight: 500;
    }

    .xr-section-summary > span {
      display: inline-block;
      padding-left: 0.5em;
    }

    .xr-section-summary-in:disabled + label {
      color: var(--xr-font-color2);
    }

    .xr-section-summary-in + label:before {
      display: inline-block;
      content: '►';
      font-size: 11px;
      width: 15px;
      text-align: center;
    }

    .xr-section-summary-in:disabled + label:before {
      color: var(--xr-disabled-color);
    }

    .xr-section-summary-in:checked + label:before {
      content: '▼';
    }

    .xr-section-summary-in:checked + label > span {
      display: none;
    }

    .xr-section-summary,
    .xr-section-inline-details {
      padding-top: 4px;
      padding-bottom: 4px;
    }

    .xr-section-inline-details {
      grid-column: 2 / -1;
    }

    .xr-section-details {
      display: none;
      grid-column: 1 / -1;
      margin-bottom: 5px;
    }

    .xr-section-summary-in:checked ~ .xr-section-details {
      display: contents;
    }

    .xr-array-wrap {
      grid-column: 1 / -1;
      display: grid;
      grid-template-columns: 20px auto;
    }

    .xr-array-wrap > label {
      grid-column: 1;
      vertical-align: top;
    }

    .xr-preview {
      color: var(--xr-font-color3);
    }

    .xr-array-preview,
    .xr-array-data {
      padding: 0 5px !important;
      grid-column: 2;
    }

    .xr-array-data,
    .xr-array-in:checked ~ .xr-array-preview {
      display: none;
    }

    .xr-array-in:checked ~ .xr-array-data,
    .xr-array-preview {
      display: inline-block;
    }

    .xr-dim-list {
      display: inline-block !important;
      list-style: none;
      padding: 0 !important;
      margin: 0;
    }

    .xr-dim-list li {
      display: inline-block;
      padding: 0;
      margin: 0;
    }

    .xr-dim-list:before {
      content: '(';
    }

    .xr-dim-list:after {
      content: ')';
    }

    .xr-dim-list li:not(:last-child):after {
      content: ',';
      padding-right: 5px;
    }

    .xr-has-index {
      font-weight: bold;
    }

    .xr-var-list,
    .xr-var-item {
      display: contents;
    }

    .xr-var-item > div,
    .xr-var-item label,
    .xr-var-item > .xr-var-name span {
      background-color: var(--xr-background-color-row-even);
      margin-bottom: 0;
    }

    .xr-var-item > .xr-var-name:hover span {
      padding-right: 5px;
    }

    .xr-var-list > li:nth-child(odd) > div,
    .xr-var-list > li:nth-child(odd) > label,
    .xr-var-list > li:nth-child(odd) > .xr-var-name span {
      background-color: var(--xr-background-color-row-odd);
    }

    .xr-var-name {
      grid-column: 1;
    }

    .xr-var-dims {
      grid-column: 2;
    }

    .xr-var-dtype {
      grid-column: 3;
      text-align: right;
      color: var(--xr-font-color2);
    }

    .xr-var-preview {
      grid-column: 4;
    }

    .xr-var-name,
    .xr-var-dims,
    .xr-var-dtype,
    .xr-preview,
    .xr-attrs dt {
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      padding-right: 10px;
    }

    .xr-var-name:hover,
    .xr-var-dims:hover,
    .xr-var-dtype:hover,
    .xr-attrs dt:hover {
      overflow: visible;
      width: auto;
      z-index: 1;
    }

    .xr-var-attrs,
    .xr-var-data {
      display: none;
      background-color: var(--xr-background-color) !important;
      padding-bottom: 5px !important;
    }

    .xr-var-attrs-in:checked ~ .xr-var-attrs,
    .xr-var-data-in:checked ~ .xr-var-data {
      display: block;
    }

    .xr-var-data > table {
      float: right;
    }

    .xr-var-name span,
    .xr-var-data,
    .xr-attrs {
      padding-left: 25px !important;
    }

    .xr-attrs,
    .xr-var-attrs,
    .xr-var-data {
      grid-column: 1 / -1;
    }

    dl.xr-attrs {
      padding: 0;
      margin: 0;
      display: grid;
      grid-template-columns: 125px auto;
    }

    .xr-attrs dt,
    .xr-attrs dd {
      padding: 0;
      margin: 0;
      float: left;
      padding-right: 10px;
      width: auto;
    }

    .xr-attrs dt {
      font-weight: normal;
      grid-column: 1;
    }

    .xr-attrs dt:hover span {
      display: inline-block;
      background: var(--xr-background-color);
      padding-right: 10px;
    }

    .xr-attrs dd {
      grid-column: 2;
      white-space: pre-wrap;
      word-break: break-all;
    }

    .xr-icon-database,
    .xr-icon-file-text2 {
      display: inline-block;
      vertical-align: middle;
      width: 1em;
      height: 1.5em !important;
      stroke-width: 0;
      stroke: currentColor;
      fill: currentColor;
    }
    </style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray &#x27;slope&#x27; ()&gt;
    array(-0.17666797)
    Coordinates:
        spatial_ref  int32 0</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'>'slope'</div></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-0d52d268-48c1-41c5-b4fc-3dd7c6317436' class='xr-array-in' type='checkbox' checked><label for='section-0d52d268-48c1-41c5-b4fc-3dd7c6317436' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>-0.1767</span></div><div class='xr-array-data'><pre>array(-0.17666797)</pre></div></div></li><li class='xr-section-item'><input id='section-10e3b977-3d99-4e4d-8daa-63dbd9fa1421' class='xr-section-summary-in' type='checkbox'  checked><label for='section-10e3b977-3d99-4e4d-8daa-63dbd9fa1421' class='xr-section-summary' >Coordinates: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>spatial_ref</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>int32</div><div class='xr-var-preview xr-preview'>0</div><input id='attrs-e97cc16d-e99d-4279-b01e-ea3a801c6b61' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-e97cc16d-e99d-4279-b01e-ea3a801c6b61' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-e1e2d8cb-5458-48eb-a1ce-d5e249317825' class='xr-var-data-in' type='checkbox'><label for='data-e1e2d8cb-5458-48eb-a1ce-d5e249317825' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>crs_wkt :</span></dt><dd>GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AXIS[&quot;Latitude&quot;,NORTH],AXIS[&quot;Longitude&quot;,EAST],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]]</dd><dt><span>semi_major_axis :</span></dt><dd>6378137.0</dd><dt><span>semi_minor_axis :</span></dt><dd>6356752.314245179</dd><dt><span>inverse_flattening :</span></dt><dd>298.257223563</dd><dt><span>reference_ellipsoid_name :</span></dt><dd>WGS 84</dd><dt><span>longitude_of_prime_meridian :</span></dt><dd>0.0</dd><dt><span>prime_meridian_name :</span></dt><dd>Greenwich</dd><dt><span>geographic_crs_name :</span></dt><dd>WGS 84</dd><dt><span>grid_mapping_name :</span></dt><dd>latitude_longitude</dd><dt><span>spatial_ref :</span></dt><dd>GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AXIS[&quot;Latitude&quot;,NORTH],AXIS[&quot;Longitude&quot;,EAST],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]]</dd></dl></div><div class='xr-var-data'><pre>array(0)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-242482c2-0811-4706-bf8b-0148c0575021' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-242482c2-0811-4706-bf8b-0148c0575021' class='xr-section-summary'  title='Expand/collapse section'>Attributes: <span>(0)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'></dl></div></li></ul></div></div>



.. raw:: html

    <div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
    <defs>
    <symbol id="icon-database" viewBox="0 0 32 32">
    <path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
    <path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
    <path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
    </symbol>
    <symbol id="icon-file-text2" viewBox="0 0 32 32">
    <path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
    <path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    <path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    <path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
    </symbol>
    </defs>
    </svg>
    <style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
     *
     */

    :root {
      --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
      --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
      --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
      --xr-border-color: var(--jp-border-color2, #e0e0e0);
      --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
      --xr-background-color: var(--jp-layout-color0, white);
      --xr-background-color-row-even: var(--jp-layout-color1, white);
      --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
    }

    html[theme=dark],
    body.vscode-dark {
      --xr-font-color0: rgba(255, 255, 255, 1);
      --xr-font-color2: rgba(255, 255, 255, 0.54);
      --xr-font-color3: rgba(255, 255, 255, 0.38);
      --xr-border-color: #1F1F1F;
      --xr-disabled-color: #515151;
      --xr-background-color: #111111;
      --xr-background-color-row-even: #111111;
      --xr-background-color-row-odd: #313131;
    }

    .xr-wrap {
      display: block !important;
      min-width: 300px;
      max-width: 700px;
    }

    .xr-text-repr-fallback {
      /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
      display: none;
    }

    .xr-header {
      padding-top: 6px;
      padding-bottom: 6px;
      margin-bottom: 4px;
      border-bottom: solid 1px var(--xr-border-color);
    }

    .xr-header > div,
    .xr-header > ul {
      display: inline;
      margin-top: 0;
      margin-bottom: 0;
    }

    .xr-obj-type,
    .xr-array-name {
      margin-left: 2px;
      margin-right: 10px;
    }

    .xr-obj-type {
      color: var(--xr-font-color2);
    }

    .xr-sections {
      padding-left: 0 !important;
      display: grid;
      grid-template-columns: 150px auto auto 1fr 20px 20px;
    }

    .xr-section-item {
      display: contents;
    }

    .xr-section-item input {
      display: none;
    }

    .xr-section-item input + label {
      color: var(--xr-disabled-color);
    }

    .xr-section-item input:enabled + label {
      cursor: pointer;
      color: var(--xr-font-color2);
    }

    .xr-section-item input:enabled + label:hover {
      color: var(--xr-font-color0);
    }

    .xr-section-summary {
      grid-column: 1;
      color: var(--xr-font-color2);
      font-weight: 500;
    }

    .xr-section-summary > span {
      display: inline-block;
      padding-left: 0.5em;
    }

    .xr-section-summary-in:disabled + label {
      color: var(--xr-font-color2);
    }

    .xr-section-summary-in + label:before {
      display: inline-block;
      content: '►';
      font-size: 11px;
      width: 15px;
      text-align: center;
    }

    .xr-section-summary-in:disabled + label:before {
      color: var(--xr-disabled-color);
    }

    .xr-section-summary-in:checked + label:before {
      content: '▼';
    }

    .xr-section-summary-in:checked + label > span {
      display: none;
    }

    .xr-section-summary,
    .xr-section-inline-details {
      padding-top: 4px;
      padding-bottom: 4px;
    }

    .xr-section-inline-details {
      grid-column: 2 / -1;
    }

    .xr-section-details {
      display: none;
      grid-column: 1 / -1;
      margin-bottom: 5px;
    }

    .xr-section-summary-in:checked ~ .xr-section-details {
      display: contents;
    }

    .xr-array-wrap {
      grid-column: 1 / -1;
      display: grid;
      grid-template-columns: 20px auto;
    }

    .xr-array-wrap > label {
      grid-column: 1;
      vertical-align: top;
    }

    .xr-preview {
      color: var(--xr-font-color3);
    }

    .xr-array-preview,
    .xr-array-data {
      padding: 0 5px !important;
      grid-column: 2;
    }

    .xr-array-data,
    .xr-array-in:checked ~ .xr-array-preview {
      display: none;
    }

    .xr-array-in:checked ~ .xr-array-data,
    .xr-array-preview {
      display: inline-block;
    }

    .xr-dim-list {
      display: inline-block !important;
      list-style: none;
      padding: 0 !important;
      margin: 0;
    }

    .xr-dim-list li {
      display: inline-block;
      padding: 0;
      margin: 0;
    }

    .xr-dim-list:before {
      content: '(';
    }

    .xr-dim-list:after {
      content: ')';
    }

    .xr-dim-list li:not(:last-child):after {
      content: ',';
      padding-right: 5px;
    }

    .xr-has-index {
      font-weight: bold;
    }

    .xr-var-list,
    .xr-var-item {
      display: contents;
    }

    .xr-var-item > div,
    .xr-var-item label,
    .xr-var-item > .xr-var-name span {
      background-color: var(--xr-background-color-row-even);
      margin-bottom: 0;
    }

    .xr-var-item > .xr-var-name:hover span {
      padding-right: 5px;
    }

    .xr-var-list > li:nth-child(odd) > div,
    .xr-var-list > li:nth-child(odd) > label,
    .xr-var-list > li:nth-child(odd) > .xr-var-name span {
      background-color: var(--xr-background-color-row-odd);
    }

    .xr-var-name {
      grid-column: 1;
    }

    .xr-var-dims {
      grid-column: 2;
    }

    .xr-var-dtype {
      grid-column: 3;
      text-align: right;
      color: var(--xr-font-color2);
    }

    .xr-var-preview {
      grid-column: 4;
    }

    .xr-var-name,
    .xr-var-dims,
    .xr-var-dtype,
    .xr-preview,
    .xr-attrs dt {
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      padding-right: 10px;
    }

    .xr-var-name:hover,
    .xr-var-dims:hover,
    .xr-var-dtype:hover,
    .xr-attrs dt:hover {
      overflow: visible;
      width: auto;
      z-index: 1;
    }

    .xr-var-attrs,
    .xr-var-data {
      display: none;
      background-color: var(--xr-background-color) !important;
      padding-bottom: 5px !important;
    }

    .xr-var-attrs-in:checked ~ .xr-var-attrs,
    .xr-var-data-in:checked ~ .xr-var-data {
      display: block;
    }

    .xr-var-data > table {
      float: right;
    }

    .xr-var-name span,
    .xr-var-data,
    .xr-attrs {
      padding-left: 25px !important;
    }

    .xr-attrs,
    .xr-var-attrs,
    .xr-var-data {
      grid-column: 1 / -1;
    }

    dl.xr-attrs {
      padding: 0;
      margin: 0;
      display: grid;
      grid-template-columns: 125px auto;
    }

    .xr-attrs dt,
    .xr-attrs dd {
      padding: 0;
      margin: 0;
      float: left;
      padding-right: 10px;
      width: auto;
    }

    .xr-attrs dt {
      font-weight: normal;
      grid-column: 1;
    }

    .xr-attrs dt:hover span {
      display: inline-block;
      background: var(--xr-background-color);
      padding-right: 10px;
    }

    .xr-attrs dd {
      grid-column: 2;
      white-space: pre-wrap;
      word-break: break-all;
    }

    .xr-icon-database,
    .xr-icon-file-text2 {
      display: inline-block;
      vertical-align: middle;
      width: 1em;
      height: 1.5em !important;
      stroke-width: 0;
      stroke: currentColor;
      fill: currentColor;
    }
    </style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray &#x27;slope&#x27; ()&gt;
    array(0.30160443)
    Coordinates:
        spatial_ref  int32 0</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-array-name'>'slope'</div></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-1c2e7b2a-dd27-4a83-b417-70e36682cb57' class='xr-array-in' type='checkbox' checked><label for='section-1c2e7b2a-dd27-4a83-b417-70e36682cb57' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>0.3016</span></div><div class='xr-array-data'><pre>array(0.30160443)</pre></div></div></li><li class='xr-section-item'><input id='section-c4910541-5f7c-44ce-876d-e491da964a7c' class='xr-section-summary-in' type='checkbox'  checked><label for='section-c4910541-5f7c-44ce-876d-e491da964a7c' class='xr-section-summary' >Coordinates: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>spatial_ref</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>int32</div><div class='xr-var-preview xr-preview'>0</div><input id='attrs-798267d4-1aec-4c90-b561-f8e51fb9664f' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-798267d4-1aec-4c90-b561-f8e51fb9664f' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-c64f7f97-af02-4165-8f8d-3d1d947bea42' class='xr-var-data-in' type='checkbox'><label for='data-c64f7f97-af02-4165-8f8d-3d1d947bea42' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>crs_wkt :</span></dt><dd>GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AXIS[&quot;Latitude&quot;,NORTH],AXIS[&quot;Longitude&quot;,EAST],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]]</dd><dt><span>semi_major_axis :</span></dt><dd>6378137.0</dd><dt><span>semi_minor_axis :</span></dt><dd>6356752.314245179</dd><dt><span>inverse_flattening :</span></dt><dd>298.257223563</dd><dt><span>reference_ellipsoid_name :</span></dt><dd>WGS 84</dd><dt><span>longitude_of_prime_meridian :</span></dt><dd>0.0</dd><dt><span>prime_meridian_name :</span></dt><dd>Greenwich</dd><dt><span>geographic_crs_name :</span></dt><dd>WGS 84</dd><dt><span>grid_mapping_name :</span></dt><dd>latitude_longitude</dd><dt><span>spatial_ref :</span></dt><dd>GEOGCS[&quot;WGS 84&quot;,DATUM[&quot;WGS_1984&quot;,SPHEROID[&quot;WGS 84&quot;,6378137,298.257223563,AUTHORITY[&quot;EPSG&quot;,&quot;7030&quot;]],AUTHORITY[&quot;EPSG&quot;,&quot;6326&quot;]],PRIMEM[&quot;Greenwich&quot;,0,AUTHORITY[&quot;EPSG&quot;,&quot;8901&quot;]],UNIT[&quot;degree&quot;,0.0174532925199433,AUTHORITY[&quot;EPSG&quot;,&quot;9122&quot;]],AXIS[&quot;Latitude&quot;,NORTH],AXIS[&quot;Longitude&quot;,EAST],AUTHORITY[&quot;EPSG&quot;,&quot;4326&quot;]]</dd></dl></div><div class='xr-var-data'><pre>array(0)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-42e11d94-e087-4fcc-a67a-8324221df531' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-42e11d94-e087-4fcc-a67a-8324221df531' class='xr-section-summary'  title='Expand/collapse section'>Attributes: <span>(0)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'></dl></div></li></ul></div></div>


We will create a colorbar normalization object called ``divnorm`` to set
the range. Using the ``TwoSlopeNorm`` command and setting the parameters
``vmin=-0.2`` and ``vmax=0.4`` should cover the range of slope values.
As our colorbar is diverging, we also need to set ``vcenter=0``.

.. code:: ipython3

    divnorm = mcolors.TwoSlopeNorm(vmin=-0.2, vcenter=0, vmax=0.4)

We can now recreate the plot with our custom colorbar range by
specifying the ``norm`` parameter.

.. code:: ipython3

    # Degree symbol for colorbar
    degree_symbol = '$\degree$'

    # Create figure and iterate over months to plot slope values in cells
    fig, axes = plt.subplots(4, 3, figsize=(12, 10))
    for month, ax in zip(months, axes.flatten()):
        ds = temp_array.sel(month=month) # select the month
        ds['slope'].plot(ax=ax, cmap='RdBu_r', norm=divnorm, cbar_kwargs={'label': '{}C/year'.format(degree_symbol)}) # plot slope values with custom colorbar range
        month_name = calendar.month_name[int(month)] # get month name
        ax.set_title(month_name)

    # Add the title
    fig_title = 'Median annual temperature change in the Baltics during 2001-2020'
    fig.suptitle(fig_title, y=1, size=14)

    # Reduce whitespace around the plot
    fig.tight_layout()



.. image:: workshop_files/workshop_60_0.png


From the plots we can see that majority of the months show an increasing
trend, which is the strongest in February and December. Although parts
of the Baltics show a slight decreasing trend in other months (e.g. the
western part in January), the only month with a clear decreasing trend
is July. No clear spatial pattern exists in the detected significant
trend locations.

Plotting significant trends
~~~~~~~~~~~~~~~~~~~~~~~~~~~

As we examined above when printing out detected significant trend counts
per month, only some grid cells indicated significant trends in the
study period. We could also see that the three possible trend
classes—decreasing, no trend and increasing— are not present in the case
of some months.

To see whether the temperature changes seen in the previous plot were
also statistically significant, we will now plot the ``numeric_trend``
variable. We will use the three numeric trend classes to set the
colorbar. To avoid getting unnecessary ticks in the colorbar, we need to
specify the corresponding parameter in ``cbar_kwargs``.

.. code:: ipython3

    # Set colorbar range
    divnorm = mcolors.TwoSlopeNorm(vmin=-1, vcenter=0, vmax=1)

    # Colormap
    colors = ['#A89355', '#C8C9CA', '#65C39E']
    cmap = matplotlib.colors.ListedColormap(colors)

    # Create figure and iterate over months to plot significance of trends in cells
    fig, axes = plt.subplots(4, 3, figsize=(12, 10))
    for month, ax in zip(months, axes.flatten()):
        ds = temp_array.sel(month=month) # select month
        ds['numeric_trend'].plot(ax=ax, norm=divnorm, cmap=cmap, cbar_kwargs={'ticks': [-1, 0, 1], 'label': '{}C/year'.format(degree_symbol)}) # plot trends
        month_name = calendar.month_name[int(month)] # get month name
        ax.set_title(month_name)

    # Add the title
    fig_title = 'Significance of temperature trends in the Baltics during 2001-2020'
    fig.suptitle(fig_title, y=1, size=14)

    # Reduce whitespace around the plot
    fig.tight_layout()



.. image:: workshop_files/workshop_64_0.png


Although there is no clear temporal or spatial pattern in the
significant trends detected, we can see that increasing trends were
mostly present in the southern half of the study area, i.e. Latvia and
Lithuania. In November, significant increasing trends also reached
southern Estonia. The only significant decreasing trends were detected
in July, particularly in the eastern part of Estonia.

Perhaps somewhat surprisingly, the winter months with the highest slope
values—February and December— did not show up as significant trends.

Automatic plotting
~~~~~~~~~~~~~~~~~~

We have now learned to plot the hard way by customizing the figures
manually. Xarray and Matplotlib can actually do certain things for us
when plotting, so that quick plots can be displayed with a single line
of code.

For this, we use the ``imshow`` command and specify the coordinates with
``longitude`` and ``latitude``. The ``col`` parameter make sure that we
get a plot for each month. Parameter ``col_wrap`` tells Matplotlib to
plot three months per row.

.. code:: ipython3

    # Plot monthly slope values automatically
    temp_array['slope'].plot.imshow('longitude', 'latitude', col='month', col_wrap=3)




.. parsed-literal::

    <xarray.plot.facetgrid.FacetGrid at 0x14acb42f448>




.. image:: workshop_files/workshop_68_1.png


References
----------

-  Atta-ur-Rahman, and Dawood, M. (2017). Spatio-statistical analysis of
   temperature fluctuation using Mann–Kendall and Sen’s slope approach.
   Clim. Dyn. 48, 783–797. doi:10.1007/s00382-016-3110-y.

-  Cornes, R. C., van der Schrier, G., van den Besselaar, E. J. M., and
   Jones, P. D. (2018). An Ensemble Version of the E-OBS Temperature and
   temperature Data Sets. J. Geophys. Res. Atmos. 123, 9391–9409.
   doi:10.1029/2017JD028200.

-  Jaagus, J., Sepp, M., Tamm, T., Järvet, A., and Mõisja, K. (2017).
   Trends and regime shifts in climatic conditions and river runoff in
   Estonia during 1951–2015. Earth Syst. Dyn. 8, 963–976.
   doi:10.5194/esd-8-963-2017.

-  Kendall, M. G. (1957). Rank Correlation Methods. Biometrika 44, 298.
   doi:10.2307/2333282.

-  Mann, H. B. (1945). Non-Parametric Test Against Trend. Econometrica
   13, 245–259. Available at:
   http://www.economist.com/node/18330371?story%7B_%7Did=18330371.

-  Sen Pranab Kumar (1968). Estimates of the Regression Coefficient
   Based on Kendall’s Tau. J. Am. Stat. Assoc. 63, 1379–1389.

-  Silva Junior, C. H. L., Almeida, C. T., Santos, J. R. N., Anderson,
   L. O., Aragão, L. E. O. C., and Silva, F. B. (2018). Spatiotemporal
   rainfall trends in the Brazilian legal Amazon between the years 1998
   and 2015. Water (Switzerland) 10, 1–16. doi:10.3390/w10091220.

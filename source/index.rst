.. EF5Manual documentation master file, created by
   sphinx-quickstart on Thu Feb 20 14:26:00 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

EF5 User Manual
=======================

Version 1.0 (July, 2016)

.. contents::
   :local:
   :depth: 2

About EF5
---------
EF5 is designed to facilitate creation of ensemble forecasts for flash flood prediction. It incorporates multi-model support while maintaining a single set of input data. Currently, the only supported model is the Coupled Routing and Excess Storage (CREST) hydrologic model. Additionally, EF5 is optimized for parallel computing—portions of CREST use OpenMP for multi-core processing.

Hydrologic Water Balance Models
-------------------------------
### CREST
The Coupled Routing and Excess Storage (CREST) distributed hydrological model is a hybrid strategy developed by the University of Oklahoma and the NASA SERVIR Project Team. CREST simulates the spatiotemporal variation of water and energy fluxes and storages on a user-defined grid. Its scalability is achieved through sub-grid scale representation of soil moisture storage (using a variable infiltration curve) and runoff generation (using linear reservoirs). Originally developed for global flood predictions, it is also suitable for basin-scale applications.

More detailed information about CREST can be found in the publication:  
`Wang, J., et al. (2011) <https://dx.doi.org/10.1080/02626667.2010.543087>`_

### SAC-SMA
The Sacramento Soil Moisture Accounting (SAC-SMA) Model was developed by the U.S. National Weather Service. Its purpose is to parameterize soil moisture characteristics so that applied moisture is logically distributed, percolation is realistically simulated, and streamflow is effectively modeled.

More detailed information about SAC-SMA is available at:  
`U.S. NWS SAC-SMA algorithm description <http://www.nws.noaa.gov/oh/hrl/nwsrfs/users_manual/part2/_pdf/23sacsma.pdf>`_

### HP
The Hydrophobic (HP) water balance model assumes an entirely impervious surface where all rainfall is transformed directly into surface runoff.

Routing Models
--------------
### Linear Reservoir
Adapted from the CREST implementation, the linear reservoir routing model uses two reservoirs—one for overland (surface) runoff and one for subsurface runoff.

### Kinematic Wave
Kinematic wave routing is a simplified approximation of the Barré de Saint-Venant equations. It assumes that gravitational and friction forces cancel and neglects acceleration terms.

Snow Melt Models
----------------
### Snow-17
Snow-17 is a temperature-index based snow melt module widely used by the U.S. NWS.

Inundation Models
-----------------
### Simple Inundation
The Simple Inundation model provides basic inundation modeling functionality.

Compiling EF5
-------------
EF5 uses the TIFF and GEOTIFF libraries for raster formats. Compilation can be performed in three steps:

1. **(Developer Only)** – If you are modifying the Makefile.am, run:

   .. code-block:: bash

      autoreconf --force --install

2. **Configure** – If you wish to install EF5 to a custom path, run:

   .. code-block:: bash

      ./configure --prefix=/path/to/installation

3. **Compile** – Build EF5 with:

   .. code-block:: bash

      make CXXFLAGS="-O3 -fopenmp"

Upon successful compilation, a binary named ``ef5`` will be created in the ``bin`` directory.

Configuration File
------------------
The configuration file controls all user-adjustable settings for EF5, including input forcings, output options, and run methods. It is generally case **insensitive** (except for file paths on case-sensitive systems). Three styles of comments are supported: bash (#), C (/* */), and C++ (//).

.. code-block:: ini

   # All variables (except file paths) are case insensitive
   // Multiple comment types are supported
   /*
      Including multi-line C-style comments
   */

### Basic Information
Specifies file locations for the digital elevation model (DEM), drainage direction map (DDM), and flow accumulation map (FAM).

.. code-block:: ini

   [Basic]
   DEM=/EF5Demo/FF/basic/DEM.asc
   DDM=/EF5Demo/FF/basic/DDM.asc
   FAM=/EF5Demo/FF/basic/FAM.asc
   PROJ=laea
   ESRIDDM=true
   SELFFAM=true

### Precipitation Information
Defines the properties of the precipitation forcing files.

.. code-block:: ini

   [PrecipForcing Q2Precip]
   TYPE=BIF
   UNIT=mm/h
   FREQ=5u
   LOC=/EF5Demo/FF/precip
   NAME=Q2_YYYYMMDDHHUU.bif

### Potential Evapotranspiration (PET) Information
Defines the PET forcing file details.

.. code-block:: ini

   [PETForcing PET]
   TYPE=BIF
   UNIT=mm/h
   FREQ=m
   LOC=/EF5Demo/FF/pet
   NAME=PET_MM.bif

### Gauge Locations
Specifies the locations of gauges for output and parameter assignment.

.. code-block:: ini

   [Gauge OKC]
   LON=-97.01
   LAT=35.68
   OBS=/EF5Demo/obs/okc.csv
   BASINAREA=341.88
   OUTPUTTS=TRUE

   [Gauge AR]
   LON=-93.62
   LAT=34.37

### Basins
Groups gauge locations into basins.

.. code-block:: ini

   [Basin FF]
   GAUGE=OKC
   GAUGE=AR

### Parameter Sets
Control the distributed model parameter settings. Parameters are specified per gauge.

#### CREST Parameter Set

.. code-block:: ini

   [CrestParamSet ABRFC]
   wm_grid=/path/to/wm.tif
   im_grid=/path/to/im.tif
   fc_grid=/path/to/ksat.tif
   b_grid=/path/to/b.tif
   gauge=03455500
   wm=1.00
   b=1.0
   im=0.01
   ke=1.0
   fc=1.00
   iwu=50.0

#### SAC-SMA Parameter Set

.. code-block:: ini

   [SacParamSet ABRFC]
   UZTWM_grid=/path/to/uztwm.tif
   UZFWM_grid=/path/to/uzfwm.tif
   UZK_grid=/path/to/uzk.tif
   ZPERC_grid=/path/to/zperc.tif
   REXP_grid=/path/to/rexp.tif
   LZTWM_grid=/path/to/lztwm.tif
   LZFSM_grid=/path/to/lzfsm.tif
   LZFPM_grid=/path/to/lzfpm.tif
   LZSK_grid=/path/to/lzsk.tif
   LZPK_grid=/path/to/lzpk.tif
   PFREE_grid=/path/to/pfree.tif
   gauge=01055000
   UZTWM=1.0
   UZFWM=1.0
   UZK=1.0
   PCTIM=0.101
   ADIMP=0.10
   RIVA=1.001
   ZPERC=1.0
   REXP=1.0
   LZTWM=1.0
   LZFSM=1.0
   LZFPM=1.0
   LZSK=1.0
   LZPK=1.0
   PFREE=1.0
   SIDE=0.0
   RSERV=0.3
   ADIMC=1.0
   UZTWC=0.55
   UZFWC=0.14
   LZTWC=0.56
   LZFSC=0.11
   LZFPC=0.46

#### HP Parameter Set
*To be completed in a future revision.*

#### Linear Reservoir Parameter Set

.. code-block:: ini

   [lrparamset rundu]
   gauge=rundu
   coem=1611.115479
   river=307.980042
   under=2531.556641
   leako=0.918236
   leaki=0.017568
   th=8.140809
   iso=0.000040
   isu=0.000073

#### Kinematic Wave Parameter Set

.. code-block:: ini

   [KWParamSet rundu]
   GAUGE=rundu
   UNDER=1.673110
   LEAKI=0.043105
   TH=6.658569
   ISU=0.000000
   ALPHA=2.991570
   BETA=0.932080
   ALPHA0=4.603945

#### Snow-17 Parameter Set

.. code-block:: ini

   [snow17paramset tarbela]
   GAUGE=tarbela
   UADJ=0.184653
   MBASE=0.047224
   MFMAX=1.068658
   MFMIN=0.516059
   TIPM=0.911706
   NMF=0.077336
   PLWHC=0.093812
   SCF=2.219492

#### Simple Inundation Parameter Set

.. code-block:: ini

   [simpleinundationparamset rundu]
   gauge=rundu
   alpha=2.991570
   beta=0.932080

Tasks
-----
Tasks define which model to run, the simulation period, timestep, and related settings.

.. code-block:: ini

   [Task RunFF]
   STYLE=SIMU
   MODEL=CREST
   BASIN=FF
   PRECIP=Q2_PRECIP
   PET=PET
   OUTPUT=/EF5Demo/FF/output/
   PARAM_SET=FF
   TIMESTEP=5u
   TIME_BEGIN=201006010000
   TIME_END=201006010030

* **STYLE**: Task style (e.g., SIMU, SIMU_RP, CALI_DREAM, etc.)
* **MODEL**: Water balance model (e.g., CREST, SAC, HyMOD)
* **BASIN**: Basin block name
* **PRECIP/PET**: Forcing file blocks
* **PARAM_SET**: Parameter set block name
* **TIMESTEP, TIME_BEGIN, TIME_END**: Simulation timing parameters

Execute Block
-------------
Specifies which task to execute when EF5 runs.

.. code-block:: ini

   [Execute]
   TASK=RunFF

Running EF5
-----------
To run EF5, execute the binary with an optional control file argument:

.. code-block:: bash

   ef5 [controlfile]

Calibrating the Models
----------------------

### Complete Sample Configuration File for Calibrating

.. code-block:: ini

   /*
    * This is an example configuration file for EF5
    */


   [Basic]
   DEM=data/basic/dem_KY.tif
   DDM=data/basic/ddm_KY.tif
   FAM=data/basic/fam_KY.tif
   PROJ=geographic
   ESRIDDM=true
   SelfFAM=false

   [PETForcing CLIMO]
   LOC=data/pet/
   FREQ=1m
   UNIT=mm/d
   NAME=PET_MM_KY.tif
   TYPE=TIF

   [PrecipForcing MRMS]
   TYPE=TIF
   UNIT=mm/h
   FREQ=2u
   LOC=data/precip/
   NAME=PrecipRate_00.00_YYYYMMDD-HHUU00.tif

   [Gauge 0]
   lat=36.9883
   lon=-89.1326
   outputts=true
   basinarea=421966.3

   [Gauge 03404900]
   lon=-84.093599999999995
   lat=36.951400000000000
   basinarea=139.341000000000008
   obs=data/observations/usgs/Streamflow_Time_Series_CMS_UTC_USGS_03404900.csv

   [Gauge 03282040]
   lon=-83.810299999999998
   lat=37.500599999999999
   basinarea=200.205999999999989
   obs=data/observations/usgs/Streamflow_Time_Series_CMS_UTC_USGS_03282040.csv

   [Gauge 03250190]
   lon=-83.831699999999998
   lat=38.023899999999998
   basinarea=218.854000000000013
   obs=data/observations/usgs/Streamflow_Time_Series_CMS_UTC_USGS_03250190.csv

   [Gauge 03208950]
   lon=-82.438900000000004
   lat=37.123899999999999
   basinarea=172.234000000000009
   obs=data/observations/usgs/Streamflow_Time_Series_CMS_UTC_USGS_03208950.csv

   [Gauge 03208500]
   lon=-82.295800000000000
   lat=37.206899999999997
   basinarea=740.736999999999966
   obs=data/observations/usgs/Streamflow_Time_Series_CMS_UTC_USGS_03208500.csv

   [Gauge 03206600]
   lon=-82.296099999999996
   lat=38.017200000000003
   basinarea=99.714500000000001
   obs=data/observations/usgs/Streamflow_Time_Series_CMS_UTC_USGS_03206600.csv

   [Gauge 03284525]
   lon=-84.411100000000005
   lat=37.988300000000002
   basinarea=2.486400000000000
   obs=data/observations/usgs/Streamflow_Time_Series_CMS_UTC_USGS_03284525.csv

   [Gauge 03478400]
   lon=-82.133899999999997
   lat=36.631700000000002
   basinarea=69.670699999999997
   obs=data/observations/usgs/Streamflow_Time_Series_CMS_UTC_USGS_03478400.csv

   [Basin 0]
   gauge=0
   gauge=03404900
   gauge=03282040
   gauge=03250190
   gauge=03208950
   gauge=03208500
   gauge=03206600
   gauge=03284525
   gauge=03478400

   [CrestParamSet EF5KY]
   wm_grid=data/parameters/CREST/wm_KY.tif
   im_grid=data/parameters/CREST/im_KY.tif
   fc_grid=data/parameters/CREST/ksat_KY.tif
   b_grid=data/parameters/CREST/b_KY.tif
   gauge=03282040
   wm=1.0
   b=1.0
   im=0.01
   ke=1.0
   fc=1.00
   iwu=75.0

   [KWParamSet EF5KY]
   under_grid=data/parameters/KW/ksat_KY.tif
   leaki_grid=data/parameters/KW/leaki_KY.tif
   alpha_grid=data/parameters/KW/alpha_KY.tif
   beta_grid=data/parameters/KW/beta_KY.tif
   alpha0_grid=data/parameters/KW/alpha0_KY.tif
   gauge=03282040
   alpha0=1.0
   alpha=1.0
   beta=1.0
   under=0.0001
   leaki=1.0
   th=10.0
   isu=00.0

   # The below is a block specific for CREST optimization
   [CrestCaliParams 0CRESTCALI]
   # Optimization for the location defined above
   gauge=03282040
   objective=CC        # options: CC, NSCE, BIAS
   dream_ndraw=20000   # Number of iterations, recomended: 10000
   wm=0.05,10
   b=0.05,10
   im=0.005,1
   ke=0.001,1
   fc=0.0,150.0
   iwu=0.0,300.0

   # The below is a block specific for KW optimization
   [kwcaliparams 0KWCALI]
   # Optimization for the location defined above
   gauge=03282040
   under=0.0001,0.00010001
   leaki=0.02,10.0
   th=10,10.00001
   isu=0.0,0.000001
   alpha=0.05,10
   beta=0.05,10
   alpha0=0.05,10

   [Task CREST_Simulation]
   STYLE=CALI_DREAM
   MODEL=crest
   ROUTING=KW
   BASIN=0
   PRECIP=MRMS
   PET=CLIMO
   OUTPUT=results
   STATES=data/states
   defaultparamsgauge=03282040
   PARAM_SET=EF5KY
   ROUTING_PARAM_Set=EF5KY
   CALI_PARAM=0CRESTCALI
   ROUTING_CALI_PARAM=0KWCALI
   TIMESTEP=30u
   TIME_BEGIN=20220727120000
   TIME_END=20220730120000

   [Execute]
   task=CREST_Simulation




Appendix
--------
### Complete Sample Configuration File

.. code-block:: ini

   /*
    * This is an example configuration file for EF5
    */

   [Basic]
   DEM=/EF5Demo/FF/basic/DEM.asc
   DDM=/EF5Demo/FF/basic/DDM.asc
   FAM=/EF5Demo/FF/basic/FAM.asc
   PROJ=laea
   ESRIDDM=true

   [PrecipForcing Q2Precip]
   TYPE=BIF
   UNIT=mm/h
   FREQ=5u
   LOC=/EF5Demo/FF/precip
   NAME=Q2_YYYYMMDDHHUU.bif

   [PETForcing PET]
   TYPE=BIF
   UNIT=mm/h
   FREQ=m
   LOC=/EF5Demo/FF/pet
   NAME=PET_MM.bif

   [Gauge OKC]
   LON=-97.01
   LAT=35.68
   OBS=/EF5Demo/obs/okc.csv

   [Gauge AR]
   LON=-93.62
   LAT=34.37

   [Basin FF]
   GAUGE=OKC
   GAUGE=AR

   [CrestParamSet FF]
   GAUGE=AR
   COEM=24.230076 EXPM=0.502391 RIVER=1.73056
   UNDER=0.291339 LEAKO=0.56668 LEAKI=0.251648
   TH=63.20205 GM=1.364364 PWM=71.96465
   PB=0.964355 PIM=6.508687 PKE=0.19952
   PFC=2.578529 IWU=53.52593 ISO=5.899539
   ISU=17.31128
   GAUGE=OKC
   COEM=24.230076 EXPM=0.502391 RIVER=1.73056
   UNDER=0.291339 LEAKO=0.56668 LEAKI=0.251648
   TH=63.20205 GM=1.364364 PWM=71.96465
   PB=0.964355 PIM=6.508687 PKE=0.19952
   PFC=2.578529 IWU=53.52593 ISO=5.899539
   ISU=17.31128

   [Task RunFF]
   STYLE=SIMU
   MODEL=CREST
   BASIN=FF
   PRECIP=Q2_PRECIP
   PET=PET
   OUTPUT=/EF5Demo/FF/output/
   PARAM_SET=FF
   TIMESTEP=5u
   TIME_BEGIN=201006010000
   TIME_END=201006010030

   [Execute]
   TASK=RunFF


Add your content using ``reStructuredText`` syntax. See the
`reStructuredText <https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html>`_
documentation for details.


.. toctree::
   :maxdepth: 2
   :caption: Contents:


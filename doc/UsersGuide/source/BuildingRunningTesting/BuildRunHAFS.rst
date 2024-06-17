.. _BuildRunHAFS:

*******************************
Building and Running HAFS
*******************************

This chapter walks users through a basic experiment using HAFS. Data comes from Hurricane Laura (2020-08-25 12z) and is already available on disk on supported systems. 

Currently, the HAFS application works on these NOAA HPC platforms: 

* wcoss_dell_p3
* wcoss_cray
* hera
* jet
* orion

=================================
Set Up the Directory Structure
=================================

.. note::

    Once this setup has been performed, it does not need to be repeated for successive experiments. 

HAFS is configured to use a particular directory structure for operations, and running HAFS will therefore go more smoothly when users set up a similar directory structure for running HAFS on their own system. On NOAA :term:`RDHPC <RDHPCS>` systems, users typically have disk space under a particular project. Users on any system can save their project directory in an environtment variable to reference later:

.. code-block:: console

    export PROJECT=/path/to/project/directory

Within a directory where the user has write access, create the following directories:

.. code-block:: console

    mkdir $PROJECT/save
    mkdir $PROJECT/noscrub

Within each of these directories, create a directory with your username. For example: 

.. code-block:: console

    mkdir $PROJECT/save/Jane.Doe
    mkdir $PROJECT/noscrub/Jane.Doe


=================================
Clone and Checkout the Repository
=================================

.. code-block:: console

    cd $PROJECT/save/<username>
    git clone -b <BRANCH> --recursive https://github.com/hafs-community/HAFS.git

Select the branch to clone by setting the ``<BRANCH>`` option to the branch name. In general, the ``production/hafs.v#`` branch with the highest version number will be up-to-date with code for the most recent or upcoming release. The ``feature/`` branches contains code that is in development by various HAFS developers. This code is typically merged to ``develop`` and/or to the upcoming ``production/hafs.v#`` branch when it is ready. 

.. note::
   ``develop`` is the default branch; the ``develop`` branch will be cloned if no branch is specified.

======================
Build and Install HAFS
======================

The ``install_hafs.sh`` script builds HAFS by calling several other scripts with distinct functions:

    * ``machine-setup.sh`` Determine shell, identify machine, and load modules
    * ``build_all.sh`` Compile components: forecast, post, tracker, utils, tools, hycom, ww3, and gsi
    * ``install_all.sh`` Copy executables to ``exec`` directory
    * ``link_fix.sh`` Link fix files (fix files are available on disk on supported platforms)

To run ``install_hafs.sh``, navigate to ``sorc``:

.. code-block:: console

    cd $PROJECT/save/<username>/HAFS/sorc
    ./install_hafs.sh > install_hafs.log 2>&1

.. note::

    Building HAFS can take a while, potentially upwards of an hour. To see output printed to the console as HAFS builds, users can omit ``> install_hafs.log 2>&1`` when running ``install_hafs.sh``. 

Once ``install_hafs.sh`` has run, ``install_hafs.log`` should appear in the ``sorc`` directory. Users can also check the log files in the ``HAFS/sorc/logs`` directory to see if the build was successful or if there were any errors. A successful build should result in a ``build_*.log`` file for each executable: 

    * build_forecast.log
    * build_gsi.log
    * build_hycom_utils.log
    * build_post.log
    * build_tools.log
    * build_tracker.log
    * build_utils.log
    * build_ww3_utils.log

Additionally, several executables should appear in a new ``HAFS/exec`` directory. These executables include:

    * hafs_forecast_*.x
    * hafs_gsi_enkf.x
    * hafs_gsi.x
    * hafs_hycom_utils_*.x
    * hafs_post.x
    * hafs_tools_*.x
    * hafs_tracker_*.x
    * hafs_utils_*.x
    * hafs_ww3_*.x

.. Hint::
   Got errors? Look into the ``HAFS/sorc/logs`` directory.

===================
Run the HAFS System
===================

----------------
Edit system.conf
----------------

To configure an experiment, run: 

.. code-block:: console

    cd $PROJECT/save/<username>/HAFS/parm
    cp system.conf.<system> system.conf
    vi system.conf

where ``<system>`` is replaced by the name of one of the supported platforms listed :ref:`above <BuildRunHAFS>`.

Edit the following:

    * ``disk_project``: Project name for disk space. 
    * ``tape_project`` (optional): :term:`HPSS` project name.
    * ``cpu_account``: CPU account name for submitting jobs to the batch system (may be the same as ``disk_project``)
    * ``archive=disk``: Archive location (make sure you have write permission)
    * ``CDSAVE``: HAFS parent directory
    * ``CDNOSCRUB``: Track files will be copied to this location --- contents will not be scrubbed (user must have write permission)
    * ``CDSCRUB`` If scrub is set to yes, this directory will be removed (user must have write permission)

For example, an edited ``system.conf`` file on Hera might resemble the following:

.. code-block:: console

    ## This is the system-specific configuration file for Hera
    [config]
    ## Project disk area
    disk_project=epic
    ## Project hpss tape area
    tape_project=emc-hwrf
    ## CPU account name for submitting jobs to the batch system.
    cpu_account=epic
    ## Archive path
    archive=disk:/scratch2/NAGAPE/epic/Gillian.Petro

    [dir]
    ## Save directory.  Make sure you edit this.
    CDSAVE=/scratch2/NAGAPE/epic/save/Gillian.Petro
    ## Non-scrubbed directory for track files, etc.  Make sure you edit this.
    CDNOSCRUB=/scratch2/NAGAPE/epic/noscrub/Gillian.Petro/hafstrak
    ## Scrubbed directory for large work files.  Make sure you edit this.
    CDSCRUB=/scratch2/NAGAPE/epic/scrub/Gillian.Petro
    ## Syndat directory for finding which cycles to run
    syndat=/scratch1/NCEPDEV/hwrf/noscrub/input/SYNDAT-PLUS
    COMOLD={oldcom}
    COMIN={COMhafs}
    COMOUT={COMhafs}
    COMINnhc={ENV[DCOMROOT|-/dcom]}/nhc/atcf/ncep
    COMINjtwc={ENV[DCOMROOT|-/dcom]}/{ENV[PDY]}/wtxtbul/storm_data
    COMgfs=/scratch1/NCEPDEV/hwrf/noscrub/hafs-input/COMGFSv16
    COMINobs={COMgfs}
    COMINgfs={COMgfs}
    COMINgdas={COMgfs}
    COMINarch={COMgfs}/syndat
    COMrtofs=/scratch1/NCEPDEV/hwrf/noscrub/hafs-input/COMRTOFSv2
    COMINrtofs={COMrtofs}
    COMINmsg={COMINgfs}
    COMINhafs={COMINgfs}
    DATMdir=/scratch1/NCEPDEV/{disk_project}/noscrub/{ENV[USER]}/DATM
    DOCNdir=/scratch1/NCEPDEV/{disk_project}/noscrub/{ENV[USER]}/DOCN
    ## A-Deck directory for graphics
    ADECKhafs=/scratch1/NCEPDEV/hwrf/noscrub/input/abdeck/aid
    ## B-Deck directory for graphics
    BDECKhafs=/scratch1/NCEPDEV/hwrf/noscrub/input/abdeck/btk
    ## cartopyDataDir directory for graphics
    cartopyDataDir=/scratch1/NCEPDEV/hwrf/noscrub/local/share/cartopy


.. _physics:

---------------------------
HAFS Physics Configuration
---------------------------

Look in ``HAFS/parm/hafs.conf`` to determine what physics suites are running.

.. figure:: https://github.com/hafs-community/HAFS/wiki/docs_images/hafs_ccpp_suites.png
    :width: 50%
    :alt: CCPP suites listed in hafs.conf (updated 06/29/2023)

To determine what physics schemes are included in the suites mentioned above, run:

.. code-block:: console

    more HAFS/sorc/hafs_forecast.fd/FV3/ccpp/suites/suite_FV3_HAFS_v1_gfdlmp_tedmf_nonsst.xml


.. COMMENT: Current physics
    ccpp_suite_regional=FV3_HAFS_v1_thompson_nonsst
    ccpp_suite_glob=FV3_HAFS_v1_thompson_nonsst
    ccpp_suite_nest=FV3_HAFS_v1_thompson_nonsst

.. _namelist-files:

---------------------------
HAFS Nesting Configuration
---------------------------

Two types of nesting configurations are available: (i) regional* and (ii) globnest.

* Two namelist files (templates) for regional configuration are:

  * ``HAFS/parm/forecast/regional/imput.nml.tmp``
  * ``HAFS/parm/forecast/regional/input_nest.nml.tmp``

* One namelist file (template) for globnest configuration is:

  * ``HAFS/parm/forecast/globnest/input.nml.tmp``

.. figure:: https://github.com/hafs-community/HAFS/wiki/docs_images/hafs_namelist_files.png
    :width: 50 %
    :alt: Example namelist file for HAFS (updated 06/29/2023)

\* operational implementation

----------------------------
XML File to Run the Workflow
----------------------------

.. code-block:: console

    cd /path/to/HAFS/rocoto
    vi hafs_workflow.xml.in

In ``HAFS/rocoto/hafs_workflow.xml.in`` the following can be modified to set the number of cycles and tasks.

* ``<!ENTITY CYCLE THROTTLE “5”>``: The number of cycles that can be activated at one time
* ``<!ENTITY TASK_THROTTLE “120”>``: The number of tasks that can be activated at one time
* ``<!ENTITY MAX_TRIES “1”>``: The maximum number of tries for all tasks

-------------------------------
Edit the Cron Job Driver Script
-------------------------------

Change the cron job driver script to set up the experiment and storm.

.. code-block:: console

    cd /path/to/HAFS/rocoto
    vi cronjob_hafs.sh

Make sure to uncomment ``#set -x`` and edit ``HOMEhafs`` as appropriate. For example: 

.. code-block:: console

    #!/bin/sh
    set -x
    date

    HOMEhafs=${HOMEhafs:-/scratch2/NAGAPE/epic/save/<username>/HAFS}

-----------------------------
Run HAFS and Check Progress
-----------------------------

Run the driver script in the ``rocoto`` directory to launch the experiment: 

.. code-block:: console

    ./cronjob_hafs.sh

To run through all tasks in the experiment, tasks need to be launched once their dependencies are satisfied. Users can launch tasks manually by running the ``rocotorun`` command regularly and repeatedly until all tasks are complete: 

.. code-block:: console

    rocotorun -d hafs-HAFS-13L-2020082512.db -w hafs-HAFS-13L-2020082512.xml

Instead of running ``rocotorun`` manually, users can instead automate this task by adding it to a crontab on systems where :term:`cron` is available: 

.. code-block:: console

    crontab -e
    */5 * * * * cd /path/to/HAFS/rocoto && ./cronjob_hafs.sh

For example, a user named Jane Doe might paste ``*/5 * * * * cd /scratch2/NAGAPE/epic/save/Jane.Doe/HAFS/rocoto && ./cronjob_hafs.sh`` into her crontab. 

.. note::

   On Orion, cron is only available on the orion-login-1 node.


To check experiment progress, users can run the ``rocotostat`` command:

.. code-block:: console

    rocotostat -d hafs-HAFS-13L-2020082512.db -w hafs-HAFS-13L-2020082512.xml

To check which specific tasks are in progress, users can run:

.. code-block:: console

    squeue -u <username>



.. COMMENT: 
    ./hafs_rt_status.sh
    hafs-HAFS_rt_regional_static_C192s1n4_atm_ocn_wav-00L-2020082512.xml



---
layout: page-fullwidth
title: Updating a UM suite after the ARCHER2 O/S upgrade
subheadline:  ARCHER2 O/S Upgrade 
permalink: '/archer2/os-upgrade/'
breadcrumb: true
---



{% include alert info="This page will be continually updated." %}

In May/June 2023, Archer2 underwent a [major software upgrade](https://docs.archer2.ac.uk/faq/upgrade-2023/). All the system software, including compilers and libraries were updated, and as a result, we have had to rebuild much of the UM supporting software. This means users need to make some changes to their suites and test their workflows before resuming work. 

We have ported and tested some of the commonly-used suites, listed below. And we have provided a guide to the suite changes required. As ever, since UM suites can be set up in so many different ways, we can not provide a comprehensive set of instructions. Please get in touch with the [helpdesk](https://cms-helpdesk.ncas.ac.uk/) if you run into diffculties. 

## Instructions 

The following instructions draw on the naming style typically found in climate suites, but the ideas should apply to all UM suites. Suite modifications derive to accommodate changes to the slurm job scheduler. Minimal user-level changes are required and suites should run successfully. We stress that **the UM executables for the reconfigurarion and the atmosphere model must be rebuilt**.


### Atmosphere Suites
For atmosphere-only suites add the ```{% raw %} --cpus-per-task={{MAIN_OMPTHR_ATM}} {% endraw %}``` clause to the atmopshere resources ```[[[environment]]]``` section in ```archer2.rc```. For example:

{% raw %}
~~~
[[ATMOS_RESOURCE]]
    ...
    [[[environment]]]
        OMP_NUM_THREADS={{MAIN_OMPTHR_ATM}}
        ROSE_LAUNCHER_PREOPTS = {{ATM_SLURM_FLAGS}} --cpus-per-task={{MAIN_OMPTHR_ATM}}
~~~
{% endraw %}

### Coupled Suites
Coupled atmosphere-ocean suites require changes to suite files ```rose-suite.conf``` and ```archer2.rc.```
1. In ```rose-suite.conf``` change the Science Configuration Module (see the table below for the required mappings)

  | Pre OS upgrade MOCI module | Post OS upgrade MOCI module |
  | --- | ---|
  | GC3-PrgEnv/2.0/2021.12.15 |  GC3-PrgEnv/v1 |
  | GC3-PrgEnv/2.0/2021.11.22 |  GC3-PrgEnv/v2 |
  | GC3-PrgEnv/2.0/2022.12.09 |  GC3-PrgEnv/v3 |
  | GC4-PrgEnv/2021.12.1      |  GC4-PrgEnv/v1 |
  | GC5-PrgEnv/2023.01.1      |  GC5-PrgEnv/v1 |

{:start="2"}
2. Update the cce module version and remove the ucx module swap entries in ```archer2.rc```, ie, change

{% raw %}
~~~
     module load cce/12.0.0
     module swap craype-network-ofi craype-network-ucx
     module swap cray-mpich cray-mpich-ucx/8.1.15
    {{MODULE_CMD}}
~~~
{% endraw %}

to

{% raw %}
~~~
    module load cce/15.0.0
    {{MODULE_CMD}}
~~~
{% endraw %}

{:start="3"}
3. Update the ```ROSE_LAUNCHER_PREOPTS``` for the UM, NEMO, and XIOS. \\
  There is no longer any need to distinguish between single and multithreaded cases, but note the clauses ``` --hint=nomultithread --distribution=block:block``` must appear in the ```ROSE_LAUNCHER_PREOPTS``` for the UM, NEMO, and XIOS

{% raw %}
~~~
       [[UM_RESOURCE]]
           [[[environment]]]
                  ROSE_LAUNCHER_PREOPTS_UM  = --het-group=0 --nodes={{ATMOS_NODES}} --ntasks={{ATMOS_TASKS}} --tasks-per-node={{ATMOS_PPNU*NUMA}} --cpus-per-task={{OMPTHR_ATM}} --hint=nomultithread --distribution=block:block --export=all,OMP_NUM_THREADS={{OMPTHR_ATM}},HYPERTHREADS={{HYPERTHREADS}},OMP_PLACES=cores
    
       [[NEMO_RESOURCE]]
             [[[environment]]]
                  ROSE_LAUNCHER_PREOPTS_NEMO  = --het-group=1 --nodes={{OCEAN_NODES}} --ntasks={{OCEAN_TASKS}} --tasks-per-node={{OCEAN_PPNU*NUMA}} --cpus-per-task={{OMPTHR_OCN}} --hint=nomultithread --distribution=block:block --export=all,OMP_NUM_THREADS={{OMPTHR_OCN}},HYPERTHREADS={{HYPERTHREADS}},OMP_PLACES=cores
    
                  {% if XIOS_NPROC is defined and XIOS_NPROC > 0 %}
                  ROSE_LAUNCHER_PREOPTS_XIOS  = --het-group=2 --nodes={{XIOS_NODES}} --ntasks={{XIOS_TASKS}} --tasks-per-node={{XIOS_PPNU*NUMA}} --cpus-per-task=1 --hint=nomultithread --distribution=block:block --export=all,OMP_NUM_THREADS=1,HYPERTHREADS=1
                  {% endif %}
~~~
{% endraw %}

## Ported suites 

The following suites have been updated and tested following the OS Upgrade.

| UM version | Suite id | Description | Branches + Note |
| --- | --- | --- | --- |
| 11.1 | u-be303/archer2 | UKESM1.0 AMIP | |
| 11.2 | u-bc613/archer2 | UKESM1.0 Historical | see changes to the hetjob config in site/archer2.rc |
| 11.2 | u-bc964/archer2 | UKESM1.0 pre-industrial control | see changes to the hetjob config in site/archer2.rc |
| 11.6 | u-bs251/archer2 | GA7.0 N96 AMIP Climate Development | |
| 11.7 | u-ca634 | GA8.0GL9.0 AMIP Climate Development| |
| 12.2 | u-cm785/archer2 | GC4 N96 ORCA025| |
| 13.2 | u-cy010 | GC5 N216 ORCA025 | |

## How to restart suites

**Note - only follow this section if the fcm_make\* tasks do not appear in the Cylc gui for the last active cycle**

Suites that were running at the time ARCHER2 went down need to have their fcm_make* tasks re-inserted and re-run in order to rebuild the module executables.

* After making the above changes, restart the suite in a held state:
  
  ```puma$ rose suite-run --restart -- --hold```
  
* Identify the cycle-point of the last active cycle (e.g. 19990401T0000Z)

* For atmos only suites
  
  ```archer2$ cd <your-suite>/share``` \\
  ```archer2$ mv fcm_make_um fcm_make_um_preupgrade```

  Insert the build tasks by running:
    
  ```puma$ cylc insert --no-check SUITE-ID fcm_make_um.CYCLE-POINT``` \\
  ```puma$ cylc insert --no-check SUITE-ID fcm_make2_um.CYCLE-POINT```

  For example:  
  ```cylc insert --no-check u-cm123 fcm_make_um.19990401T0000Z```

* For coupled suites
  
  ```archer2$ cd <your-suite>/share``` \\
  ```archer2$ mv fcm_make_um fcm_make_um_preupgrade``` \\
  ```archer2$ mv fcm_make_ocean fcm_make_ocean_preupgrade```

  Insert the build tasks by running:
  
  ```puma$ cylc insert --no-check SUITE-ID fcm_make_um.CYCLE-POINT``` \\
  ```puma$ cylc insert --no-check SUITE-ID fcm_make2_um.CYCLE-POINT``` \\
  ```puma$ cylc insert --no-check SUITE-ID fcm_make_ocean.CYCLE-POINT``` \\
  ```puma$ cylc insert --no-check SUITE-ID fcm_make2_ocean.CYCLE-POINT```

* Right click on the ```fcm_make_*``` tasks and select **"Release"**

* Wait for the code extraction tasks to finish, then right click on the ```fcm_make2_*``` tasks and select **"Release"** to rebuild the code.

* Once the rebuild tasks have finished release the suite to continue running by selecting menu item **"Control -> Release Suite (unpause)"**

---
layout: page-fullwidth
subheadline: UMUI
title: Using the UMUI on PUMA2 
teaser: Here you will find instructions on using the UMUI on PUMA2 
permalink: /puma2/umui
---

## UMUI Transition to PUMA2 Timeline

* Monday 16th October 16:00hrs: UMUI switched off on current PUMA server
* Tuesday 17th October (AM): UMUI job database copied over to PUMA2 and UMUI service restarted on PUMA2
  
## Important differences from using the UMUI on PUMA:

1) If your username on PUMA2 is different from the username you had on PUMA, you will not be able to edit existing jobs.  You first need to copy the job to a new Expt/Job Id.

2) Change the run host from `login.archer2.ac.uk` to one of the ARCHER2 login nodes: `ln01`, `ln02`, `ln03` or `ln04`.

4) Any paths to local directories like `/home/<user>` need to be changed to `/home/n02/n02/<user>`.

5) The SUBMIT button no longer works.  To submit a job to ARCHER2, after processing the job as usual, you will then need to run the UMSUBMT_ARCHER2 on the command line:
   ```
   cd ~/umui_jobs/<jobid>
   ./UMSUBMIT_ARCHER2
   ```

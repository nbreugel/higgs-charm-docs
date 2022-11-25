## The RAW step
**This step takes place in the `prod_<YOURTAG>/CMSSW_10_6_17_patch1/src` directory.**

Following the GEN step, one has the run the SIM-DIGI-RAW steps. These are already included in the configuration file [RAW_cfg.py](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/LHEtoAOD/RAW_cfg.py). A submission script has been prepared to run in parallel on several LHE files via HTCondor. This script can be found under [submit_RAW.py](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/LHEtoAOD/submit_RAW.py).

It can be ran providing the following arguments:

  * **_--config:_** the configuration file to run (will be [RAW_cfg.py](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/LHEtoAOD/RAW_cfg.py) for this step)
  * **_--indir:_** path to input directory that contains input GEN (.root) files
  * **_--outdir:_** path to output directory in which the .root files will be stored (note that the tag specified later will be appended to this directory name!)
  * **_--jobflavour:_** Limit to the duration of the job on condor, as specified [here](https://batchdocs.web.cern.ch/local/submit.html)
  * **_--tag:_** A tag to specify the name of the working directory and output directory

For example:

```shell
python submit_RAW.py \
--config=./RAW_cfg.py \
--indir=/pnfs/iihe/cms/store/user/nbreugel/HcToFourMuons/GEN \
--outdir=/pnfs/iihe/cms/store/user/nbreugel/HcToFourMuons/RAW \
--jobflavour=tomorrow \
--tag=Hc4Mu
```

Make sure your grid proxy is valid before submitting the jobs to HTCondor. This step can take quite a while to run, so a `jobflavour=nextweek` could be necessary.

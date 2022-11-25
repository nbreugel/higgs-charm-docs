## The GEN step
**This step takes place in the `prod_<YOURTAG>/CMSSW_10_6_17_patch1/src` directory.**

The first step is called the generator step or GEN step for short. It uses the previously generated LHE files as input, and outputs a collection of ROOT files containing the showered particles. 

For testing purposes, this step can be run on a file-per-file basis using
```shell
cmsRun GEN_cfg.py infile=/PATH/TO/INPUT/FILE.lhe outfile=/PATH/TO/OUTPUT/FILE.root nevents=-1
```

The value of `nevents=-1` is interpreted as a full run over all of the events in the input file. For testing purposes, this value can be set to a lower number to decrease the CPU time.

To execute the GEN step on a collection of LHE files, an HTCondor submission script has been prepared and can be found under [submit_GEN.py](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/LHEtoAOD/submit_GEN.py). This script produces a directory containing the HTCondor logs and an HTCondor `.submit` file. The script takes in several arguments:

  * **_--config:_** the configuration file to run (will be [GEN_cfg.py](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/LHEtoAOD/GEN_cfg.py) for this step)
  * **_--indir:_** path to input directory that contains input LesHouches (.lhe) files
  * **_--outdir:_** path to output directory in which the .root files will be stored (note that the tag specified later will be appended to this directory name!)
  * **_--jobflavour:_** Limit to the duration of the job on condor, as specified [here](https://batchdocs.web.cern.ch/local/submit.html)
  * **_--tag:_** A tag to specify the name of the working directory and output directory

For example:
```shell
python submit_GEN.py \
--config=./GEN_cfg.py \
--indir=/pnfs/iihe/cms/store/user/nbreugel/HcToFourMuons/LHE \
--outdir=/pnfs/iihe/cms/store/user/nbreugel/HcToFourMuons/GEN \
--jobflavour=tomorrow \
--tag=Hc4Mu
```

Before submitting the jobs to HTCondor, make sure you have a valid grid proxy:
```shell
voms-proxy-init --voms cms --valid 192:00
```
and additionally, have the variable `X509_USER_PROXY` point towards the X509 proxy file:
```shell
export X509_USER_PROXY=$(voms-proxy-info -path)
echo $X509_USER_PROXY
```

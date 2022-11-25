## The HLT step
**This step takes place in the `prod_<YOURTAG>/CMSSW_10_2_16_UL/src` directory.**

Similar to the previous steps, the HLT step is included in the configuration file [HLT_cfg.py](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/LHEtoAOD/HLT_cfg.py) and can be run in parallel using the python script found under [submit_HLT.py](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/LHEtoAOD/submit_HLT.py). Make sure you are in the appropriate CMSSW environment (`CMSSW_10_2_16_UL`) before running this script. The input files for this step are the HLT (.root) files that were produced previously.

For example:

```shell
python submit_HLT.py \
--config=./HLT_cfg.py \
--indir=/pnfs/iihe/cms/store/user/nbreugel/HcToFourMuons/RAW \
--outdir=/pnfs/iihe/cms/store/user/nbreugel/HcToFourMuons/HLT \
--jobflavour=tomorrow \
--tag=Hc4Mu
```

Make sure your grid proxy is valid before submitting the jobs to HTCondor.

## The MINIAOD step
**This step takes place in the `prod_<YOURTAG>/CMSSW_10_6_17_patch1/src` directory.**

Finally, the RECO-AOD-MINIAOD need to be run in order to obtain a ROOT file ready for physics analysis. This step is included in the configuration file [MINIAOD_cfg.py](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/LHEtoAOD/MINIAOD_cfg.py) and can be run over several files in parallel using the python script found under [submit_MINIAOD.py](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/LHEtoAOD/submit_MINIAOD.py). The input files for this step are the HLT (.root) files that were produced previously.

For example:

```shell
python submit_MINIAOD.py \
--config=./MINIAOD_cfg.py \
--indir=/pnfs/iihe/cms/store/user/nbreugel/HcToFourMuons/HLT \
--outdir=/pnfs/iihe/cms/store/user/nbreugel/HcToFourMuons/MINIAOD \
--jobflavour=tomorrow \
--tag=Hc4Mu
```

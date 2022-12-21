# Higgs-Charm repository guide

## Introduction

The Higgs-Charm repository for full-sim MC production can be cloned using the command

```shell
git clone git@github.com:nbreugel/higgs-charm.git
```

The **main functions** of this repository are:

* The production of MadGraph cards which contain the information related to the process of interest.
* The production and submission of HTCondor scripts which run the CMS Gridpack creation process on the T2B machines.
* The setup of a pipeline to produce Ultra Legacy 2018 CMS Full-Sim samples.

It is also possible to clone this repository into an empty directory by simply passing the path to this directory as an argument to the above command. For example

```shell
git clone git@github.com:nbreugel/higgs-charm.git /path/to/clean/directory
```

## Setting up your process

### Edit the process configuration file
Inside [process_cfg.py](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/process_cfg.py), you need to fill out all of the necessary fields outlined below to configure the process you want to generate:

* **_tag_**: can be any name used to identify the process. It will be used as a directory name to save the MadGraph cards in [addons/cards/](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/addons/cards). It is important to remember this tag further along.
* **_model_name_**: here you can pass the exact name of the UFO model you would like to use to calculate the matrix elements for your process. These models can for example be found in the [feynrules model database](https://feynrules.irmp.ucl.ac.be/wiki/ModelDatabaseMainPage) and should be placed in [addons/models/](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/addons/models). Some models are already included by default:
  > * The [dim6top_LO_UFO](https://feynrules.irmp.ucl.ac.be/wiki/dim6top) model, used for the study of top quark physics with Effective Field Theory.
  > * The [Higgs_Effective_Couplings](https://feynrules.irmp.ucl.ac.be/wiki/HiggsEffectiveTheory) model, which introduces leading-order Higgs-gluon interactions.
  > * The [SMEFTsim_general_MwScheme_UFO_v3](https://feynrules.irmp.ucl.ac.be/wiki/SMEFT) model (Standard Model Effective Field Theory).

* **_processes_**: an array of strings, each representing a process you want to generate with Madgraph. These processes will be added together in the proc_card.dat.
* **_flavour_scheme_**: specify here whether you want to use the 4F (massive b, not in the proton pdf) or 5F scheme (massless b, included in the proton pdf). **This option only redefines the proton multiparticle!** It is therefore your responsibility to make sure the bottom quark mass is set accordingly in your model (preferably in a restriction card). Additionally, make sure you are using a 5F parton distribution function.
* **_precision_**: the order to which you would like to simulate your process. If `NLO` is specified, your decays need to be handled separately by MadSpin.
* **_madspin_commands_**: if the decays are handled by MadSpin, you can specify the necessary commands here similarly to the _processes_ array.

Additionally, there are fields related to the study of EFT operators. If these are not relevant to your process, you can simply state `no_reweights` in the `reweighting_strategy` field and leave all other fields empty.

* **_operators_**: an array of EFT operators you want to probe. The names should match exactly those in the `parameters.py` file in your UFO directory.
* **_baseline_values_**: an array of initial values for the Wilson coefficients of each operator. This defines the baseline scenario to which the reweighting factors will be calculated. The choice of these values is of crucial importance to ensure a proper coverage of the entire phase space. Putting all values to 0 (~SM) will most likely result in a bad coverage in the part of the Phase space where the EFT is expected to be most abundant, resulting in large weights and uncertain predictions of the yields and differential distributions. Properly assigning these values requires careful studies.

* **reweighting_strategy**: please choose one of the following options
  > * **_no_reweights_**: no reweighting is applied<br/>
  > * **_individual_**: each operator is varied individually<br/>
  > * **_rnd_scan_**: a random scan over all operators is performed<br/>
  > * **_grid_**: a rectangular grid of Wilson coefficients is scanned<br/>
  > * **_minimal_**: a tetrahedron-construction of scan points that picks the minimal number of scan points needed for a fit of a given order<br/>
  > * **_custom_**: a custom reweighting scheme is provided by the user
		
	Then navigate to the corresponding "if statement" and fill out the needed parameters:

   	> * **_individual_**: "`points_individual`" is an array of arrays where in each subarray the user has to manually specify the values of the WCs to be scanned for the corresponding operator (using the same ordering as the "`operators`" variable).<br/>
   	> * **_rnd_scan_**: specify in "`n_points`" the number of scan points (NOTE: for N operators you need at least `1 + 2*N + (N*(N-1))/2` scan points to determine the quadratic function of the cross section). Then specify the boundaries for each operator in "`boundaries`".<br/>
   	> * **_grid_**: specify in each subarray of "`boundaries_and_npoints`" the lower and upper boundary as well as the number of points to be scanned for each operator. For k points and N operators, a grid of k^N points will be constructed.<br/>
   	> * **_minimal_**: specify in each subarray of "`boundaries_minimal_scan`" the lower and upper boundary of each operator. Also the "`order`" of the fitted yield must be given (usually 2, but can be 1 for interference only or larger than 2 for multiple insertions of EFT operators).<br/>
   	> * **_custom_**: The user has to manually specify the dictionary "`reweight_dict_tmp_`" where each key,value pair represents one specific scenario of a given set of WCs.
   		
   Finally, the SM scenario (all WCs put to 0), will be included by default.
   The naming convention (when not defining "custom" as a reweighting strategy) is defined by the `translate_weight_name` helper function. It starts with "rwgt_", followed by listing all operators with their values. Decimal points are replaced by "p" and minus signs by "min".


### Create MadGraph cards for your process
The [prepare_process.py](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/prepare_process.py) script will read out the configuration from the [process_cfg.py](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/process_cfg.py) file and construct the following cards in `addons/cards/<tag>`:
> * run_card.dat<br/>
> * proc_card.dat<br/>
> * reweight_card.dat<br/>
> * customizecards.dat<br/>
> * madspin_card.dat<br/>
	
To this end, run:
```shell
python prepare_process.py
```

This will also create a copy of [submit_gridpack_template.sh](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/submit_gridpack_template.sh), which will be called `submit_gridpack_T2B.sh` and for which the chosen `tag` is correctly inserted in the script.

### Clone genproductions
*You only need to run this step once. If you have already cloned genproductions, proceed to [the next step](#14-move-cards-to-madgraph-working-directory)* 

For the production of CMS samples, we need the pull the [cms-sw/genproductions](https://github.com/cms-sw/genproductions.git) package from github. To this end, run:
```
source setup_production.sh
```
This will create a new directory called "genproductions" alongside your current mcgeneration repository and it will copy all the scripts, models, cards that you need to run your sample production. **Since the cards have already been moved to the appropriate directory, you can now proceed to the next page.**

### Move cards to MadGraph working directory
After having produced the MadGraph cards, you can move the cards to the `genproductions/bin/MadGraph5_aMCatNLO/` directory using the command
```shell
sh move_cards.sh <TAG>
```
Make sure that you use exactly the same `tag` as the one you specified in the previous step. Afterwards, you are automatically directed to the `genproductions/bin/MadGraph5_aMCatNLO/` directory.

## Producing a gridpack and launching LHE production
**NOTE: All of the following steps take place within the `genproductions/bin/MadGraph5_aMCatNLO/` directory!**

Before running the following steps, make sure you have your GRID proxy enabled and exported as a variable so the CMSSW scripts can retrieve it. This can be done by running
```shell
source setup_proxy.sh
```
Additionally, if you are interested in using the five-flavour scheme where the b and c masses (but not the Yukawa couplings) are set to zero in the `no_masses` restriction card of the `loop_sm` model, please run the following command:
```shell
source loopsm_5F_patch.sh
```
This script adds search and replace commands inside the `gridpack_generation.sh` script which alter the appropriate restriction card in the freshly installed `loop_sm` model during the production of the gridpack.

### Production of your CMS gridpack
Gridpacks can be produced by running:
```shell
python ProduceGridpack_condor.py
```
There are two optional arguments, namely the SCRAM architecture (`--scram_arch`) and the CMSSW version (`--cmssw_version`) that are to be used for the gridpack generation. If these are not given, by default the script takes `slc7_amd64_gcc700` and `CMSSW_10_2_18`.
The above command will create an HTCondor submission script. This can be submitted in order to launch the gridpack creation job. After the job has finished, you should see a tarball in your directory, which holds your gridpack.

**_NOTE: gridpack generation can take several minutes up to several hours, depending on your process and the amount of weights/diagrams that need to be saved._**

### Launch production of LHE files
Within the `genproductions/bin/MadGraph5_aMCatNLO/` folder, you will see the [ProduceLHE_condor.py](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/ProduceLHE_condor.py) file. It will create the needed scripts and condor submission configurations to run a parallel production or a chosen number of Les Houches event files (LHE).
It contains 6 configurable parameters:

* **_--tag_**: A specific tag name to create log files etc.<br/>
* **_--gridpack_**: path to the tarball from the gridpack production<br/>
* **_--outdir_**: absolute path to the output directory where the `.lhe` files will be stored. This has to be a folder with write access and anough storage space. A good example is your personal directory on the T2B PNFS storage system (`/pnfs/iihe/cms/user/$USER/myprocess`). If the directory does not yet exist, the script will try to create it, or it will terminate is it fails to do so.<br/>
* **_--jobflavour_**: jobFlavour as described in [https://batchdocs.web.cern.ch/local/submit.html](https://batchdocs.web.cern.ch/local/submit.html). This defines the walltime for each job.<br/>
* **_--neventstotal_**: total number of events to simulate.<br/>
* **_--neventsperjob_**: number of events per condor job. The number of jobs will be `--neventstotal/--neventsperjob` (rounded up).

Example:
```shell
python ProduceLHE_condor.py \
--tag=test \
--gridpack=./test_slc6_amd64_gcc630_CMSSW_9_3_16_tarball.tar.xz \
--outdir=/pnfs/cms/store/user/nbreugel/output_dir/ \
--jobflavour=longlunch \
--neventstotal=1500000 \
--neventsperjob=10000
```
This will now create several files, amongst which is `ProduceLHE_condor_<tag>.submit`. You can submit this to the HTCondor scheduler by running:
```shell
condor_submit ProduceLHE_condor_<tag>.submit
```
and you can check the status using
```shell
condor_q
```
Once all jobs have finished, the output is stored in the `outdir` in the form of a number of files named "`cmsgrid_final_1.lhe` with increasing file numbers.

## Setting up the next simulation steps

If everything went well, you should now have a series of .LHE files in your storage directory. The following steps will deal with the showering and decays of your events, as well as the interactions with the CMS detector and the simulation of the detector response.

First, run the [setupProd.sh](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/setupProd.sh) script, providing a tagname for the folder that is about to be created as an argument.

For example:

```shell
source setupProd.sh HcToFourMuons
```

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

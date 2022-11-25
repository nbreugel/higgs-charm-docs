# Producing a gridpack and launching LHE production
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

## Production of your CMS gridpack
Gridpacks can be produced by running:
```shell
python ProduceGridpack_condor.py
```
There are two optional arguments, namely the SCRAM architecture (`--scram_arch`) and the CMSSW version (`--cmssw_version`) that are to be used for the gridpack generation. If these are not given, by default the script takes `slc7_amd64_gcc700` and `CMSSW_10_2_18`.
The above command will create an HTCondor submission script. This can be submitted in order to launch the gridpack creation job. After the job has finished, you should see a tarball in your directory, which holds your gridpack.

**_NOTE: gridpack generation can take several minutes up to several hours, depending on your process and the amount of weights/diagrams that need to be saved._**

## Launch production of LHE files
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

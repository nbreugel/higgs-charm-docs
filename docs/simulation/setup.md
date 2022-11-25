# Setting up the repository

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

# Setting up the next simulation steps

If everything went well, you should now have a series of .LHE files in your storage directory. The following steps will deal with the showering and decays of your events, as well as the interactions with the CMS detector and the simulation of the detector response.

First, run the [setupProd.sh](https://github.com/nbreugel/higgs-charm/blob/main/mcgeneration/setupProd.sh) script, providing a tagname for the folder that is about to be created as an argument.

For example:

```shell
source setupProd.sh HcToFourMuons
```

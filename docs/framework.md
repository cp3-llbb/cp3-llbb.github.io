# Framework

Common framework for all cp3-llbb analyses

!!! Note

    - The instructions are for the UCLouvain ingrid SLC6 cluster (to access SAMADhi)
    - You need the proper username and password to access SAMADhi :) If you don't know what this is about, ask around
    - The current state of the art mini-AOD documentation can be found [here](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookMiniAOD2015)
    - You will probably want to install as well [GridIn](https://github.com/cp3-llbb/GridIn) to run jobs on the grid, and one of the existing analyses ([TTAnalysis](https://github.com/cp3-llbb/TTAnalysis), [HHAnalysis](https://github.com/cp3-llbb/HHAnalysis), [ZAAnalysis](https://github.com/cp3-llbb/ZAAnalysis))

## CMSSW release

**{{ framework.cmsswrelease }}**

## First time setup instructions

```bash
## the following two lines can be replaced by a call to the cms_env alias (see below)
source /nfs/soft/grid/ui_sl6/setup/grid-env.sh
source /cvmfs/cms.cern.ch/cmsset_default.sh

wget https://raw.githubusercontent.com/cp3-llbb/Framework/{{ framework.defaultbranch }}/setup_project_with_framework.sh
source setup_project_with_framework.sh --branch {{ framework.defaultbranch }}
```

The [script](https://github.com/cp3-llbb/Framework/blob/{{ framework.defaultbranch }}/setup_project_with_framework.sh) above will set up a CMSSW release area, apply the recipes in [``bootstrap_jenkins.sh``](https://github.com/cp3-llbb/Framework/blob/{{ framework.defaultbranch }}/bootstrap_jenkins.sh) and [``jenkins_postbuild.sh``](https://github.com/cp3-llbb/Framework/blob/{{ framework.defaultbranch }}/jenkins_postbuild.sh), perform an initial build, and add your and your colleagues' forks on GitHub as remotes for your ``Framework`` clone (all those that have been pushed to in the last year; you can update the list by running ``updateremotes``).
Through the options `--branch NAME` and `--pr ID`, a project area for a different version can also be created.

If you are using ingrid, here's a useful alias to put in your ``bashrc`` file:

```bash
alias cms_env="module purge; module load grid/grid_environment_sl6; module load crab/crab3; module load cms/cmssw; module load slurm/slurm_utils;"
```

Then, just do ``cms_env`` to load all the CMSSW environment.

## Test run (command line)

```bash
cd ${CMSSW_BASE}/src/cp3_llbb/Framework/test
cmsRun TestConfigurationMC.py
```

## When willing to commit things

* Remember to *branch before committing anything*: ```git checkout -b my-new-branch```
* Any branches to merge into CMSSW, packages to add, version and ``SCRAM_ARCH`` changes should be added to ``bootstrap_jenkins.sh``, ``jenkins_postbuild.sh``, ``CMSSW.release`` and ``CMSSW.arch``, respectively, such that they are also picked up by Jenkins, more details [here](fwk_jenkins.md#Bootstrap).
* The ```updateremotes``` script (run from ```setup_project_with_framework.sh```) took care of adding ```origin``` as your own repo, so to push just do the usual ```git push origin my-new-branch```
* If you change anything to the output trees (new or modified branches, new recipes etc.), the automatic tests (see below) will fail, because they compare the outputs to reference files.
  You can resolve this by regenerating the reference files with the [`test/generate_reference_trees.sh`](https://github.com/cp3-llbb/Framework/blob/{{ framework.defaultbranch }}/test/generate_reference_trees.sh) script, after committing your other changes.
  It will also print a summary of all differences in the output files. If these are as expected, you can make a new commit with the updated reference files.

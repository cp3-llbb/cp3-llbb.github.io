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

## Finding your way around the different modules

The Framework configures a CMSSW sequence and runs an [`edm::EDProducer`](http://cmsdoxygen.web.cern.ch/cmsdoxygen/{{ framework.cmsswrelease }}/doc/html/d9/d55/classedm_1_1EDProducer.html) module, [`ExTreeMaker`](../doxyfwk/classExTreeMaker.html),
to select events and create a TTree for analysis.
The structure is modular, so most of the actual code to select events and fill
branches is in the producers, analyzers, filters and categories;
`ExTreeMaker` only knows the interface Producer the different components implement (technically: it only has a pointer to the interface class, and the instances
are created by a factory - so you can also add modules in another package).

Producers ([`Framework::producer`](../doxyfwk/classFramework_1_1Producer.html))
are run first, and fill branches for event information that is present
in the input file (so not analysis-specific): leptons, jets, event information,
weight etc., while analyzers ([`Framework::analyzer`](../doxyfwk/classFramework_1_1Analyzer.html)
are run later, and typically define analysis-specific and derived objects
(selected and cross-cleaned objects, N-object combined candidates).
The decision if an the output tree should be written for an event is taken
based on the categories (instances of [`Category`](../doxyfwk/classCategory.html) implementations, see below).
In practice a typical analysis will set up (and perhaps slightly customize)
the default set of producers, define one (or more) analyzers,
and configure (or extend) categories.

Producers and analyzers can readily be understood from an example, but categories
involve more interacting components, so the overall picture may be a bit more confusing.
The [`ExTreeMaker::produce`](../doxyfwk/classExTreeMaker.html#a72a8cdffae7a6815bb96907758226220) method,
which is called for every event, first checks if the event passes all filters
(another, less frequently used, module type - the interface only specifies a method
that returns true or false for every event, without access to the producers),
and then executes the following steps:

- run all producers
- check if the event is in any of the categories, based on the producers'
  outputs only ([`Category::event_in_category_pre_analyzers`](../doxyfwk/classCategory.html#a56ad0ab7e01dda49d614885699ef52e3)), and skip to the next event if not
- run all analyzers
- check if the event is in any of the categories *that selected it before*,
  based on the analyzers' (and produces') outputs ([`Category::event_in_category_post_analyzers`](../doxyfwk/classCategory.html#a7953b521891ece15497fddd5f1e27ef5)),
  and skip to the next event if not
- save the entry in the tree

Two more things are useful to know about categories: they are register by
an analyzer (any analyzer, so one could have two analyzers that each define
some categories - the event will be in the output tree as soon as any category
from either set accepts the event), and they can also define a set of selections
(cuts). These need to be registered (in [`Category::register_cuts(cutManager)`](../doxyfwk/classCategory.html#a2ea8fab21f6d1b7a48b6813eeb8ae42d),
with [`CutManager::new_cut(name, description)`](../doxyfwk/classCutManager.html#af2790880986ac8723e10b03f1286b022)),
and set to `true`, if applicable, for every event (with [`CutManager::pass_cut(cutName)`](../doxyfwk/classCutManager.html#a0de3fda9549cd91e91d04dcd6e75a16f) -
this can be done either before or after running the analyzers (from
[`Category::evaluate_cuts_pre_analyzers`](../doxyfwk/classCategory.html#a27f95232305f83c3c807cf58a4146ee2)
or [`Category::evaluate_cuts_post_analyzers`](../doxyfwk/classCategory.html#af26360bf4b404d7bad835641313f2842), respectively).

The category responses are filled in branches with name `<prefix><categoryname>_category`,
and the cut responses in branches with name `<prefix><categoryname>_<cutname>_cut`,
where the prefix is taken from the analyzer that registered the category.

# GridIn: tools for running over grid

!!! Note

    - This guide suppose that the framework is already installed. See [the framework guide](framework.md) for detailed instructions
    - This guide installs the [GridIn](https://github.com/cp3-llbb/GridIn) tools, our [Datasets](https://github.com/cp3-llbb/Datasets) repo to list the datasets to be ran on, and [SAMADhi](https://github.com/cp3-llbb/SAMADhi), our local database
    - You probably want to have the access to this database: ask around!
    - The utilities in the ``scripts`` folders are copied to ``CMSSW/bin`` during the ``scram b``, so if these utilities have been modified you need to rebuild in order to have them in your PATH

# First time setup

```bash
source /nfs/soft/grid/ui_sl6/setup/grid-env.sh
source /cvmfs/cms.cern.ch/cmsset_default.sh
source /cvmfs/cms.cern.ch/crab3/crab.sh

cd <path_to_CMSSW>
cmsenv

cd ${CMSSW_BASE}/src
git clone -o upstream git@github.com:cp3-llbb/GridIn.git cp3_llbb/GridIn
git clone -o upstream git@github.com:cp3-llbb/SAMADhi.git cp3_llbb/SAMADhi
git clone -o upstream git@github.com:cp3-llbb/Datasets.git cp3_llbb/Datasets

scram b -j 4
cd ${CMSSW_BASE}/src/cp3_llbb/GridIn
source first_setup.sh
```

# How-to

The script you'll be working with is ``runOnGrid.py``, from the ``scripts`` folder. During the first build, this script
is copied by CMSSW into the global ``scripts`` directory, which is inside ``PATH``; you can thus access it from anywhere
in the source tree

In order to run on the grid, you need 3 things:

- First, an ``analyzer`` for the framework
- A ``configuration`` file for this analyzer
- A set of JSON files describing the datasets you want to run on

The first two points must be handled by you. For the last point, a set of JSON files for the commonly used datasets are
already included (see inside ``test/datasets``). The structure of the JSON file is described [below](#json-file-format)

You can now run on the grid. Go to the ``test`` folder, and run

```bash
runOnGrid.py -c <Your_Configuration_File> --mc datasets/mc_TT.json datasets/mc_DY.json <datasets/...>
```

``<Your_Configuration_File>`` must be substituted by the name of the configuration file, *including the ``.py`` extension*.
You should now have a new file inside the working directory, named ``crab_TTJets_TuneCUETP8M1_amcatnloFXFX_25ns.py``.
This file is a configuration file for ``crab3``. A file is created automatically for each dataset specified when running
 ``runOnGrid.py``.

!!! Note

    **By default, ``runOnGrid.py`` does not submit any jobs to the grid, it only creates the necessary files for crab.** If you want to automatically submit the jobs, you can add the ``--submit`` flag when running ``runOnGrid.py`` (**does not seems to work for the moment due to a crab bug**).

To manually launch the jobs, use the ``crab submit <crab_python_file>``. All the submitted tasks are stored inside the ``tasks`` folder.

# Book-keeping

If the job has completed successfully, you can run
```bash
runPostCrab.py <myCrabConfigFile.py>
```
This will gather the needed information (number of events, code version, source dataset, ...) and insert the sample (and possibly the parent dataset if missing) in the database

# JSON file format

Each dataset is stored inside a JSON file, containing at least the dataset pretty name, its path as well as the number
of *units* per job. The meaning of *units* depends on the type of dataset: for ``data``, a unit is a luminosity section.
For ``MC``, a unit is a file.

An example of JSON file is given below:
```json
{
  "/TTJets_TuneCUETP8M1_13TeV-amcatnloFXFX-pythia8/RunIISpring15DR74-Asympt25ns_MCRUN2_74_V9-v1/MINIAODSIM": {
    "name": "TTJets_TuneCUETP8M1_amcatnloFXFX_25ns",
    "units_per_job": 15
  }
}
```

It can contains any number of datasets, but by convention, only datasets belonging into the same group should be into
the same file (for example, it's fine to have one file for exclusive DY datasets, but not one file for all the different
TT samples). The root node must be a dictionary, where the key is the dataset path, and values are:

- ``name``: The pretty name of the dataset. This name is used to format the task name and the output path
- ``units_per_job``: For ``MC``, the number of files processed by each job. For ``data``, the number of luminosity section
processed by each job.

For a ``data`` JSON file, an additional value is mandatory:

- ``run_range``: must be an array with two entries, like ``[1, 30]``, defining the range of validity of the dataset

An optional value, but highly recommended is:

- ``certified_lumi_file``: the path (filename or url) of the golden JSON file containing certified luminosity section.
If not present, a default file will be used, presumably outdated by the time you'll run.

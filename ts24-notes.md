#  Companion

# Notes - Tim Stickland, 2018-08-24

## General

### Documentation

`README.md` updated to clarify procedure for running Companion.   Most importantly, it is simplest to
clone/download the git repo and edit the files in the local directory, and then invoke nextflow with
the name of a local directory.

e.g. if cloned/downloaded into `my_companion_project` then...
```
nextflow my_companion_project -profile docker
```

### Using local directories vs. git repo

If invoked with `sanger-pathogens/companion` instead of local directory name, the first time
it is run nextflow will clone the repo into `.nextflow/assets/sanger-pathogens`, and thereafter
it will run using that local clone (hopefully pulling to pick up repo changes??).

This repo-based usage could be really useful for collaborators using their own fork and keeping
configuration/training data/etc. in it, so they can run Companion on the latest shared version; but
it's probably too confusuing for an average user who just wants a local copy to tweak and run.

### The `dist_dir` parameter

This important parameter is commented out in the example configuration (`example-data/params_default.config`)

It defines the output directory.  It appears that without this, output isn't written at all
(useful for a dry run, though?)


## The `test-project` branch

This branch was created for testing Companaion with different data sets.  It is not intended to be
merged into the master branch or pulled into the sanger-pathogens repo!  It contains large data files,
which is obviously bad practice with git, but I'm excusing myself as I expect to delete the branch
quite soon.

### ts24 test annotation

A new data directory `ts24-test-data` was created by copying `example-data`, and replacing the
small example FASTA file with a full copy of `L_donovani.1.fasta` from Dave S.  This can be used for
a Companion run using the reference genomes distributed with the repo:  i.e. this should be the easiest
test case.

`params_default.config` was renamed `params_example-data.config`, and `params_ts24-test-data.config`
was created with a symlink `params_default.config -> params_ts24-test-data.config`: a hack to get
Companion to run with the new parameters.

Parameter changes are trivial:
```
$ diff params_example-data.config params_ts24-test-data.config
3c3
<     inseq = "${baseDir}/example-data/L_donovani.1.fasta"
---
>     inseq = "${baseDir}/ts24-test-data/L_donovani.1.fasta"
6c6
<     ref_dir = "${baseDir}/example-data/references"
---
>     ref_dir = "${baseDir}/ts24-test-data/references"
10c10
<     dist_dir = "${baseDir}/example-data-output"
---
>     dist_dir = "${baseDir}/ts24-test-data-output"
14c14
<     run_snap               = false
---
>     run_snap               = true
```

###  _E. histolytica_ annotation.

#### Adding reference data


_Entamoeba histolytica_ reference genome (`EhistolyticaHM1IMSS_Genome.fasta`)
and reference data (`EhistolyticaHM1IMSS.zip`) were supplied by Dave S. 

The reference data were prepared as documented in
https://github.com/sanger-pathogens/companion/wiki/Preparing-reference-data-sets, but HMM files for
SNAP were _not_ provided.  How to do this is not documented on the wiki page, though it is also not
clear whether good results can be attained using SNAP anyway.  This means that
_Companion will exit with an error unless_ `run_snap = false` _is set in the params file._

Workflow for adding reference:

- Create new data directory, `amber-test-data`
- Copy reference genome into `amber-test-data/genomes`
- Copy GFF (x2) and GAF files into `amber-test-data/genomes`
- Copy remainng files into `data/augustus/species/entamoeba_histolytica/`
- Create new directory `amber-test-data/references/Ehis`
- Add new `Ehis` (same as subdirectory name, above) section to `amber-test-data/references/references-in.json`;
this provided the names/paths of the files copied (above), a descriptive name, and
a pattern for matching chromosomes in the FASTA files (in this case, Ehis_1 through to Ehis_563).
```
"Ehis" : {
      "gff"                : "../genomes/EhistolyticaHM1IMSS.gff3",
      "genome"             : "../genomes/EhistolyticaHM1IMSS_Genome.fasta",
      "gaf"                : "../genomes/EhistolyticaHM1IMSS_GO.gaf",
      "name"               : "Entamoeba_histolytica HM1IMSS",
      "augustus_model"     : "../../data/augustus/species/entamoeba_histolytica/",
      "chromosome_pattern" : "Ehis_(%d+)"
    }
```
- Finally, change directory to `amber-test-data/references` (_must_ work in this directory)
and run `../../bin/update_references.lua`.  This writes `amber-test-data/references/references.json`.

#### The annotation run

New assembly `Ehis_amber.fasta` (via Dave S.) copied into `amber-test-data`.

Added a symlinked params file `params_default.config -> params_amber-test-data.config`.
Parameters were defined by Dave S., tweaked by me to include `dist_dir = "${baseDir}/amber-test-data-output"`


## TO DO

- Define `dist_dir` parameter in the example data configuration file so that output is created if Companion is
run on these data, and/or document this parameter in `README.md`; merge into sanger-pathogens repo.

- Find proper way to switch configuration instead of symlinking `params_default.config` to separate config file! 

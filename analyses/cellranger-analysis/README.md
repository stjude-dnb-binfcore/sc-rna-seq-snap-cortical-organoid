# Pipeline for running and summarizing Cell Ranger count for single or multiple libraries for sc-/sn-RNA-Seq Analysis in 10X Genomics data

## Usage

`submit-multiple-jobs.sh` is designed to be run as if it was called from this module directory even when called from outside of this directory.

The `submit-multiple-jobs.sh` script is designed to run the following two steps: 
   - Step 1: To run the `j1.sh` script to align single or multiple libraries in parallel, i.e., `run-cellranger-analysis`. 
   - Step 2: To run `j2.sh` to summarize alignment results, i.e., `summarize-cellranger-analysis`. The latter script will be on hold and executed once all libraries are aligned and `j1.sh` is complete. This is been taken care of by `waiter.sh` script.

Parameters according to the project and analysis strategy will need to be specified in the following scripts:
- `../../project_parameters.Config.yaml`: define the `metadata_dir`, `genome_reference_path`, `cellranger_parameters`, and `genome_name_cellranger`. Please submit an [issue](https://github.com/stjude-dnb-binfcore/sc-rna-seq-snap/issues) to request the path to the reference genome of preference. Our team at the Bioinformatics core at DNB maintains the following genome references: (1) human: `GRCh38`, `hg19`,  and `GRCh38_GFP_tdTomato`; (2) mouse: `GRCm39`, `mm10` and `mm9`; and (3) dual index genomes: `GRCh38ANDGRCm39`, `GRCh38_mm10`, and `GRCh38_mm9`. Otherwise, specify the path to the reference genome of your preference. 

Moreover, for non-human experiments, we recommend setting `create_bam_value` to `false` to reduce memory usage per project. 

User also need to define `sample_prefix` with the Sample ID used for the samples of the project. This requires that all of the Sample IDs are the same, e.g., DYE001, DYE002, DYE003 and so on. 

- `j1.sh`: 
  - `--force_cells`: User can add flags as necessary for their analyses and compare alignments with CellRanger, e.g., by using `--force_cells=8000` to constraint the number of cells to be used for alignment. We recommend to run by default, and after careful assessment to edit parameters. We have found that the default parameters set up here work well for most of the cases.

- `j2.sh`: 
   - In case of dual genomes in the experiment, user will need to add the `--genome ${genome_name_cellranger}` flag to summarize results generated by CellRanger for all libraries.
   

Default memory for running the alignment is set up to 16GB. In case more memory is needed to run a specific sample, this can be modified here:
- `./util/run_cellranger.py`: User can search the following and change accordingly `-n 4 -R "rusage[mem=4000] span[hosts=1]`, if that is necessary. However, we have found that these resources work well for most data. 


### Handling Technical Replicates in Cell Ranger Count

If a sample has multiple technical replicates (i.e., multiple sequencing runs of the same library), list all corresponding FASTQ file paths in the same row of the metadata file, separated by commas.

For example:


| ID | SAMPLE | FASTQ | 
:----------|:----------|:----------|
| DYE001 | seq_submission_code1_sample1 | /absolute_path/seq_submission_code1/replicate1,/absolute_path/seq_submission_code1/replicate2 | 


Cell Ranger will automatically treat these as technical replicates and merge the data into a single output for that sample during processing. There is no need to manually combine or rename the files—just list them correctly, and the pipeline takes care of the rest.


## Run module on HPC

To run all of the scripts in this module sequentially on an interactive session on HPC, please run the following command from an interactive compute node:

```
bsub < submit-multiple-jobs.sh
```

Please note that this will run the analysis module outside of the container while submitting lsf job on HPC. This is currently the only option of running the `cellranger-analysis` module. By default, we are using `python/3.9.9` and `cellranger/8.0.1` as available on St Jude HPC.


## Folder content
This folder contains scripts tasked to run and summarize Cell Ranger count for single or multiple libraries for sc-/sn-RNA-Seq Analysis in 10X Genomics data across the project. For more information and updates, please see [Cell Ranger support page](https://www.10xgenomics.com/support/software/cell-ranger/latest/analysis/running-pipelines/cr-gex-count).

This module uses CellRanger v8.0.1 for the alignment.


## Folder structure 

The structure of this folder is as follows:

```
├── j1.sh
├── j2.sh
├── README.md
├── results
|   ├── 01_logs
|   ├── 02_cellranger_count
|   |   └── DefaultParameters
|   └── 03_cellranger_count_summary
├── submit-multiple-jobs.sh
├── util
|   └── summarize_cellranger_results.py
└── waiter.sh
```
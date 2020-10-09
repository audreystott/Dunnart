# Falcon

## Description

| Workflow element | GitHub | bio.tools | BioContainers | bioconda |
|-------------|:--------:|:--------:|:--------:|:--------:|
|Pacific Biosciences assembly tool suite v0.0.8 |[&#9679;](https://github.com/PacificBiosciences/pb-assembly)||[&#9679;](https://biocontainers.pro/#/tools/pb-falcon)|[&#9679;](https://bioconda.github.io/recipes/pb-assembly/README.html)|

The [Pacific Biosciences assembly tool suite](https://github.com/PacificBiosciences/pb-assembly) covers the three main functions of Falcon; 

1. fc_run, 
2. fc_unzip, and 
3. fc_phase

## Required (minimum) inputs/parameters

- As per parameter settings in each .cfg file

## Third party tools / dependencies

Requires Conda and Nextflow to run on Zeus. 

### Input(s)

#### Data types:
Fasta (fc_run), bam files (fc_unzip), hi-C files (fc_phase). This release of the pipeline does not support hifi data, but this will come soon. Hi-C data it optional. If you only have bam files, but no fasta files, you can use https://github.com/PacificBiosciences/bam2fastx to convert them. 

#### pb-assembly specific files required:
subreads.fasta.fofn: list of fasta files for analysis. All names must be on one line.

subreads.bam.fofn: list of bam files for analysis. All names must be on *seperate* lines.

subreads.hi-c.fofn: list of hi-c files for analysis. All names must be on one line.

fc_run.cfg, fc_unzip.cfg, and fc_phase.cfg: files that specify the parameters for pb-assembly

### Parameter(s)
Assembly parameters are altered via the .cfg files. 
See https://github.com/PacificBiosciences/pb-assembly for details. 

### Output(s)
pb-assembly outputs many files, which can be used for quality checking as seen the tutorial section. However, the final outputs are 5-phase/output/phased.0.fasta and 5-phase/output/phased.1.fasta. 

# Diagram

# Usage

## Summary

| Workflow Tool |  Version | Infrastructure | Scheduler | Workflow manager | Container | Install method |
|---------------|---------|---------|-----------|------------------|-----------|----------------|
|Pacific Biosciences assembly tool suite v0.0.8|Pre-release 2nd July 2020|[Zeus, Pawsey](https://www.pawsey.org.au/systems/zeus)|SLURM|Nextflow (v19.10.0)|Singularity (v3.5.2)|Miniconda3 - this environment will be activated from the workflow install. Independent install is not required.|

## High level resource usage

Table with embedded Compute infrastructure name optimisation -> "HPC-HTC" column

| Workflow | Version | Group | Sample name (e.g. organism) | Other sample detail (e.g. *Genus species*) | Other sample detail (e.g. genome size (GB)) | Hours required | Cores | Peak RAM in GB (requested) | Drive (GB) | HPC-HTC | Month-Year |
|---------|---------|-------|---------------|---------------|------------------|----------------|-------|----------------------------|---------------|---------|------------|
|||||||||||||

## Additional notes

Any comment on major features being introduced, or default/API changes that might result in unexpected behaviours.

# Install

The following steps cover installation of the tools required for performing the *de novo* assembly of the Fat-tailed Dunnart genome using Falcon on Pawsey's HPC system Zeus. The original Falcon documentation that this workflow is derived from can be found [here](https://github.com/PacificBiosciences/pb-assembly#tutorial).

## Start an interactive SLURM session
On Zeus, you will need to run your jobs on a work node; to do so, start an interactive session as follows:

    >salloc -n 1 -t 1:00:00

## Clone the Dunnart repository

The Dunnary repository contains the scripts and config files required for running the workflow. Clone it to your current working directory, e.g. /group/$PAWSEY_PROJECT/$USER

    >cd /group/$PAWSEY_PROJECT/$USER
    >git clone https://github.com/audreystott/dunnart.git
    >cd dunnart

## Download your raw sequencing data

You will need both the fasta and bam formats of the raw sequencing data. Download them to your current working directory, which should be the `dunnart/` directory.

## Set the fasta and bam filenames in the run script

Ensure the filenames are set in the `falcon-conda.sh` script. Do so by replacing the words `YOUR_FASTA_FILE_NAME` and `YOUR_BAM_FILE_NAME` with your raw filenames for the below commands. 

Note: Your fasta and bam filenames should have a suffix .subreads.fasta.gz and .subreads.bam, respectively.

    >sed -i "s|F1_bull_test.subreads.fasta.gz|YOUR_FASTA_FILE_NAME|g" falcon-conda.sh
    >sed -i "s|F1_bull_test.subreads.bam|YOUR_BAM_FILE_NAME|g" falcon-conda.sh 

## Set the HiC filename in the Nextflow script

Set your HiC filename in the Nextflow `main.nf` script. 

Note: The files should have the suffixes .HiC_R1.fastq.gz and .HiC_R2.fastq.gz.

You will replace the words `YOUR_HIC_FILE_NAME` with your HiC filename for the below. It should look similar to this - `sample1.HiC_R*.fastq.gz`.

    >sed -i "s|F1_bull_test.HiC_R*.fastq.gz|YOUR_HIC_FILE_NAME|g" main.nf

## Set up FALCON

Run the FALCON script falcon-conda.sh.

    >bash falcon-conda.sh    

## Exit the SLURM interactive session

Once the FALCON set up script has completed running, exit the session.

    >exit

# Tutorials

## Run FALCON

Run FALCON with the sbatch script as follows:

    >sbatch --account=$PAWSEY_PROJECT sbatch_nextflow.sh

## Check job progress

While FALCON is running, you can check the progress of your jobs.

### fc_run

Jobs left:
    
    >ls 0-rawreads/daligner-chunks/ | wc -l

Jobs completed:

    >find 0-rawreads/daligner-runs/j_*/uow-00 -name "daligner.done" | wc -l

Stats for reads and pre-assembled reads:

    >singularity exec pb-assembly_0.0.8.sif DBstats 0-rawreads/build/raw_reads.db

    >singularity exec pb-assembly_0.0.8.sif DBstats 1-preads_ovl/build/preads.db 

Check pre-assembly performance:

    >cat 0-rawreads/report/pre_assembly_stats.json

Check assembly performance:

    >python get_asm_stats.py 2-asm-falcon/p_ctg.fasta

### fc_unzip 

Check haplotype resolution

    >python get_asm_stats.py 3-unzip/all_p_ctg.fasta 

    >python get_asm_stats.py 3-unzip/all_h_ctg.fasta

    >head 3-unzip/all_h_ctg.paf 

Check phase polishing

    >python get_asm_stats.py 4-polish/cns-output/cns_p_ctg.fasta
   
    >python get_asm_stats.py 4-polish/cns-output/cns_h_ctg.fasta


### fc_phase

See haplotig placement file
   
    >head 5-phase/placement-output/haplotig.placement

See final output stats 

    >python get_asm_stats.py 5-phase/output/phased.0.fasta

    >python get_asm_stats.py 5-phase/output/phased.1.fasta 

# Help / FAQ / Troubleshooting

## Note 1
We encountered a bug in the 2-asm_falcon ovlp_filtering stage, where preads.m4 had an erroneous '---' at the end of the file. We fixed this by following this github issue: https://github.com/PacificBiosciences/pbbioconda/issues/294. This step is now automatically taken care of in the nextflow pipeline.

## Note 2
The fofn files need to have all entries on one line.

## Troubleshooting
The `all.log` file is useful for checking job progress and identifying which step(s) caused the workflow to fail if troubleshooting is required. This file should be used to guide you to the detailed stderr file for the failed step(s). The detailed log files for each step are in located in the sub-directories of each process (e.g. `nf-work/<nextflowID1>/<nextflowID2>/0-rawreads/build/run-Pccabfacd84af34.bash.strerr`). The slurm log file contains only the nextflow standard output/error, which is generally not very verbose. 

# License(s)

- [Licence on pbbioconda](https://github.com/PacificBiosciences/pbbioconda/blob/master/LICENSE.md)

# Acknowledgements / citations / credits

This workflow is for the reference genome assembly of 
the Fat-tailed Dunnart ([*Sminthopsis crassicaudata*](https://bie.ala.org.au/species/urn:lsid:biodiversity.org.au:afd.taxon:86ab9ebf-cbd1-49f8-9786-312407738477), as part of an [Australian Biocommons](https://www.biocommons.org.au/) collaboration. Raw sequences were generated by Stephen Frankenberg (University of Melbourne) as part of the [Oz Mammals Genomics](https://ozmammalsgenomics.com/) (OMG) Framework Initiative, and downloaded from the Bioplatforms Australia data portal. The Initiative is supported by funding from [Bioplatforms Australia](https://bioplatforms.com/) through the Australian Government National Collaborative Research Infrastructure Strategy (NCRIS). 

The first release is credited to Pawsey Supercomputing Centre in collaboration with the Australian Biocommons.
# Description
This workflow is for the reference genome assembly of 
the Fat-tailed Dunnart ([*Sminthopsis crassicaudata*](https://bie.ala.org.au/species/urn:lsid:biodiversity.org.au:afd.taxon:86ab9ebf-cbd1-49f8-9786-312407738477), as part of an [Australian Biocommons](https://www.biocommons.org.au/) collaboration. Raw sequences were generated by Stephen Frankenberg (University of Melbourne) as part of the [Oz Mammals Genomics](https://ozmammalsgenomics.com/) (OMG) Framework Initiative, and downloaded from the Bioplatforms Australia data portal.

The tools used in this workflow are:
- [Pacific Biosciences assembly tool suite](https://github.com/PacificBiosciences/pb-assembly)
- Nextflow v19.10.0
- Singularity v3.5.2
- Miniconda3

## Installation and set up
The following steps cover installation of the tools required for performing the de novo assembly of the fat-tailed Dunnart genome using Falcon on Pawsey's HPC system Zeus. The original Falcon documentation that this workflow is derived from can be found [here](https://github.com/PacificBiosciences/pb-assembly#tutorial).

### Start an interactive SLURM session
On Zeus, you will need to run your jobs on a work node; to do so, start an interactive session as follows:

    >salloc -n 1 -t 1:00:00

### Clone the Dunnart repository

The Dunnary repository contains the scripts and config files required for running the workflow. Clone it to your current working directory, e.g. /group/$PAWSEY_PROJECT/$USER

    >cd /group/$PAWSEY_PROJECT/$USER
    >git clone https://github.com/audreystott/dunnart.git
    >cd dunnart

### Download your raw sequencing data

You will need both the fasta and bam formats of the raw sequencing data. Download them to  your current working directory, which should be the `dunnart/` directory.

### Set the fasta and bam file variables in the run script

In the cloned repository, you will find a run script `falcon-conda.sh`. You will need to replace the words `YOUR_FASTA_FILE_NAME` and `YOUR_BAM_FILE_NAME` with your raw filenames for the below. Your fasta and bam filenames should have a suffix .subreads.fasta.gz and .subreads.bam, respectively.

    >sed -i "s|dunnart-fasta|YOUR_FASTA_FILE_NAME|g" falcon-conda.sh
    >sed -i "s|dunnart-bam|YOUR_BAM_FILE_NAME|g" falcon-conda.sh 

### Set the HiC files variable in the Nextflow script

Your HiC filenames should have the suffix .HiC_R1.fastq.gz or .HiC_R2.fastq.gz. 

For example, if you have my_sample.HiC_R1.fastq.gz and my_sample.HiC_R2.fastq.gz. You will replace the words `YOUR_HIC_FILE_NAME` with my_sample.HiC_R*.fastq.gz for the below.

    >sed -i "s|F1_bull_test.HiC_R*.fastq.gz|YOUR_HIC_FILE_NAME|g" main.nf

### Set up FALCON

Run the FALCON script falcon-conda.sh.

    >bash falcon-conda.sh    

### Exit the SLURM interactive session

Once the FALCON set up script has completed running, exit the session.

    >exit

## Run FALCON

Run FALCON with the sbatch script as follows:

    >sbatch --account=$PAWSEY_PROJECT sbatch_nextflow.sh

## Tutorial
### Check job progress

While FALCON is running, you can check the progress of your jobs.

#### fc_run

Jobs left:
    
    >ls 0-rawreads/daligner-chunks/ | wc -l

Jobs completed:

    >find 0-rawreads/daligner-runs/j_*/uow-00 -name "daligner.done" | wc -l

Stats for reads and pre-assembled reads:

    >singularity exec pb-assembly_0.0.8.sif DBstats raw_reads.db

    >singularity exec pb-assembly_0.0.8.sif DBstats 1-preads_ovl/build/preads.db 

Check pre-assembly performance:

    >cat 0-rawreads/report/pre_assembly_stats.json

Check assembly performance:

    >singularity exec pbcore_1.7.1--py27_0.sif python pb-assembly/scripts/get_asm_stats.py 2-asm-falcon/p_ctg.fasta

#### fc_unzip 

Check haplotype resolution

    >singularity exec pbcore_1.7.1--py27_0.sif python pb-assembly/scripts/get_asm_stats.py 3-unzip/all_p_ctg.fa 

    >singularity exec pbcore_1.7.1--py27_0.sif python pb-assembly/scripts/get_asm_stats.py 3-unzip/all_h_ctg.fa

    >head 3-unzip/all_h_ctg.paf 

Check phase polishing

    >singularity exec pbcore_1.7.1--py27_0.sif python pb-assembly/scripts/get_asm_stats.py 4-polish/cns-output/cns_p_ctg.fasta
   
    >singularity exec pbcore_1.7.1--py27_0.sif python pb-assembly/scripts/get_asm_stats.py 4-polish/cns-output/cns_h_ctg.fasta


#### fc_phase

See haplotig placement file
   
    >head 5-phase/placement-output/haplotig.placement

See final output stats 

    >singularity exec pbcore_1.7.1--py27_0.sif python pb-assembly/scripts/get_asm_stats.py 5-phase/output/phased.0.fasta

    >singularity exec pbcore_1.7.1--py27_0.sif python pb-assembly/scripts/get_asm_stats.py 5-phase/output/phased.1.fasta 

## Workflow infrastructure requirements
- Pawsey [Zeus](https://www.pawsey.org.au/systems/zeus)
- More to add

## Acknowledgements
We would like to acknowledge the contribution of the Oz Mammals Genomics Initiative [consortium](https://ozmammalsgenomics.com/consortium/) in the generation of data used for the Dunnart reference genome assembly. The Initiative is supported by funding from Bioplatforms Australia through the Australian Government National Collaborative Research Infrastructure Strategy (NCRIS).

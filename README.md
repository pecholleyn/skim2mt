# skim2phylo  

**skim2phylo** is a snakemake pipeline for the batch assembly, annotation, and phylogenetic analysis of mitochondrial genomes and ribosomal genes from low coverage "genome skims". The pipeline was designed specifically to work with sequence data from museum collections. 

An additional pipeline **gene2phylo** is provided which can be used to implement further phylogenetic analysis for set of aligned genes, using individual genes, a partitioned gene alignment and astral.

## Setup 

The pipeline is written in Snakemake and uses conda environments and singularity images to install the necessary tools. 

It is reccomended to install conda using Mambaforge. See details here https://snakemake.readthedocs.io/en/stable/getting_started/installation.html 

Once conda is installed, you can pull the github repo and set up the base conda environment.

```
# get github repo
git clone https://github.com/o-william-white/skim2phylo

# change dir
cd skim2phylo

# setup conda env
conda env create -n skim2phylo -f envs/conda_env.yaml
```

## Example usage

Before you run skim2phylo on your own data, it is recommended to run at least one of the example datasets provided. This will confirm there are no user-specific issues with the setup and it also installs all the dependencies. 

The example data includes simulated mitochondrial and ribosomal reads from 21 different butterfly speices. The example below will only analyse a smaller subset of 7 samples to reduce computation time.  

### Mitochondrial example

To run the mitochondrial example data, run the code below. *Note that the first time you run this, it will take some time to install each of the conda environments*, so it is a good time to take a tea break :).
```
source activate skim2phylo

snakemake \
   --snakefile skim2phylo.smk \
   --cores 6 \
   --use-conda \
   --use-singularity \
   --configfile example_data/config_mitochondrion.yaml \
   --rerun-incomplete
``` 

If you have access to a HPC, you can submit snakemake workflows as jobs. For example, using sbatch:
```
sbatch --cpus-per-task=24 --mem=30G run_mitochondrion_example.sh
```

Snakemake requires a config.yaml and samples.csv to define input paramters and sequence data for each sample. For the mitochondrion example data provided, the config file is located here `example_data/config_mitochondrion.yaml` and it looks like this: 
```
# path to sample sheet csv with columns for ID,forward,reverse
samples: example_data/samples_7_mitochondrion.csv

# name of output directory
output_dir: results_mitochondrion_example

# fastp depulication (True/False)
fastp_dedup: True

# GetOrganelle target sequence type (mitochondrion, ribosomal)
target_type: mitochondrion

# mitos refseq database (refseq39, refseq63f, refseq63m, refseq63o, refseq89f, refseq89m, refseq89o)
mitos_refseq: refseq39

# mito code (2 = Vertebrate, 4 = Mold, 5 = Invertebrate, 9 = Echinoderm, 13 = Ascidian, 14 = Alternative flatworm
mitos_code: 5

# barrnap kindgom (Bacteria:bac, Archaea:arc, Eukaryota:euk, Metazoan Mitochondria:mito)
barrnap_kingdom: NA

# alignment trimming method to use (gblocks or clipkit)
alignment_trim: gblocks

# alignment missing data threshold for alignment (0.0 - 1.0)
missing_threshold: 0.5

# name of outgroup sample (optional)
# use "NA" if there is no obvious outgroup
# if more than one outgroup use a comma separated list i.e. "sampleA,sampleB"
outgroup: "Eurema_blanda"

# plot dimensions (cm)
plot_height: 20
plot_width: 20

# number of threads to use
threads: 6
```

The example samples.csv file is located here `example_data/samples_mitochondrion.csv` and the first 10 lines look like  this: 

| ID | forward | reverse | seed | gene |
|----|---------|---------|------|------|
| Eurema_blanda     | example_data/mitochondrion/Eurema_blanda_1.fq.gz     | example_data/mitochondrion/Eurema_blanda_2.fq.gz     | example_data/seed_mitochondrion.fasta | example_data/gene_mitochondrion.fasta |
| Junonia_villida   | example_data/mitochondrion/Junonia_villida_1.fq.gz   | example_data/mitochondrion/Junonia_villida_2.fq.gz   | example_data/seed_mitochondrion.fasta | example_data/gene_mitochondrion.fasta |
| Kallima_paralekta | example_data/mitochondrion/Kallima_paralekta_1.fq.gz | example_data/mitochondrion/Kallima_paralekta_2.fq.gz | example_data/seed_mitochondrion.fasta | example_data/gene_mitochondrion.fasta |
| Litinga_cottini   | example_data/mitochondrion/Litinga_cottini_1.fq.gz   | example_data/mitochondrion/Litinga_cottini_2.fq.gz   | example_data/seed_mitochondrion.fasta | example_data/gene_mitochondrion.fasta |
| Mallika_jacksoni  | example_data/mitochondrion/Mallika_jacksoni_1.fq.gz  | example_data/mitochondrion/Mallika_jacksoni_2.fq.gz  | example_data/seed_mitochondrion.fasta | example_data/gene_mitochondrion.fasta |
| Parasarpa_zayla   | example_data/mitochondrion/Parasarpa_zayla_1.fq.gz   | example_data/mitochondrion/Parasarpa_zayla_2.fq.gz   | example_data/seed_mitochondrion.fasta | example_data/gene_mitochondrion.fasta |
| Precis_pelarga    | example_data/mitochondrion/Precis_pelarga_1.fq.gz    | example_data/mitochondrion/Precis_pelarga_2.fq.gz    | example_data/seed_mitochondrion.fasta | example_data/gene_mitochondrion.fasta |

## Main output files

Below is a table summarising all of the output files generated by the pipeline.

| Directory             | Description                                                                                            |
|-----------------------|--------------------------------------------------------------------------------------------------------|
| fastqc                | Fastqc reports for input reads                                                                         |
| fastp                 | Fastp reports for input reads                                                                          |
| getorganelle          | GetOrganelle output with a directory for each sample                                                   |
| assembled_sequence    | Assembled sequences selected from GetOrganelle output and renamed                                      |
| seqkit                | Seqkit summary of the assembly                                                                         |
| blastn                | Blastn output of each assembly                                                                         |
| minimap               | Mapping output of quality filtered reads against each assembly                                         |
| blobtools             | Blobtools assembly summary collating blastn and mapping output                                         |
| assess_assembly       | Plots of annotations, mean depth, GC content and proportion mismatches                                 |
| annotations           | Annotation outputs of mitos or barrnap depending on target type                                        |
| summary               | Summary per sample (seqkit stats) and contig (GC content, length, coverage, taxonomy and annotations)  |
| protein_coding_genes  | Unaligned fasta files of annotated genes identified across all samples                                 |
| mafft                 | Mafft aligned fasta files of annotated genes identified across all samples                             |
| mafft_filtered        | Mafft aligned fasta files after the removal of sequences based on a missing data threshold             |
| alignment_trim        | Ambiguous parts of alignment removed using either gblocks or clipkit                                   |
| iqtree                | Iqtree phylogenetic analysis of annotated genes                                                        |
| plot_tree             | Plots of phylogenetic trees                                                                            |
| logs                  | Log files for each step in pipeline                                                                    |


### Ribosomal example

To run the ribosomal example data. run the code below. 
```
snakemake \
   --snakefile skim2phylo.smk \
   --cores 6 \
   --use-conda \
   --use-singularity \
   --configfile example_data/config_ribosomal.yaml \
   --rerun-incomplete
```
As above, the ribosomal example can be submitted using sbatch
```
sbatch --cpus-per-task=24 --mem=30G run_ribosomal_example.sh
```


### Filtering putative contaminants 

If you are working with museum collections, it is possible that you may assemble and annotate sequences from contaminant/non-target species. *Contaminant sequences can be identfied based on the blast search output or unusual placement in the phylognetic trees* (see blobtools and plot_tree outputs). 

A python script is provided to remove putative contaminants, and generate the files required for a final phylogenetic analysis. For example, lets say we wanted to remove all sequences from the sample named "Kallima_paralekta" and 5.8S ribosomal sequence, you could run the script as follows: 

```
python scripts/remove_contaminants.py \
   --input results_mitochondrion_example/mafft_filtered/ results_ribosomal_example/mafft_filtered/ \
   --cont Kallima_paralekta 5.8S \
   --output results_genes_example \
   --overwrite
```

### Re-running phylogenetic analysis

If you had to remove any putative contaminants, you will now have a directory containing a combination of mitochondrial and ribosomal genes across multiple samples. The next step is to repeat the phylogenetic analysis. The supplementory pipeline **gene2phylo** will generate individual gene trees and a partitioned gene tree using iqtree and a species tree using astral.

To run the ribosomal example data. run the code below.
```
snakemake \
   --snakefile gene2phylo.smk \
   --cores 4 \
   --use-conda \
   --configfile example_data/config_genes.yaml \
   --rerun-incomplete
```
As above, the example can be submitted using sbatch
```
sbatch --cpus-per-task=24 --mem=10G run_genes_example.sh
```

### Barcode only 

If you are only interested in the assembly of genes and potentially barcode sequences without the phylogenetic analysis, you can stop the pipeline after the extraction of gene sequences. For example using the `--until` parameter. 

```
snakemake \
   --snakefile skim2phylo.smk \
   --cores 6 \
   --use-conda \
   --use-singularity \
   --configfile example_data/config_mitochondrion.yaml \
   --rerun-incomplete \
   --until extract_protein_coding_genes
```
  
### Running you own data

You can generate your own config.yaml and samples.csv files. 

GetOrganelle requires reference data in the format of seed and gene reference fasta files. You can use the default reference data for GetOrganelle, but I would recomend using custom reference databases where possible. See here for details of how to set up your own databases https://github.com/Kinggerm/GetOrganelle/wiki/FAQ#how-to-assemble-a-target-organelle-genome-using-my-own-reference 

I have shared a basic python script called go_fetch.py in another repo https://github.com/o-william-white/go_fetch to download and format reference data formatted for GetOrganelle. Go fetch downloads the reference from NCBI using biopython, removes repetitive sequences using trf, and formats the data for GetOrganelle.

If you have any questions, please do get in touch in the issues or by email o.william.white@gamil.com


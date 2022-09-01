# Workflow Metagenomics
# Tim Reska 

The following github presents a pipeline for the assembly of transcriptomes based on short or long reads as well as 
combined approach.


# Applied Sofwarte
## basecalling 
-guppy basecaller

```shell
$ guppy_basecaller -i /input_directory_fast5 -s /output_directory -c <configuration_filge.cfg> [options]
```
The configuration file comprises the used nanopore device and the corresponding kit


## Adapter Removal
-Porechop

```shell
$ porechop -i <fastq_output_guppy.fastq> -o <porechopped.fastq> 
```
The above command will use the default values of porechop to search for adapters in the input fastq file, trim the reads and write them to file porechopped.fastq

## Filtering
-NanoFilt

```shell
$ NanoFilt -q 7 -l 200 <porechopped.fastq> > <nanofiltered.fastq>
```
Removes all reads with a read quality score below 7 and shorter than 200bp

## Assembly
-Flye

```shell
$ flye --meta --nano-hq <input.fastq> -o /output_dir --genome-size 1m -i 7
```
The above command uses the high-quality polished nanopore reads and assembles the genome. The estimated --genome-size is an obligatory input. The --meta option is for metagenomic approaches. 

## Polishing
-minimap2

```shell
$ minimap2 -ax map-ont <flye_assembly.fasta> <reads.fastq> > <output.sam> -j 10
```
-Racon
-Medaka

## Binning
-MetaBat2
-MaxBin2
-Vamb

## Bin Refining
-DAS Tool

## Bin coverage
-CoverM

## Bin QC
-CheckM

## Bin Classification
-GDTB-Tk

## Bin Clustering
-dRep

## Downstream Analysis
## Sorting program
-samtools applied on both to sort the reads by coordinates


## Guppy - Translating electrices signal into nucleotides

Required input: fast5 files and the configuration file that sets the basecalling parameters depending on which flowcell and library preparation kit was used to produce the data

```guppy_basecaller -c dna_r9.4.1_450bps_hac.cfg -x 'auto' --recursive -i <input fast5 dirsctory> -s <output directory>```

R10.4 paper commands: The generated raw Nanopore data were basecalled in super-accurate mode using Guppy v. 5.0.16 with the dna_r9.4.1_450bps_sup.cfg model for R9.4.1 and the dna_r10.4_e8.1_sup.cfg model for R10.4 chemistry

## Porechop - Adapter trimming

The known Nanopore adapters that Porechop looks for are defined in the adapters.py file

Basic adapter trimming: 

```porechop -i input_reads.fastq.gz -o output_reads.fastq.gz```


## NanoFilt - Read trimming and filtering

NanoFilt to trim and filter MinION reads; NanoFilt does not provide options for input or output files. Therefore we will use the two redirect operators “>” and “<“ to

```NanoFilt –l 500 --headcrop 10 < porechopped.fastq > nanofilt_trimmed.fastq```

R10.4 paper: reads with a lower length than 200 bp and a Phred quality score below 7 and 10 for R9.4.1 and R10.4 reads, respectively

## Flye assembler - de novo assembler

Flye is a de novo assembler for single-molecule sequencing reads, such as those produced by PacBio and Oxford Nanopore Technologies

```flye --nano-hq inputfile.fa --threads N --meta```

R10.4 Paper: --meta --nano-hq
For Q20 data, use a combination of --nano-hq and --read-error 0.03

## Minimap2 - Mapping / Polishing

Takes the original ONT reads as input and realigns them to the assembly generated by Flye

```minimap2 -ax asm5 ref.fa asm.fa > aln.sam```


## Racon - Error Correction / Polishing 

Provides a fasta consensus algorithm that uses either 2nd generation short reads or raw noisy long-reads to correct draft assemblies

Use minimap2 to map the trimmed nanofilt reads back to the flye assembly and then use the filtered reads and the mapping to build the consensus assembly.

```racon [options ...] <sequences> <overlaps> <target sequences>```

    
sequences: input file in FASTA/FASTQ format (can be compressed with gzip) containing sequences used for correction

overlaps: input file in MHAP/PAF/SAM format (can be compressed with gzip) containing overlaps between sequences and target sequences

target sequences: input file in FASTA/FASTQ format (can be compressed with gzip) containing sequences which will be corrected
     

## Medaka - Error Correction / Polishing 


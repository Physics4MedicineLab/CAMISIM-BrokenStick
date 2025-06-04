# CAMISIM-BrokenStick

## Overview

**CAMISIM-BrokenStick** introduces a new simulation modality as an extension of the original *de novo* mode implemented in [CAMISIM](https://github.com/CAMI-challenge/CAMISIM) (Fritz et al., 2019). This new modality enables simulation of microbial communities where both strain-level diversity and a user-defined population structure are required.

Like other CAMISIM modes, this modality supports the generation of new strains via `sgEvolver`. Its distinguishing feature is how community composition is defined: rather than randomly sampling from a log-normal distribution, **CAMISIM-BrokenStick** takes a user-specified distribution of input genomes and redistributes their abundances among simulated strains using the **broken stick model**.

## Documentation

Full documentation for CAMISIM is available [here](https://github.com/CAMI-challenge/CAMISIM/wiki/User-manual).

This section describes the additions introduced in this **BrokenStick** modality.

## Purpose of the **BrokenStick** Modality

The **BrokenStick** mode addresses a key limitation in existing metagenomic simulators: the inability to combine predefined population structures with strain-level diversity. CAMISIM traditionally operates in two modes:

* **from profile**: accepts an external abundance profile but does not simulate strains.
* **de novo**: simulates strains via `sgEvolver`, but generates community composition via random log-normal sampling.

**CAMISIM-BrokenStick** bridges these two paradigms: it belongs to the *de novo* category but replaces log-normal sampling with a user-defined abundance file. The resulting community structure reflects both the input distribution and strain-level reshaping via the broken stick model.

### Relation to CAMISIM's Existing Modes

The original *de novo* modes in CAMISIM include four possible approaches to re-distribute the abundances:

* `differential`
* `replicates`
* `timeseries_normal`
* `timeseries_lognormal`

All these rely on log-normal sampling and differ only in how multiple samples are generated. Their behavior is indistinguishable when inspecting a single-sample output. In contrast, **CAMISIM-BrokenStick** directly uses user-specified abundances and redistributes them over generated strains.

The redistribution step uses the **broken stick model**, implemented via a Beta distribution-based sampling. Details are provided [below](https://github.com/Physics4MedicineLab/CAMISIM-BrokenStick#redistribution-of-abundances).

## Input: Abundance File Format

The **abundance file** (`abundance.tsv`) is a two-column, tab-separated file with no header:

```
<genome_ID>    <relative_abundance>
```

* `genome_ID` must match identifiers in `metadata.tsv` and `genome_to_id.tsv`.
* `relative_abundance` must reflect the target abundance of each input genome.

Two usage modes are supported:

1. **Subset mode**: include only genomes of interest.
2. **Full input mode**: list all input genomes, with unwanted ones set to 0 at the bottom.

**Note**: If abundances do not sum to 1, CAMISIM will automatically normalize them, preserving relative proportions. This behavior mirrors the default *de novo* mode.

Ensure that `num_real_genomes` in the configuration file is set appropriately - either equal to or smaller than the number of listed genomes.

## Configuration File: New Parameters

Three additional parameters are introduced:

```ini
path_to_abundance_file = /absolute/path/to/abundance.tsv
equally_distributed_strains = True
input_genomes_to_zero = True
```

* `path_to_abundance_file`: absolute path to `abundance.tsv`.
* `equally_distributed_strains`: if `True`, each input genome contributes an equal number of strains.
* `input_genomes_to_zero`: if `True`, input genomes will have abundance set to zero, and their abundance fully redistributed to the simulated strains.

Only `path_to_abundance_file` is required **exclusively** for this modality. The other two parameters also apply to other modes, if present.

An example configuration is provided [here](https://github.com/Physics4MedicineLab/CAMISIM-BrokenStick#example-of-configuration-file).

## Redistribution of Abundances

The redistribution algorithm uses a **broken stick model** to assign relative abundances of each input genome to its simulated strains.

### Example

Given an input abundance file:

```
E.coli       0.5  
S.aureus     0.3  
S.pneumoniae 0.2  
```

With 3 original genomes and `genomes_total = 9`, 6 strains are simulated. If:

* `equally_distributed_strains = True`
* `input_genomes_to_zero = True`

Then output might look like:

```
E.coli                       0.0  
S.aureus                     0.0  
S.pneumoniae                 0.0  
simulated_E.coli.Taxon001    0.0012  
simulated_E.coli.Taxon012    0.4988  
simulated_S.aureus.Taxon007  0.1473  
simulated_S.aureus.Taxon032  0.1527  
simulated_S.pneumoniae.Taxon024  0.0400  
simulated_S.pneumoniae.Taxon017  0.1600  
```

The sum of strain abundances for each genome matches its original value. The distribution is generated via a Beta distribution and reflects biologically realistic variation.

## Example of Configuration File

```ini
[Main]
seed = 42
phase = 
max_processor = 8
dataset_id = RL
output_directory = path_to_population/out
temp_directory = /tmp
gsa = False
pooled_gsa = False
anonymous = True
compress = 1

[ReadSimulator]
readsim = CAMISIM-BrokenStick/tools/art_illumina-2.3.6/art_illumina
error_profiles = CAMISIM-BrokenStick/tools/art_illumina-2.3.6/profiles
samtools = CAMISIM-BrokenStick/tools/samtools-1.3/samtools
profile = mbarc
size = 0.1
type = art
fragments_size_mean = 270
fragment_size_standard_deviation = 27

[CommunityDesign]
ncbi_taxdump = CAMISIM-BrokenStick/tools/ncbi-taxonomy_20170222.tar.gz
strain_simulation_template = CAMISIM-BrokenStick/scripts/StrainSimulationWrapper/sgEvolver/simulation_dir
number_of_samples = 3

[community0]
metadata = path_to_population/.../metadata.tsv
id_to_genome_file = path_to_population/.../genome_to_id.tsv
id_to_gff_file = 
path_to_abundance_file = path_to_population/.../abundance.tsv
genomes_total = 15
num_real_genomes = 3
max_strains_per_otu = 1
ratio = 1
equally_distributed_strains = True
input_genomes_to_zero = True
mode = known_distribution
log_mu = 1
log_sigma = 2
gauss_mu = 1
gauss_sigma = 1
view = False
```

## References

* Fritz, A., Hofmann, P., et al. (2019). **CAMISIM: Simulating metagenomes and microbial communities**. *Microbiome*, 7:17.
  [https://doi.org/10.1186/s40168-019-0633-6](https://doi.org/10.1186/s40168-019-0633-6)
  GitHub: [CAMI-challenge/CAMISIM](https://github.com/CAMI-challenge/CAMISIM)

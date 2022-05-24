![header](imgs/header.jpg)

# AlphaFold

This package provides an implementation of AlphaFold-Multimer on the lilac cluster. For simplicity, we refer to this model as AF2 throughout the rest of this document. 

![CASP14 predictions](imgs/casp14_predictions.gif)


## Genetic databases for AF2

AlphaFold needs multiple genetic (sequence) databases to run:
*   [BFD](https://bfd.mmseqs.com/),
*   [MGnify](https://www.ebi.ac.uk/metagenomics/),
*   [PDB70](http://wwwuser.gwdg.de/~compbiol/data/hhsuite/databases/hhsuite_dbs/),
*   [PDB](https://www.rcsb.org/) (structures in the mmCIF format),
*   [PDB seqres](https://www.rcsb.org/) – only for AlphaFold-Multimer,
*   [Uniclust30](https://uniclust.mmseqs.com/),
*   [UniProt](https://www.uniprot.org/uniprot/) – only for AlphaFold-Multimer,
*   [UniRef90](https://www.uniprot.org/help/uniref).

It is stored as a shared resource here:

```
    /lila/data/alphafold-db/
```

You should see the following directory structure:

```
$DOWNLOAD_DIR/                             # Total: ~ 2.2 TB (download: 438 GB)
    bfd/                                   # ~ 1.7 TB (download: 271.6 GB)
        # 6 files.
    mgnify/                                # ~ 64 GB (download: 32.9 GB)
        mgy_clusters_2018_12.fa
    params/                                # ~ 3.5 GB (download: 3.5 GB)
        # 5 CASP14 models,
        # 5 pTM models,
        # 5 AlphaFold-Multimer models,
        # LICENSE,
        # = 16 files.
    pdb70/                                 # ~ 56 GB (download: 19.5 GB)
        # 9 files.
    pdb_mmcif/                             # ~ 206 GB (download: 46 GB)
        mmcif_files/
            # About 180,000 .cif files.
        obsolete.dat
    pdb_seqres/                            # ~ 0.2 GB (download: 0.2 GB)
        pdb_seqres.txt
    small_bfd/                             # ~ 17 GB (download: 9.6 GB)
        bfd-first_non_consensus_sequences.fasta
    uniclust30/                            # ~ 86 GB (download: 24.9 GB)
        uniclust30_2018_08/
            # 13 files.
    uniprot/                               # ~ 98.3 GB (download: 49 GB)
        uniprot.fasta
    uniref90/                              # ~ 58 GB (download: 29.7 GB)
        uniref90.fasta
```


## Setup and installation
### Method :one: AF2 module

AF2 has a number of dependencies that need to be loaded first before you can begin running the software. **The simplest way to run AF2 is using the AF2 installation that is available in the cluster software stack.**

#### Load modules

```
    module load alphafold/2.2.0/af_2.2.0
```
#### Launch interactive job

Please launch an interactive job if not using a job submission script. This can be helpful for test runs or debugging purposes.
Here is an example command to create an interactive bash shell on a compute node:

```
    bsub -q gpuqueue -n 4 -R rusage[mem=10]span[ptile=4] -gpu "num=1:mode=shared"  -W 6:00  -Is bash
```

Once the module is loaded, you can now begin running AF2 following the instructions [here](https://github.mskcc.org/HPC/alphafold#running-alphafold-multimer-v211).
> Additionally, custom aliases were created to simplify the run command depending on the AF2 module being used.

1. ```runaf``` used for alphafold/2.0.1/af_2.0.1
2. ```runaf2``` used for alphafold/2.1.1/af_2.1.1
3. ```runaf-2.2.0``` used for alphafold/2.2.0/af_2.2.0


### Method :two: AF2 in conda environment

Alternatively, you can create a virtual environment to run AF2 which is completely independent from using the AF2 module above.

#### Download this repository

```
    git clone https://github.mskcc.org/hpc/alphafold.git
```

#### Move into AF2 directory

```
    cd alphafold
```

#### Install Miniconda

```
    wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh && bash Miniconda3-latest-Linux-x86_64.sh
```


#### Install AF2 on a conda virtual environment
```
    conda env create -f environments.yml python=3.8 -n alphafold
```

The conda ```environments.yml``` file and ```requirements.txt``` installs all necessary packages for AF2.
You can name the conda environment using the ```-n``` flag, in this case it is named as alphafold.

#### Activate AF2 conda environment
```
    conda activate alphafold
```

#### Load CUDA Toolkit
```
    module load cuda/11.3
```

#### Launch interactive job

Please launch an interactive job if not using a job submission script. This can be helpful for test runs or debugging purposes.
Here is an example command to create an interactive bash shell on a compute node:

```
    bsub -q gpuqueue -n 4 -R rusage[mem=10]span[ptile=4] -gpu "num=1:mode=shared"  -W 6:00  -Is bash
```

> Please use the full run command ```bash run_alphafold.sh``` to replace the custom aliases when running AF2 using a conda environment.

## Running AlphaFold

The [launch script](run_alphafold.sh) below shows the parameters to run AF2:

```
Usage: run_alphafold.sh <OPTIONS>
Required Parameters:
-d <data_dir>         Path to directory of supporting data
-o <output_dir>       Path to a directory that will store the results.
-f <fasta_path>       Path to a FASTA file containing sequence. If a FASTA file contains multiple sequences, then it will be folded as a multimer
-t <max_template_date> Maximum template release date to consider (ISO-8601 format - i.e. YYYY-MM-DD). Important if folding historical test sets
Optional Parameters:
-g <use_gpu>          Enable NVIDIA runtime to run with GPUs (default: true)
-r <run_relax>        Whether to run the final relaxation step on the predicted models. Turning relax off might result in predictions with distracting stereochemical violations but might help in case you are having issues with the relaxation stage (default: true)
-e <enable_gpu_relax> Run relax on GPU if GPU is enabled (default: true)
-n <openmm_threads>   OpenMM threads (default: all available cores)
-a <gpu_devices>      Comma separated list of devices to pass to 'CUDA_VISIBLE_DEVICES' (default: 0)
-m <model_preset>     Choose preset model configuration - the monomer model, the monomer model with extra ensembling, monomer model with pTM head, or multimer model (default: 'monomer')
-c <db_preset>        Choose preset MSA database configuration - smaller genetic database config (reduced_dbs) or full genetic database config (full_dbs) (default: 'full_dbs')
-p <use_precomputed_msas> Whether to read MSAs that have been written to disk. WARNING: This will not check if the sequence, database or configuration have changed (default: 'false')
-l <num_multimer_predictions_per_model> How many predictions (each with a different random seed) will be generated per model. E.g. if this is 2 and there are 5 models then there will be 10 predictions per input. Note: this FLAG only applies if model_preset=multimer (default: 5)
-b <benchmark>        Run multiple JAX model evaluations to obtain a timing that excludes the compilation time, which should be more indicative of the time required for inferencing many proteins (default: 'false')
```

1.  Put your query sequence in a fasta file <filename.fasta>.

2.  Running the script

    ```
    # Example run (Uses GPU)
    runaf-2.2.0 -d /lila/data/alphafold-db/ -o /<path>/<to output_dir>/ -f ./example/query.fasta -t 2021-11-01
    
    # OR only uses CPU
    runaf-2.2.0 -d /lila/data/alphafold-db/ -o /<path>/<to output_dir>/ -f ./example/query.fasta -t 2021-11-01 -g false
    ```

### AlphaFold output

The outputs will be saved in a subdirectory of the directory provided via the
`-o` flag. The outputs include the computed MSAs, unrelaxed structures, relaxed structures,
ranked structures, raw model outputs, prediction metadata, and section timings.
The output directory should have the following structure:

```
.
└── <target name>
    ├── features.pkl
    ├── msas
    │   ├── bfd_uniclust_hits.a3m
    │   ├── mgnify_hits.sto
    │   ├── pdb_hits.hhr
    │   └── uniref90_hits.sto
    ├── ranked_0.pdb
    ├── ranked_1.pdb
    ├── ranked_2.pdb
    ├── ranked_3.pdb
    ├── ranked_4.pdb
    ├── ranking_debug.json
    ├── relaxed_model_1.pdb
    ├── relaxed_model_2.pdb
    ├── relaxed_model_3.pdb
    ├── relaxed_model_4.pdb
    ├── relaxed_model_5.pdb
    ├── result_model_1.pkl
    ├── result_model_2.pkl
    ├── result_model_3.pkl
    ├── result_model_4.pkl
    ├── result_model_5.pkl
    ├── timings.json
    ├── unrelaxed_model_1.pdb
    ├── unrelaxed_model_2.pdb
    ├── unrelaxed_model_3.pdb
    ├── unrelaxed_model_4.pdb
    └── unrelaxed_model_5.pdb
```

## Running AlphaFold-Multimer (v2.2.0)

All steps are the same as when running the monomer system, but you will have to

*   provide an input fasta with multiple sequences,
*   set `-m multimer` option when running run_alphafold.sh script,
*   optionally set the `-l` flag with true or false that determine
    whether all input sequences in the given fasta file are prokaryotic. If that
    is not the case or the origin is unknown, set to `false` for that fasta.

Example run:

```
runaf-2.2.0 -d /lila/data/alphafold-db/ -o /<path>/<to output_dir>/ -f multimer_query.fasta -t 2021-11-01 -m multimer -l true
```

### Examples

Below are examples on how to use AlphaFold in different scenarios.

#### Folding a monomer

Say we have a monomer with the sequence `<SEQUENCE>`. The input fasta should be:

```fasta
>sequence_name
<SEQUENCE>
```

Then run the following command:

```
runaf-2.2.0 -d /lila/data/alphafold-db/ -o /<path>/<to output_dir>/ -f monomer.fasta -t 2021-11-01 -m monomer
```

#### Folding a homomer

Say we have a homomer from a prokaryote with 3 copies of the same sequence
`<SEQUENCE>`. The input fasta should be:

```fasta
>sequence_1
<SEQUENCE>
>sequence_2
<SEQUENCE>
>sequence_3
<SEQUENCE>
```

Then run the following command:

```
runaf-2.2.0 -d /lila/data/alphafold-db/ -o /<path>/<to output_dir>/ -f homomer.fasta -t 2021-11-01 -m multimer -l true
```

#### Folding a heteromer

Say we have a heteromer A2B3 of unknown origin, i.e. with 2 copies of
`<SEQUENCE A>` and 3 copies of `<SEQUENCE B>`. The input fasta should be:

```fasta
>sequence_1
<SEQUENCE A>
>sequence_2
<SEQUENCE A>
>sequence_3
<SEQUENCE B>
>sequence_4
<SEQUENCE B>
>sequence_5
<SEQUENCE B>
```

Then run the following command:

```
runaf-2.2.0 -d /lila/data/alphafold-db/ -o /<path>/<to output_dir>/ -f heteromer.fasta -t 2021-11-01 -m multimer -l false
```

### AlphaFold-Multimer output

The output directory for a multimer run should have the following structure:

```
.
└── <target name>
    ├── features.pkl
    ├── msas
    │   ├── A
    │   │   ├── bfd_uniclust_hits.a3m
    │   │   ├── mgnify_hits.sto
    │   │   ├── pdb_hits.sto
    │   │   ├── uniprot_hits.sto
    │   │   └── uniref90_hits.sto
    │   ├── B
    │   │   ├── bfd_uniclust_hits.a3m
    │   │   ├── mgnify_hits.sto
    │   │   ├── pdb_hits.sto
    │   │   ├── uniprot_hits.sto
    │   │   └── uniref90_hits.sto
    │   ├── C
    │   │   ├── bfd_uniclust_hits.a3m
    │   │   ├── mgnify_hits.sto
    │   │   ├── pdb_hits.sto
    │   │   ├── uniprot_hits.sto
    │   │   └── uniref90_hits.sto
    │   ├── chain_id_map.json
    │   ├── D
    │   │   ├── bfd_uniclust_hits.a3m
    │   │   ├── mgnify_hits.sto
    │   │   ├── pdb_hits.sto
    │   │   ├── uniprot_hits.sto
    │   │   └── uniref90_hits.sto
    │   └── E
    │       ├── bfd_uniclust_hits.a3m
    │       ├── mgnify_hits.sto
    │       ├── pdb_hits.sto
    │       ├── uniprot_hits.sto
    │       └── uniref90_hits.sto
    ├── ranked_0.pdb
    ├── ranked_1.pdb
    ├── ranked_2.pdb
    ├── ranked_3.pdb
    ├── ranked_4.pdb
    ├── ranking_debug.json
    ├── relaxed_model_1_multimer_v2_pred_0.pdb
    ├── relaxed_model_2_multimer_v2_pred_0.pdb
    ├── relaxed_model_3_multimer_v2_pred_0.pdb
    ├── relaxed_model_4_multimer_v2_pred_0.pdb
    ├── relaxed_model_5_multimer_v2_pred_0.pdb
    ├── result_model_1_multimer_v2_pred_0.pkl
    ├── result_model_2_multimer_v2_pred_0.pkl
    ├── result_model_3_multimer_v2_pred_0.pkl
    ├── result_model_4_multimer_v2_pred_0.pkl
    ├── result_model_5_multimer_v2_pred_0.pkl
    ├── timings.json
    ├── unrelaxed_model_1_multimer_v2_pred_0.pdb
    ├── unrelaxed_model_2_multimer_v2_pred_0.pdb
    ├── unrelaxed_model_3_multimer_v2_pred_0.pdb
    ├── unrelaxed_model_4_multimer_v2_pred_0.pdb
    └── unrelaxed_model_5_multimer_v2_pred_0.pdb
```
Here is an example of ranked_0.pdb viewed on a protein viewer (Mol*) extension on VSCode:
![Screen Shot 2022-02-01 at 4 59 38 PM](https://github.mskcc.org/storage/user/1019/files/be353a1a-0d52-4f3e-a5bb-6c24d9960ac7)


## API Changes
### API changes between v2.1.1 and v2.2.0
*   The is_prokaryote option -l is removed
*   New option -l is now used for setting the number of multimer predictions per model
*   Options for relaxation -r and to enable GPU relaxation -e are added
*   AF2 parameters link has been updated in the download_db.sh script (users should download this new parameters when using AF2 v2.2.0)

### API changes between v2.0.0 and v2.1.1
*   The preset flag -p was split into -c (db_preset) and -m (model_preset) in our run_alphafold.sh
    * Four model presets (for option -m) are now supported
        * monomer
        * monomer_casp14
        * monomer_ptm
        * multimer
    * Two db preset configurations (for option -c) are supported
        * full_dbs
        * reduced_dbs
*   The model names to use are not specified using -m option anymore. If you want to customize model names you will have to modify the appropriate MODEL_PRESETS dictionary in alphafold/model/config.py


## Submitting a batch job

Below are some templates for your LSF script. You can customize the script as necessary and submit to the LSF queue by running ```bsub < sample_bsub.sh```

#### Monomer with ```full_dbs``` (using conda environment)
```
    #!/bin/bash
    #BSUB -J alphafold
    #BSUB -q gpuqueue
    #BSUB -n 12
    #BSUB -R span[ptile=12]
    #BSUB -R rusage[mem=8]
    #BSUB -gpu "num=1:mode=shared:j_exclusive=yes"
    #BSUB -W 5:00
    #BSUB -o /lila/data/hpcadmin/home/cheongj1/outputs/errors.%J
    #BSUB -eo /lila/data/hpcadmin/home/cheongj1/outputs/output.%J
    #BSUB -L /bin/bash
    
    cd /lila/data/hpcadmin/home/cheongj1/alphafold/
    
    module load cuda/11.3
    source /home/cheongj1/miniconda3/bin/activate
    conda activate alphafold
    
    bash run_alphafold.sh -d /lila/data/alphafold-db/ -o /<path>/<to output_dir>/ -f /<path>/<to fasta file>/ -t 2021-11-01 >alpha.out 2>alpha.err 
```

#### Multimer with ```reduced_dbs``` (using AF2 module)
```
    #!/bin/bash
    #BSUB -J alphafold-run
    #BSUB -q gpuqueue
    #BSUB -n 12
    #BSUB -R span[ptile=12]
    #BSUB -R rusage[mem=8]
    #BSUB -gpu "num=1:mode=shared:j_exclusive=yes"
    #BSUB -W 5:00
    #BSUB -o /lila/data/hpcadmin/home/%U/outputs/errors.%J
    #BSUB -eo /lila/data/hpcadmin/home/%U/outputs/output.%J
    #BSUB -L /bin/bash
    
    # Unload all loaded modules and reset everything to original state
    module purge
    
    # Load alphafold module
    module load alphafold/2.2.0/af_2.2.0
    
    # Run alphafold
    runaf-2.2.0 -d /lila/data/alphafold-db/ -o /<path>/<to output_dir>/ -f /<path>/<to fasta file>/ -t 2021-11-01 -m multimer -l false -c reduced_dbs >alpha.out 2>alpha.err
```

#### Multimer with ```full_dbs``` across 4 GPUs (using AF2 module)
```
    #!/bin/bash
    #BSUB -J alphafold-run
    #BSUB -q gpuqueue
    #BSUB -n 20
    #BSUB -R span[ptile=20]
    #BSUB -R rusage[mem=40]
    #BSUB -gpu "num=4:mode=shared:j_exclusive=yes"
    #BSUB -W 40:00
    #BSUB -o /lila/data/hpcadmin/home/%U/outputs/errors.%J
    #BSUB -eo /lila/data/hpcadmin/home/%U/outputs/output.%J
    #BSUB -L /bin/bash

    # Unload all loaded modules
    module purge

    # Load alphafold module and required environments
    module load alphafold/2.2.0/af_2.2.0
    module load cudnn/8.1.0-cuda11.2

    export OPENMM_CPU_THREADS=8
    export NVIDIA_VISIBLE_DEVICES=all
    export TF_FORCE_UNIFIED_MEMORY=1
    export TF_FORCE_GPU_ALLOW_GROWTH=true
    export XLA_PYTHON_CLIENT_MEM_FRACTION=8.0
    export XLA_PYTHON_CLIENT_ALLOCATOR=platform
    export XLA_PYTHON_CLIENT_PREALLOCATE=false
    export XLA_FLAGS=--xla_gpu_force_compilation_parallelism=1

    # Run alphafold
    runaf-2.2.0 \
    -d /lila/data/alphafold-db/ \
    -o /<path>/<to output_dir>/ \
    -f /<path>/<to fasta file>/ \
    -t 2022-04-15 \
    -a 0,1,2,3 \
    -r true \
    -g true \
    -e true \
    -m multimer \
    -c full_dbs \
    >alpha-latest.out \
    2>alpha-latest.err
```

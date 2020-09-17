[![Build Status](https://travis-ci.com/PaccMann/paccmann_sarscov2.svg?branch=master)](https://travis-ci.com/PaccMann/paccmann_sarscov2)
# paccmann_rl

Pipeline to reproduce the results of the [PaccMann^RL on SARS-CoV-2 paper](https://arxiv.org/abs/2005.13285).

## Description

In the repo we provide a conda environment and instructions to reproduce the pipeline described in the manuscript:

1. Train a multimodal protein-compound interaction classifier, also known as the affinity predictor ([source code](https://github.com/PaccMann/paccmann_predictor))
2. Train a toxicity predictor ([source code](https://github.com/PaccMann/toxsmi))
3. Train a generative model for omic profiles, also known as the ProteinVAE ([source code](https://github.com/PaccMann/paccmann_omics))
4. Train a generative model for molecules, also known as the SELFIESVAE ([source code](https://github.com/PaccMann/paccmann_chemistry))
5. Train PaccMann^RL on SARS-CoV-2 using the pretained models from above ([source code](https://github.com/PaccMann/paccmann_generator))


**NOTE:** In the linked repositories, there are often multiple examples for training. For the use case of `paccmann_sarscov2`, relevant examples are named `affinity` or `encoded_proteins`.

## Requirements

- `conda>=3.7`
- The following data from [here](https://ibm.ent.box.com/v/paccmann-sarscov2-data)  
  View the respective README.md files on data sources.  
- The git repos linked in the [previous section](#description)

<!-- **NOTE:** please refer to the [README.md](https://ibm.ent.box.com/v/paccmann-pytoda-data/file/548614344106) and to the manuscript for details on the datasets used and the preprocessing applied. -->

## Setup

### Install the environment

Create a conda environment:

```sh
conda env create -f conda.yml
```

Activate the environment:

```sh
conda activate paccmann_sarscov2
```

### Download data and optional pretrained models

Download the data reported in the [requirements section](#requirements).
From now on, we will assume that they are stored in the root of the repository in a folder called `data`, following this structure:

```console
data
├── pretraining
│   ├── ProteinVAE
│   ├── SELFIESVAE
│   ├── affinity_predictor
│   ├── laguage_models
│   └── toxicity_predictor
└── training
```
This is around **6GB** of data, required for pretaining multiple models.
Also, the workload required to run the full pipeline is intensive and might not be straightforward to run all the steps on a desktop laptop.

For these reasons we also provide [pretrained models](https://ibm.ent.box.com/v/paccmann-sarscov2-models) (ca. 700MB) for download. In the [Pipeline](#pipeline) section this allows you to skip directly to [PaccMann^RL on SARS-CoV-2](#paccmannrl-on-sars-cov-2).

If you chose to download our pretrained models, the directory structure looks like this:

```console
models
├── ProteinVAE
├── SELFIESVAE
├── Tox21
└── affinity
```

In this case you would only require the data under `data/training/` (8MB).

**NOTE:** no worries, the `data` folder is in the [.gitignore](./.gitignore).

### Clone the repos

To get the scripts to run each of the component create a `code` folder and clone the repos. Simply type this:

```sh
mkdir code && cd code && \
  git clone --branch sarscov2 https://github.com/PaccMann/paccmann_predictor && \ 
  git clone --branch 0.0.1 https://github.com/PaccMann/toxsmi && \
  git clone --branch sarscov2 https://github.com/PaccMann/paccmann_omics && \ 
  git clone --branch sarscov2 https://github.com/PaccMann/paccmann_chemistry && \ 
  git clone --branch sarscov2 https://github.com/PaccMann/paccmann_generator && \
  cd ..
```
The branch is given to ensure a version working with the provided conda environment.

**NOTE:** no worries, the `code` folder is in the [.gitignore](./.gitignore).

## Pipeline

Now it's all set to run the full pipeline.

Calling any of the scripts with the `-h` or `--help` flag will provide you with some information on the arguments.

**NOTE:** in the following, we assume a folder `models` has been created in the root of the repository. No worries, the `models` folder is in the [.gitignore](./.gitignore).

### affinity predictor
```console
(paccmann_sarscov2) $ python ./code/paccmann_predictor/examples/affinity/train_affinity.py \
    ./data/pretraining/affinity_predictor/filtered_train_binding_data.csv \
    ./data/pretraining/affinity_predictor/filtered_val_binding_data.csv \
    ./data/pretraining/affinity_predictor/sequences.smi \
    ./data/pretraining/affinity_predictor/filtered_ligands.smi \
    ./data/pretraining/language_models/smiles_language_chembl_gdsc_ccle_tox21_zinc_organdb_bindingdb.pkl \
    ./data/pretraining/language_models/protein_language_bindingdb.pkl \
    ./models/ \
    ./code/paccmann_predictor/examples/affinity/affinity.json \
    affinity
```

### toxicity predictor
TODO embedding path in toxsmi.json
```console
(paccmann_sarscov2) $ python ./code/toxsmi/scripts/train_tox.py \
    ./data/pretraining/toxicity_predictor/tox21_train.csv \
    ./data/pretraining/toxicity_predictor/tox21_test.csv \
    ./data/pretraining/toxicity_predictor/tox21.smi \
    ./data/pretraining/language_models/smiles_language_tox21.pkl \
    ./models/ \
    ./code/toxsmi/params/mca.json \
    Tox21
    --embedding_path ./data/pretraining/toxicity_predictor/smiles_vae_embeddings.pkl
```

### protein VAE
``` console
(paccmann_sarscov2) $ python ./code/paccmann_omics/examples/encoded_proteins/train_protein_encoding_vae.py \
    ./data/pretraining/proteinVAE/tape_encoded/avg/train_representation.csv \
    ./data/pretraining/proteinVAE/tape_encoded/avg/val_representation.csv \
    ./models/ \
    ./code/paccmann_omics/examples/encoded_proteins/protein_encoding_vae_params.json \
    ProteinVAE
```

### SELFIES VAE
``` console
(paccmann_sarscov2) $ python ./code/paccmann_chemistry/examples/train_vae.py \
    ./data/pretraining/SELFIESVAE/train_chembl_22_clean_1576904_sorted_std_final.smi \
    ./data/pretraining/SELFIESVAE/test_chembl_22_clean_1576904_sorted_std_final.smi \
    ./data/pretraining/language_models/selfies_language.pkl \
    ./models/ \
    ./code/paccmann_chemistry/examples/example_params.json \
    SELFIESVAE
```

### PaccMann^RL on SARS-CoV-2
``` console
(paccmann_sarscov2) $ python ./code/paccmann_generator/examples/affinity/train_conditional_generator.py \
    ./models/SELFIESVAE \
    ./models/ProteinVAE \
    ./models/affinity \
    ./data/training/merged_sequence_encoding/uniprot_covid-19.csv \
    ./code/paccmann_generator/examples/affinity/conditional_generator.json \
    paccmann_sarscov2 \
    35 \
    data/training/unbiased_predictons \
    --tox21_path ./models/Tox21
```

This will create a `biased_models` folder containing the conditional generators, biased for all provided proteins from [covid-19.uniprot.org](https://covid-19.uniprot.org/) except one, in the example for ACE2_HUMAN. The biased generator generates compounds with a shifted distribution compared to unbiased predictions. Ideally, the model generalizes to ACE2_HUMAN and the biased compounds have overall higher affinity (to ACE2_HUMAN) **according to the affinity predictor**. See the pdf files in `biased_models/paccmann_sarscov2_35/results` to observe the effect at different stages of training.  
**NOTE:** no worries, the `biased_models` folder is in the [.gitignore](./.gitignore).

## References

If you use `paccmann_sarscov2` in your projects, please cite the following:

```bib
@misc{born2020paccmannrl,
    title={PaccMann$^{RL}$ on SARS-CoV-2: Designing antiviral candidates with conditional generative models},
    author={Jannis Born and Matteo Manica and Joris Cadow and Greta Markert and Nil Adell Mill and Modestas Filipavicius and María Rodríguez Martínez},
    year={2020},
    eprint={2005.13285},
    archivePrefix={arXiv},
    primaryClass={q-bio.QM}
}
```

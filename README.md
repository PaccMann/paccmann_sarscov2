[![Build Status](https://travis-ci.com/PaccMann/paccmann_sarscov2.svg?branch=master)](https://travis-ci.com/PaccMann/paccmann_sarscov2)
# paccmann_rl

Pipeline to reproduce the results of the [PaccMann^RL on SARS-CoV-2 paper](https://arxiv.org/abs/2005.13285).

## Description

In the repo we provide a conda environment and instructions to reproduce the pipeline described in the manuscript:

1. Train a multimodal protein-compound interaction classifier, also known as the affinity predictor ([source code](https://github.com/PaccMann/paccmann_predictor))
2. Train a toxicity predictor ([source code](https://github.com/PaccMann/toxsmi))
3. Train a generative model for omic profiles, also known as the ProteinVAE ([source code](https://github.com/PaccMann/paccmann_omics))
4. Train a generative model for molecules, also known as the SVAE ([source code](https://github.com/PaccMann/paccmann_chemistry))
5. Train PaccMann^RL on SARS-CoV-2 using the pretained models from above ([source code](https://github.com/PaccMann/paccmann_generator))


**NOTE:** In the linked repositories, there are often multiple examples for training. For the use case of `paccmann_sarscov2`, relevant examples are named `affinity` or `encoded_proteins`.

## Requirements

- `conda>=3.7`
- TODO The following data from [here](https://ibm.ent.box.com/v/paccmann-sarscov2-data)  
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

### Download data

Download the data reported in the [requirements section](#requirements).
From now on, we will assume that they are stored in the root of the repository in a folder called `data`, following this structure:

TODO update final version
```console
data
├── pretraining
│   ├── SELFIESVAE
│   │   ├── test_chembl_22_clean_1576904_sorted_std_final.smi
│   │   └── train_chembl_22_clean_1576904_sorted_std_final.smi
│   ├── affinity_predictor
│   │   ├── README.md
│   │   ├── filtered_ligands.smi
│   │   ├── filtered_test_binding_data.csv
│   │   ├── filtered_train_binding_data.csv
│   │   ├── filtered_val_binding_data.csv
│   │   └── sequences.smi
│   ├── ProteinVAE
│   │   ├── README.md
│   │   ├── all_sequence_data.fasta
│   │   └── tape_encoded
│   │       └── avg
│   │           ├── test_representation.csv
│   │           ├── train_representation.csv
│   │           └── val_representation.csv
│   └── toxicity_predictor
│       ├── README.md
│       ├── smiles_language_tox21.pkl
│       ├── smiles_vae_embeddings.pkl
│       ├── tox21.smi
│       ├── tox21_score.csv
│       ├── tox21_test.csv
│       └── tox21_train.csv
└── training
    ├── README.md
    ├── tape_encoded
    │   └── avg.csv
    └── uniprot_covid-19.fasta
```
**NOTE:** in the [Pipelines](#pipeline) section you have the option to use our pretrained models. In this case you would only require the data under `data/training/`.

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
The branch is given to ensure a tested version.

**NOTE:** no worries, the `code` folder is in the [.gitignore](./.gitignore).

## Pipeline

Now it's all set to run the full pipeline.

**NOTE:** the workload required to run the full pipeline is intesive and might not be straightforward to run all the steps on a desktop laptop. For this reason, we also provide [pretrained models](https://ibm.ent.box.com/v/paccmann-sarscov2-models) that can be downloaded and used to run the different steps. 

if you chose to download our pretrained models, the directory structure looks like this:
TODO update final
```console
models
├── OrganDB
│   ├── model_params.json
│   ├── results
│   │   ├── best_predictions.npy
│   │   └── metrics.json
│   ├── smiles_language.pkl
│   └── weights
│       ├── best_ROC-AUC_mca.pt
│       ├── best_loss_mca.pt
│       ├── best_precision-recall\ score_mca.pt
│       └── done_training_mca.pt
├── ProteinVAE
│   ├── model_params.json
│   └── weights
│       ├── 1249_epoch_decoder.pt
│       ├── 1249_epoch_encoder.pt
│       ├── 1249_epoch_vae.pt
│       ├── best_both_decoder.pt
│       ├── best_both_encoder.pt
│       ├── best_both_vae.pt
│       ├── best_kl_decoder.pt
│       ├── best_kl_encoder.pt
│       ├── best_kl_vae.pt
│       ├── best_rec_decoder.pt
│       ├── best_rec_encoder.pt
│       └── best_rec_vae.pt
├── SELFIESVAE
│   ├── loss_tracker.json
│   ├── model_params.json
│   ├── selfies_language.pkl
│   └── weights
│       ├── best_kld.pt
│       ├── best_loss.pt
│       ├── best_rec.pt
│       └── saved_model_epoch_4_iter_3000.pt
├── Tox21
│   ├── model_params.json
│   ├── results
│   │   ├── best_predictions.npy
│   │   └── metrics.json
│   ├── smiles_language.pkl
│   └── weights
│       ├── best_ROC-AUC_mca.pt
│       ├── best_loss_mca.pt
│       └── best_precision-recall\ score_mca.pt
├── affinity_prediction
├── base_affinity
│   ├── model_params.json
│   ├── protein_language.pkl
│   ├── smiles_language.pkl
│   └── weights
│       ├── best_ROC-AUC_bimodal_mca.pt
│       └── best_loss_bimodal_mca.pt
└── language_models
    ├── protein_language_bindingdb.pkl
    └── smiles_language_chembl_gdsc_ccle_tox21_zinc_organdb_bindingdb.pkl
```

**NOTE:** in the following, we assume a folder `models` has been created in the root of the repository. No worries, the `models` folder is in the [.gitignore](./.gitignore).

### affinity predictor
TODO
```console
(paccmann_sarscov2) $ python ./code/paccmann_predictor/examples/affinity/train_affinity.py \
    ./data/splitted_data/gdsc_cell_line_ic50_train_fraction_0.9_id_997_seed_42.csv \
    ./data/splitted_data/gdsc_cell_line_ic50_test_fraction_0.1_id_997_seed_42.csv \
    ./data/gdsc-rnaseq_gene-expression.csv \
    ./data/gdsc.smi \
    ./data/2128_genes.pkl \
    ./data/smiles_language_chembl_gdsc_ccle.pkl \
    ./models/ \
    ./code/paccmann_predictor/examples/example_params.json paccmann
```

### toxicity predictor
TODO
```console
(paccmann_sarscov2) $ python ./code/toxsmi/scripts/train_tox.py \
    ./data/
```

### protein VAE
TODO
``` console
(paccmann_sarscov2) $ python ./code/paccmann_omics/examples/encoded_proteins/train_protein_encoding_vae.py \
    ./data/splitted_data/tcga_rnaseq_train_fraction_0.9_id_242870585127480531622270373503581547167_seed_42.csv \
    ./data/splitted_data/tcga_rnaseq_test_fraction_0.1_id_242870585127480531622270373503581547167_seed_42.csv \
    ./data/2128_genes.pkl \
    ./models/ \
    ./code/paccmann_omics/examples/example_params.json pvae
```

### SELFIES VAE
TODO
``` console
(paccmann_sarscov2) $ python ./code/paccmann_chemistry/examples/train_vae.py \
    ./data/splitted_data/train_chembl_22_clean_1576904_sorted_std_final.smi \
    ./data/splitted_data/test_chembl_22_clean_1576904_sorted_std_final.smi \
    ./data/smiles_language_chembl_gdsc_ccle.pkl \
    ./models/ \
    ./code/paccmann_chemistry/examples/example_params.json svae
```

### PaccMann^RL on SARS-CoV-2
TODO
TODO all generator related on travis and Dockerfile, .txt and .yml
``` console
(paccmann_sarscov2) $ python ./code/paccmann_generator/examples/affinity/train_conditional_generator.py \
    ./models/svae \
    ./models/pvae \
    ./models/paccmann \
    ./data/smiles_language_chembl_gdsc_ccle.pkl \
    ./data/gdsc_transcriptomics_for_conditional_generation.pkl \
    ./code/paccmann_generator/examples/example_params.json \
    paccmann_rl breast
```

**NOTE:** this will create a `biased_model` folder containing the conditional generator and the baseline SMILES generator used. In this case: `breast_paccmann_rl` and `baseline`. No worries, the `biased_models` folder is in the [.gitignore](./.gitignore).

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

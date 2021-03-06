---
title: "Risorse di calcolo"
author: "RCF"
subtitle:    "purpleJeans, blackJeans e le altre risorse di calcolo di RCF"
description: "Elenco delle risorse di calcolo gestite da RCF."
date:        2020-11-15
image:       ""
tags:        []
categories:  []
showtoc: true
---

Le seguenti risorse di calcolo sono a disposizione degli utenti RCF:

# HPC-GPU-DNN purpleJeans

| Risorsa           | Partizioni | Nodi | CPU/Nodo                                             | Core/Nodo | Memoria/Nodo   | Note                                                                       |
|-------------------|------------|------|------------------------------------------------------|-----------|----------------|-------------------------------------------------------------------------------------------------------------------------|
| purpleJeans (2019)| xhicpu     |    4 | 2 CPU Intel(R) Xeon(R) Xeon 16-Core 5218 2,3Ghz 22MB |        32 |         192 GB | Cluster dedicato alla ricerca nell’ambito del machine learning e big data. Mellanox CX4 VPI SinglePort FDR IB 56Gb/s x16. |
|                   | xgpu       |    4 | 2 CPU Intel(R) Xeon(R) Xeon 16-Core 5218 2,3Ghz 22MB |        32 |         192 GB | 4 GPU  (NVLINK) NVIDIA Tesla V100 32GB SXM2. Infiniband Mellanox CX4 VPI SinglePort FDR IB 56Gb/s x16. |

![purpleJeans](http://purplejeans.uniparthenope.it/ganglia/stacked.php?m=load_one&c=Purplejeans-UniParthenope&r=hour)

# HPC-GPU blackJeans

| Risorsa           | Partizioni | Nodi | CPU/Nodo                                             | Core/Nodo | Memoria/Nodo   | Note                                                                       |
|-------------------|------------|------|------------------------------------------------------|-----------|----------------|-------------------------------------------------------------------------------------------------------------------------|
| blackJeans (2010) | gpu        |   12 | 2  CPU Intel(R) Xeon(R) CPU X5650 @ 2.67GHz          |        12 |          24 GB | Cluster dedicato alla produzione di simulazioni e previsioni meteorologiche, oceanografiche, dispersione di inquinanti in aria e acqua. 1 GPU  NVIDIA Tesla 2050. Infiniband Mellanox 40 Gb. |
|            (2014) | hicpu      |    6 | 2 CPU Intel(R) Xeon(R) CPU E5-2680 v2 @ 2.80GHz      |        24 |          48 GB | Infiniband Mellanox 40 Gb. |
|            (2019) | xhicpu     |    4 | 2 CPU Intel(R) Xeon(R) Gold 6230 CPU @ 2.10GHz       |        40 |          64 GB | Infiniband Mellanox 40 Gb. |

![purpleJeans](http://blackjeans.uniparthenope.it/ganglia/stacked.php?m=load_one&c=Blackjeans-UniParthenope&r=hour)

# Altre risorse di calcolo

| Risorsa           | Partizioni | Nodi | CPU/Nodo                                             | Core/Nodo | Memoria/Nodo   | Note                                                                       |
|-------------------|------------|------|------------------------------------------------------|-----------|----------------|-------------------------------------------------------------------------------------------------------------------------|
| redJeans (2012)   |            |    8 | 1 CPU i7                                             |         4 |          16 GB | Cluster Beowulf dedicato alla didattica. |
| saturn (2015)     | gpu        |    1 | 2 CPU Intel(R) Xeon(R) CPU E5-2609 v3 @ 1.90GHz      |        12 |          32 GB | Macchina dedicata alla didattica e alla ricerca prototipale. 2 GPU GeForce GTX Titan X 12GB. |
| blackhole (2015)  | gpu        |    1 | 2 CPU Intel(R) Xeon(R) E5-2620 v3 @ 2.40GHz          |        12 |          64 GB | Macchina dedicata alla didattica e alla ricerca prototipale. 1 GPU GeForce GTX Titan X 12GB. 1 GPU Quadro K4200. |
| storm (2006)      | gpu        |    1 | 1 CPU Intel(R) Xeon(R)                               |         4 |          12 GB | Macchina dedicata alla didattica e alla ricerca prototipale. 2 GPU NVIDIA Tesla 1060. 1 GPU Quadro |

# Open source

* **GVirtuS** GPGPU Virtualization Service, per condividere una GPU CUDA fra multiple macchine fisiche o virtuali.  https://github.com/gvirtus
* **DagOnStar** Motore di Workflow, per applicazioni operazionali. https://github.com/dagonstar

# Open data

* **CMMMA** Centro per il Monitoraggio e la Modlelistica Marina e Atmosferica, collezione di dati da previsioni meteomarine. http://data.meteo.uniparthenope.it/opendap/opendap/

# ALSgeneScanner

## Table of Contents
1. [Introduction](#introduction)
2. [Citation](#citation)
3. [Documentation](#documentation)
	* [Minimum requirements](#minimum-requirements)
	* [Obtaining](#obtaining)
	* [Usage](#usage)
	* [Docker and Singularity](#docker-and-singularity)	
 	* [Gene list](#gene-list)	
4. [Litterature review](#litterature-review)
5. [Core Contributors](#core-contributors)
6. [Contributing](#contributing)
7. [Licence](#licence)


## Introduction

ALSgeneScanner is a pipeline designed for the analysis of NGS data of ALS patients. It perfoms alignment, variant calling, structural variant callin and repeat expansion calling as well as variant annotation using Annovar. It restricts the analysis to a subset of genes (~150) which have been shown to be associated with ALS. A complete list of the included genes is available as a [Google Spreadsheet](https://goo.gl/mngrMC). It also prioritize variants according to the scientific evidence of the gene association and the effect prediction of the variant. At present this can only be done using the reference genome hg19. 

ALSgeneScanner is based on the DNAscan analysis framework. For a detailed description of all ALSgeneScanner components please read [the ALSgeneScanner preprint](https://www.biorxiv.org/content/biorxiv/early/2018/07/26/378158.full.pdf) and the [DNAscan preprint](https://www.biorxiv.org/content/biorxiv/early/2018/02/18/267195.full.pdf)


![alt text](https://github.com/KHP-Informatics/ALSgeneScanner/raw/master/Figure1_ALSgeneScaner.jpg)


## Citation

[Alfredo Iacoangeli et al. ALSgeneScanner: a pipeline for the analysis and interpretation of DNA NGS data of ALS patients. bioRxiv, 2018](https://www.biorxiv.org/content/biorxiv/early/2018/07/26/378158.full.pdf)

## Documentation

### Minimum requirements

- Ubuntu >= 14.04.
- RAM: 12Gbs
- Scratch space for usage: If you are performing the alignment stage and your input data is in fastq.gz format we recommend using at least 3 times the size of your input data. E.g. your fastq.gz files are 100Gb, thus you would need 300Gb of free space. If you don't wish to perform the alignment, a proportion of data-to-analyse:free-space of 1:1 would be enough. E.g. Input data is a 50Gb bam file, you would need only 50Gb of free space. 

### Obtaining

**Version:** 0.1

Please make sure all dependencies are installed before running ALSgeneScanner. Instructions on how to install all dependencies are available in the following chapter. However a bash script to set up all dependencies (Annovar needs a manual registration and download step) is available in scripts.

#### Local Deployment

To use ALSgeneScanner please first obtain the most recent DNAscan development tree using git:

```bash
git clone https://github.com/KHP-Informatics/DNAscan.git
```

Once you have downloaded DNAscan, you can set up all needed dependencies running the install_ALSgeneScanner.sh script available in DNAscan/scripts. Before running install_ALSgeneScanner.sh please download and uncompress Annovar by registering at the following [link](http://www.openbioinformatics.org/annovar/annovar_download_form.php). install_ALSgeneScanner.sh will install all software dependencies as well as hg19 reference genome and its hisat2 and bwa indexes (these jobs run in the background and will finish after the script ends) as well as update paths_and_configs.py. 

```bash

$ ./DNAscan/scripts/install_ALSgeneScanner.sh /path/to/DNAscan /path/to/annovar $num_cpus

source ~/.bashrc

```

Where $num_cpus is the number of cpus available on the machine, e.g. 4. This script will download and install all dependencies as well as create the necessary index files. The latter might require a little time depending on the number of CPUs used. To find out the number of CPUs available on your computer use the command nproc.

#### Obtain with Docker

The easiest way to get started with ALSgeneScanner or DNAscan is to use its Docker image:

```bash
sudo docker run  -v /path/to/your/data_folder:/container/path/where/you/want/your/data [-p 8080:8080] --storage-opt size=50G  -it compbio/dnascan /bin/bash
```
Please set the needed container size taking into account the "Minimum requirements".

The -v option adds your data folder (assuming you have some data to run DNAscan on), -p mirrors the container port 8080 to your host port 8080. This will be necessary if you want to use the iobio services.
The --storage-opt size=Ngigas option defines the maximum size of the container, setting it to N gigabytes. This number would depend on you plans. If you want to perform annotation, the databases used by DNAscan (clinvar,CADD,etc) are about 350G. We recommend N = 500 if you want to install the whole pipeline (including annotation). To this number you should add what you need for your analysis, e.g. if you are planning to download data, the size of your data to analyse etc. A way to workaround this is to use the mirrored host folder as outdir for your analysis and as annovar folder. This folder does not have a size limit. 

IMPORTANT: To detach from the container without stopping it, use Ctrl+p, Ctrl+q.
IMPORTANT: When running DNAscan inside a docker container, if you want to use the iobio services (and for example upload your results into the gene.iobio platform), these would not be visible by your browser unless they are in the folder which is mirrored on the host system. Considering the previous command to run an ubuntu image, the easiest way to do this would be to use the folder where you imported the data inside the container (/container/path/where/you/want/your/data) as an outdir when running DNAsca. In this way the DNAscan results can be found in /path/to/your/data_folder on the host system.

If you want to add data to the container while this is already running you can use the docker cp command. First detach from the container without stopping it using Ctrl+p, Ctrl+q, then cp your data inside the container:

```bash
docker cp [OPTIONS] /path/to/your/data CONTAINER_ID:/container/path/where/you/want/your/data
```
and execute a bash shell inside your container:

```bash
docker exec -it CONTAINER_ID /bin/bash
```
The container ID can be found using the ps command:

```bash
docker ps 
```

### Usage

To run the whole ALSgeneScanner pipeline on paired-end reads in fastq format stored in data1.fq.gz and data2.fq.gz, you can run the following command running the main DNAscan script in /path/to/DNAscan/scripts :


#### Usage example

##### fastq input

Let's assume we have human paired end whole exome sequening data in two fastq files and want to perform snvs/indels calling vs hg19, annotation and explore the results using the iobio services. The DNAscan command line would be:

 ```bash
./DNAscan/scripts/DNAscan.py -in data1.fq.gz -in2 data2.fq.gz -alsgenescanner -out /path/to/outdir

```

##### bam input

 ```bash
./DNAscan/scripts/DNAscan.py -in data.bam -format bam -alsgenescanner -out /path/to/outdir

```

##### vcf input

 ```bash
./DNAscan/scripts/DNAscan.py -in data.vcf.gz -format vcf -alsgenescanner -out /path/to/outdir

```


### Docker and Singularity 

**Docker** is an open-source project that automates the deployment of applications inside software containers. Using containers to deploy our system and creating our analysis environment would allow us to make our work independent of the machine we work on. This would improve the reproducibility of our science, the portability and reliability of our deployments and avoid any machine specific issues. For this reason working using containers isn't just recommended but also makes things easier. Since docker is widely used and maintained we recommend it as container technology to use if possible. Unfortunately Docker does require sudo privileges to run its containers making its use difficult on HPC facilities.

**Singularity** is also a container project similar to Docker and does not require sudo privileges to run. This can be very important if you decide to use our framework on a machine for which you do not have such privileges. E.g. your institution HPC cluster. In this case you can use Singularity to convert the docker image into aSsingularity image and run a bash shell in the resulting Singularity container:

```bash 
$ singularity shell docker://compbio/dnascan
```
After starting the bash shell inside the singularity container you can find a working deployment of DNAscan in /DNAscan

```bash 
$ Singularity.dnascan> cd /DNAscan
$ Singularity.dnascan> cat /DNAscan/docker/welcome_message.txt
```

##### Docker setup

###### ubuntu

[LINK](https://store.docker.com/editions/community/docker-ce-server-ubuntu)

###### MAC

[LINK](https://store.docker.com/editions/community/docker-ce-desktop-mac)

###### Windows

[LINK](https://store.docker.com/editions/community/docker-ce-desktop-windows)

##### Singularity setup

###### Linux

[LINK](http://singularity.lbl.gov/install-linux)

###### MAC

[LINK](http://singularity.lbl.gov/install-mac)

### Gene list

ALSgeneScanner restricts the analysis to a subset of genes (~150) which have been shown to be associated with ALS. A complete list of the included genes is available as a [Google Spreadsheet](https://goo.gl/mngrMC). This list can be customized. If you want to do so, please follow the instructions in the [DNAscan github](https://github.com/KHP-Informatics/DNAscan)

## Litterature review

Manual literature review identified 486 articles describing a total of 127 genes and loci associated with ALS. 
A list of the 486 article titles can be found at this [link](https://docs.google.com/spreadsheets/d/1dNoquP8aSAr85X9xqPGjKnvaxpQadJ4iyXuk5hsYx3Y/edit?usp=sharing).

A list of the 127 genes and loci associated with ALS can be found at this [link](https://docs.google.com/spreadsheets/d/1uwuIrWwrl4UzVYMOJPyDQ7wTcn3LvfjJWPPRGvKfi_U/edit?usp=sharing).




## Core Contributors
- [Dr Alfredo Iacoangeli](alfredo.iacoangeli@kcl.ac.uk), UK
- [Dr Stephen J Newhouse](stephen.j.newhouse@gmail.com), UK

### Contributors
- [NAME](email), [COUNTRY OR AFFILIATION]

For a full list of contributors see [LINK](./CONTRIBUTORS.md)

## Contributing

Here’s how we suggest you go about proposing a change to this project:

1. [Fork this project][fork] to your account.
2. [Create a branch][branch] for the change you intend to make.
3. Make your changes to your fork.
4. [Send a pull request][pr] from your fork’s branch to our `master` branch.

Using the web-based interface to make changes is fine too, and will help you
by automatically forking the project and prompting to send a pull request too.

[fork]: https://help.github.com/articles/fork-a-repo/
[branch]: https://help.github.com/articles/creating-and-deleting-branches-within-your-repository
[pr]: https://help.github.com/articles/using-pull-requests/


## Licence 
- [MIT](./LICENSE.txt)


*********



<p align="center">
  Core Developers funded as part of:</br> 
  <a href="https://www.mndassociation.org/">MNDA</a></br> 
  <a href="http://www.maudsleybrc.nihr.ac.uk/">NIHR Maudsley Biomedical Research Centre (BRC), King's College London</a></br>
  <a href="http://www.ucl.ac.uk/health-informatics/">Farr Institute of Health Informatics Research, UCL Institute of Health Informatics, University College London</a>
</p>


![alt text](https://raw.githubusercontent.com/KHP-Informatics/MND-DataManagementAnalysis-System/master/funder_logos.001.jpeg)

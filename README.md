OpenEBench (OEB) Benchmarking Workflows

This is where the source code of the benchmarking workflows are located. The dedicated code for each ***benchmarking event*** resides in its own subdirectory e.g. EuCanImage Brain Cancer Diagnosis Benchmarking Event (Brain_Cancer_Diagnosis) example for Segmentation (ECI_UC0_Seg) challenge.

- [Overview](#overview)
- [Benchmarking workflow general description](#benchmarking-workflow-general-description)
  - [Directory structure for benchmarking workflow code](#directory-structure-for-benchmarking-workflow-code)
- [HOW TO: (File) naming requirements](#how-to-file-naming-requirements)
  - [Challenges](#challenges)
  - [Ground truth files](#ground-truth-files)
  - [Participant output (=input) files](#participant-output-input-files)
- [HOW TO: DEVELOP](#how-to-develop)
  - [1. Copy template](#1-copy-template)
  - [2. Establish proper validation](#2-establish-proper-validation)
  - [3. Calculate the metrics](#3-calculate-the-metrics)
  - [4. Consolidation](#4-consolidation)
  - [5. Adapt `nextflow.config` and `main.nf`](#5-adapt-nextflowconfig-and-mainnf)
  - [6. Don't forget to update `README.md`](#6-dont-forget-to-update-readmemd)
  - [7. Build images](#7-build-images)
  - [8. Test run](#8-test-run)
- [HOW TO: "PRODUCTION"](#how-to-production)
  - [1. Get input data (= participant output)](#1-get-input-data--participant-output)
  - [2. Adapt configs](#2-adapt-configs)
  - [3. Containers & images](#3-containers--images)
- [Origin](#origin)

## Overview

The `[COMMUNITY_NAME]` has organized a number of ***benchmarking events*** to evaluate the performance of different tasks that the methods of interest (i.e. ***participants***) might be able to perform. A method can participate in one or several events, depending on its functions.

Within a benchmarking event, one or more ***challenges*** will be performed. A *challenge is primarily defined by the ground truth dataset used for performance assessment*. A challenge is evaluated within a ***benchmarking workflow***, which can be run with either `Docker`, or `Singularity`if implemented. The benchmarking workflow will *compute all metrics relevant for the benchmarking event*. A list of`challenge IDs`and `input files`(i.e. the output files *-prediction files-* of one participant for all specific challenges they may want to participate in) will be passed to the workflow.

In order to compare the performance of the participants within a challenge/event, the respective benchmarking workflow will be run on `input files` from all the eligible participants. The calculated merics will be written to `JSON`files which can be eventually submitted to OEB for database storage.

> *In EuCanImage, the Brain Cancer Diagnosis benchmarking event example may consist of three challenges:  **Detection, Segmentation, and Classification** (Detection and Classification are not implemented yet in the benchmarking workflow of the event). Likewise, a software tool or participant (i.e. AI model inference) can participate in one or several events, depending on its functions.*

## Benchmarking workflow general description

In a first step the provided input files are **validated**. Subsequently, all specified **metrics** are computed, using the matched ground truth files, if applicable. Finally, the **results are gathered** in OEB specific **minimal dataset** `JSON` files per participant.
Based on the created `JSON` files, results can be vizualized on OEB per challenge, such that performance of participants can be compared for each metric.

In order to eventually be compatible with the OEB infrastructure, benchmarking workflows are written in `Nextflow` and are structured in a predefined manner that will be described in the following sections.

> DON'T FREAK OUT IF YOU'RE UNFAMILIAR WITH `NEXTFLOW`! MOST CHANGES YOU'LL MAKE ARE IN `PYTHON`! 😉

### Directory structure for benchmarking workflow code

```bash
[benchmarking-event]/
   |- README.md
   |- main.nf
   |- nextflow.config
   |- parameters_file.config
   |- specification/
        |- example_files/
        |- specification.md
   |- docker_recipes/
        |- README.md
        |- build.sh
        |- validation/
            |- Dockerfile
            |- requirements.txt
            |- validation.py
            |- ...
        |- metrics/
            |- Dockerfile
            |- requirements.txt
            |- compute_metrics.py
            |- ...
        |- consolidation/
            |- Dockerfile
            |- requirements.txt
            |- aggregation.py
            |- merge_data_model_files.py
            |- ...
            |- assessment_chart/
                |- assessment_chart.py
                |- ...  
```

Within such a directory we find the `main.nf` and `nextflow.config` files, which specify the workflow and all its event-specific parameters, respectively, as well as an optional `parameters_file.config`, which contains the *input file(s)*, *participant name* and *challenge ID(s)* for a particular participant and allows running the workflow with it. The `participant name` that you define here and passed as an argument for your *tool/model* will be appearing on OEB plots after uploading the results there.

main.nf`ideally does NOT have to be changed (at least not much) between benchmarking events, as it simply connects the three steps`validation`, `metrics`and`consolidation`inherent to the OEB workflow structure. In contrast, file and tool names have to be adapted in`[participant]_[event].config ` for dedicated workflow runs, as well as other dedicated benchmarking event names used for output files parsing.

> ATTENTION: Keep `nextflow.config` unchanged within an event, in order to be able to directly compare the different participant runs

Within the benchmarking event's directory resides a subdirectory `specification` with a detailed description of required input and output file formats, as well as of the metrics to be calculated for the respective benchmarking event. The *actual code* is hidden in the directory `docker_recipes`; For each of the three benchmarking workflow steps required by OEB, a separate docker container will be built:

1. validation
2. metrics
3. consolidation

The *docker* directories contain Dockerfiles, requirements, constraints, and dedicated python scripts. In order to create datasets that are compatible with the [Elixir Benchmarking Data Model][elixir-data-model], the `input_file`, the ground truth files in the `goldstandar_dir`and the`JSON`template, all actually residing as example in the `input_dir`  of the benchamarking workflow directory (Brain_Cancer_Diagnosis)`baseDir` arguments are received by the respective docker containers. The provided *python scripts* are where the action happens: **These scripts are where you most likely will have to make adjustments for different benchmarking events**.

## (File) naming requirements

### Challenges

Challenge acronym has to be in the specific format as specified by the community. These acronyms will be appearing on OEB plots after uploading the results there.

> *In EuCanImage example it has to be in the form:*

```
ECI_UC[#]_[Detect OR Seg OR Class]
```

> where `ECI`is the acronym of EuCanImage and it is followed by `UC[#]` for the use case number acronym, and followed by the challenge abbreviation (*Detect* for Detection, *Seg* for Segmentation, and *Class* for Classification).

#### Examples:

> ECI_UC8_Detect
> ECI_UC8_Class

### Ground truth files

Depending on the challenge type, the gold standard file(s) MUST be named into different format as the defined by the community.

> *In Brain Cancer Diagnosis *segmentation* challenge example:*

```
gt_[].nii.gz
```

#### Examples:

> gt_IXI002-Guys-0828-T1.nii.gz
>
> gt_IXI015-HH-1258-T1.nii.gz

### Input file (i.e. participant predictions output)

Input file(s) MUST be in the format as specified by the Community. These can be `.txt`, `.csv`, `.tsv`, `NIfTI`,...

In case of any specific naming convention, these requierements may be also specified. The input filename is usually checked during the validation step.

Whenever possible a single file is preferred as input file, but in case of more than one file, these files must share the same metadata to participate within a unique method.

> *In Brain Cancer Diagnosis *segmentation* challenge  example, as several input files have ben compressed to a single `.tar`file, after decompressed the input files must follow the format:*

```
brain_[].nii.gz
```

#### Examples:

> brain_IXI002-Guys-0828-T1.nii.gz
>
> brain_IXI012-HH-1211-T1.nii.gz

Metrics

Metrics names MUST be **exactly** the same in the respective `compute_metrics.py` and `aggregation_minimal_template.json` file of a benchmarking workflow. These metrics names will then appear in the resulting `JSON` files of the workflow, and will be appearing on OEB plots after uploading the results there.

#### Examples:

```
False Positive Rate (FPR)
DSC (Dice Similarity Coefficients) 
```

## HOW TO: DEVELOP

For an example of a benchmarking workflow and further instructions, refer to the [Brain Cancer Diagnosis benchmarking workflow][bcd-bwf] template.

### 1. Copy template

If not done so already, copy the whole contents of the `Brain_Cancer_Diagnosis` directory into the directory for your new benchmarking event. Specify the objectives of your event by adapting the contents of `specification/` .

### 2. Establish proper validation

OEB requires all inputs to be validated.

Consider the `input_file` format and the number of prediction output files to be used as `input_file`. If more than one are to be benchmarked, consider the use of a single compressed file and/or pass a list of input files by using a string to be later parsed, or a e.g. `YAML`or `CSV`file as argument. Update the respective `readme`and/or `specification`.

To check for correct input file formats for your benchmarking event, adapt the validation in [`validation.py`][validation-py].

Update the corresponding `requirements.txt`, `constraints.txt` and `Dockerfile` for installation of additional packages, if necessary.

### 3. Calculate the metrics

Adapt [`compute_metrics.py`][metrics-py] to compare the participant output to the community provided gold standard file(s).

> NOTE: the filename and extension of the gold standard file(s) is hardcoded in [`compute_metrics.py`][metrics-py] for the Segmentation challenge example . Change this according to your benchmarking event.

Update the corresponding `requirements.txt`, `constraints.txt` and `Dockerfile` for installation of additional packages, if applicable.

### 4. Consolidation

The`JSON` outputs from the first two steps will be gathered here, and *aggregation objects* for OEB vizualisation will be created based on the `aggregation_minimal_template.json`. Thus, this is the file you need to adapt in order to control which metrics are plotted and which will be the plots in OEB. You can set visualization types for local plotting in [`manage_assessment_data.py`][assess-py]. The current python scripts have been copied from [TCGA_benchmarking_dockers](https://github.com/inab/TCGA_benchmarking_dockers), and only support 2D plots with x and y axes.

### 5. Adapt `nextflow.config` and `main.nf`

In the former you'll have to adjust the docker container names and general workflow parameters, whereas in the latter you'll only have to make changes if you have introduced new workflow parameters (or want to change the wiring of steps, which is not recommended for the sake of attempted OEB compatibility).

### 6. Don't forget to update `README.md`

Define the file formats and any other specific requirements for your benchmarking workfkow.

Describe the type of validation and metric calculation you perform in the `README.md` in your benchmarking event directory (see example from [Brain Cancer Diagnosis benchmarking workflow][bcd-bwf]).

### 7. Build images

After making the necessary changes for your specific event, you will have to build the docker images locally by either of the following two methods:

1. Go to the `docker_recipes/` directory and run the following
   (note: the `tag_id` should match the one in your `nextflow.config`)

```
run `./build.sh <tag_id>`
```

2. Go to the specific docker directory for each step in `docker_recipes/`:

- `validation/`, `metrics/`, or `consolidation/`
  and run the following

```
docker build . -t [community]/[validation OR metrics OR consolidation]:<tag_id>
```

If you want to update the docker containers, please remove your original images first:

```
docker image ls #look for the IMAGE_ID of your docker image
docker rmi [IMAGE_ID]
```

Then, you can rebuild the docker image locally (see above).

### 8. Test run

This benchmarking workflow can accept the following parameters to be run:

```bash
nextflow run main.nf -profile docker --input {wf.prediction.file} --community_id {community.id} --event_id {event.id} --participant_id {participant.id} --goldstandard_dir {goldstandard.dir} --challenges_ids {analyzed.challenges} --template {template.path} --validated_result {validated.file.path} --assessment_results {assessment_results.file.path} --consolidated_result {consolidated_result.file.path} --outdir {outdir.dir} --statsdir {statsdir.dir}
```

You can use the following command to run the Brain Cancer Diagnosis benchmarking workflow  example with the provided test files from command line:

```
nextflow run main.nf -profile docker --participant_id TOOL --challenges_ids ECI_UC0_Seg  --outdir OUTPUT-DIR

```

# Running with participant arguments and output to desired directory:

```bash
nextflow run main.nf -profile docker --participant_id test_tool  --challenges_ids ECI_UC0_Seg (--outdir your/preferred/outdir/)
# --outdir argument will be redirected to default

# Or for running with configuration file:
nextflow run main.nf -profile docker -c parameters_file.config
# parameters_file.config if not customized will use participant and challenge arguments set by default
```

> NOTE: Parameters from the [nextflow.config][nextflow-config] file are read **in addition** to the ones specified with the `-c` flag, but the latter will override any parameters of the same name in the nextflow.config.  See Nextflow [Configuration](https://www.nextflow.io/docs/latest/config.html) for more information on parameters priority configuration.

## HOW TO: "PRODUCTION"

When you have completed the steps described above you can finally run the benchmarking workflow on real data. Below are some hints to help you get going.

### 1. Get input file(s)(i.e. participant output predictions)

Place the input file(s) into a directory like `inpu_data/participant_dir/` and make sure the files are named as described in the section [Input file (i.e. participant predictions output)](#input-file)

### 2. Adapt configs

You're going to run the workflow for one participant at a time, but you can specify multiple input files and challenges for that participant if you have prepared your workflow scripts accordingly. To do so, create a participant specific `parameters_file.config`. There you'll specify the relevant input files and challenge acronyms.

### 3. Containers & images

Make sure you have the images appropriate for your system ready. If you're running docker you can use the images you built locally in the [HOW TO: DEVELOP](#7-build-images) section. Make sure to rename the images (see bash command below) and adjust the paths in the `nextflow.config` accordingly.

```bash
docker tag [COMMUNITY]/consolidation:tag your_docker_repo/consolidation:[TAG]
docker push [DOCKER-REPO]/consolidation:[TAG]
```

> ATTENTION: always make sure you're using up to date versions of the images. More specifically: DO make sure you have removed old local images and checked your `nextflow.config`

### 4. Run

You can use the following command to run a benchmarking workflow:

```
nextflow run main.nf -profile docker --participant_id [TOOL]  --challenges_ids [CHALLENGE(S)_ACRONYM(S)] (--outdir [OUTDIR/])

# Running with configuration file:
nextflow run main.nf -profile docker -c [PARAMETERS_FILE.config] 


```

## Origin

The Brain Cancer Diagnosis benchmarking workflow is an adaptation of the [APAeval OEB benchmarking workflow ][apa-bwf]. The structure of the output files is compatible with the [ELIXIR Benchmarking Data Model][elixir-data-model], and the workflow is compatible with the OEB VRE setup.

# EuCanImage Brain Cancer Diagnosis Benchmarking Workflow

*OpenEBench based Nextflow workflow for the assessment of a bioinformatics tool's performance in **Brain Cancer Diagnosis** from brain imaging data obtained using magnetic resonance imaging methods (NIfTI standard data format)* in the ECI_UC0_Seg challenge.

---

This README describes the EucanImage Brain Cancer Diagnosis workflow example for an OpenEBench benchmarking workflow. For a more general introduction to benchmarking workflows see the main benchmarking workflow [README.md][readme]. For the specification of metrics, input and output file formats, see [the benchmarks specification][spec].

- [Benchmarking Workflow Parameters](#benchmarking-workflow-parameters)
  - [1. Input data](#1-input-data)
  - [2. Output data](#2-output-data)
- [(File) naming requirements](#file-naming-requirements)
- [Description of steps](#description-of-steps)
  - [1. Validation](#1-validation)
  - [2. Metrics Computation](#2-metrics-computation)
  - [3. Results Consolidation](#3-results-consolidation)
- [Usage](#usage)

## Benchmarking Workflow Parameters

These are the parameters of the EuCanImage Benchmarking Qorkflow (Brain Cancer Diagnosis) compatible with the OpenEBench Benchmarking Qorkflow structure and the Elixir Benchmarking Data Model.

See [OpenEBench Workflow parameters](https://openebench.readthedocs.io/en/latest/technical_references/4_benchmarking_workflows.html?highlight=workflow%20parameters#workflow-parameters) from the [OpenEBench Documentation](https://openebench.readthedocs.io/en/latest/index.html) for more information.

### 1. Input data

*OpenEBench Workflow parameters*

- **community_id**: OpenEBench `community_id`, or `community label` that will be later used for registration in OpenEBench i.e. `OEBC012`/`EuCanImage`.
- **challenges_ids**: list of performance evaluation methods, as OpenEBench `challenges ids` or `challenges labels`,  which are performed in the benchmark. Check step against predefined challenges ids can be executed during the validation phase. This example describes a single challenge, `ECI_UC0_Seg`. These challenge(s) label(s) will be depicted by the OpenEBench widgets.
- **participant_id**: name of the tool/model used to generate the predictions. _The final benchmarking plots are going to display this name_.
- **event_id**: OpenEBench benchmarking `event name` or `event id`, compatible with the Elixir data model, and used in this workflow for minimal dataset generation.
- **input_file**: The predictions' file path(s) (not directory) submitted by the participant. Provided that nextflow cannot handle a parameter list but a string, there are several approaches that can be applied in case of more than one predictions file for one single metadata file (same participant). This example workflow uses a single tar file with all compressed `NIfTI` files The untar of the input file is handled outside the workflow and inside the dockers to simplify the workflow structure, although it can be also performed at workflow level by using an additional process run by nextflow.
- **goldstandard_dir**: directory where the `gold standard` or reference data to compute the metrics are found.
- **public_ref_dir**: (optional) directory which, if necessary, can contain one or more reference files as a guide to benchmark your results, and/or to validate the input data inside your workflow. You can skip this directory if you are not using it.

*Other optional parameters*

- **minimal_aggregation_template.json**: template JSON file compatible with the Elixir Benchmarking Data Model and containing the minimal data needed for the generation locally of the particiapant's final dataset, which integrates the aggregation of the assessment results and the specific visualizations to be generated by the benchmarking workflow. If not provided in the worfklow, the `consolidated_result` file is not generated by the benchmarking workflow locally. 
*Although this parameter is not necessary for the deployment of a Benchmarking Workflow in OpenEBench, this template file helps to the workflow deployment and the obtaining of a consolidated file when the workflow is run offline and with no other participant assessments*.
- **input_data**: Optional directory for all input data.

_OpenEBench parameter(s)'name(s) defintions are case sensitive and will be integrated in the OpenEBench database_.

### 2. Output data

*OpenEBench Workflow parameters*

- **validation_result**: JSON file path where the validated participant is written and corresponds to a minimal dataset compatible with the Elixir Benchmarking Data Model.
- **assessment_results**: JSON file path where the validated participant is written and corresponds to a minimal dataset compatible with the Elixir Benchmarking Data Model.
- **consolidated_result**: JSON file where all the datasets generated during the workflow are merged, which is compatible with the Elixir Benchmarking Data Model, and it is ready to be validated and pushed to Level 1.
- **outdir**: directory where the run results are saved - one or more aggregation files used in the visualization, and several SVG/PNG plots.
- **statsdir**: directory where nextflow statistics (timeline, trace, report,etc.) are written.

*Other optional parameters*

- **otherdir**: optional directory where other community’ specific results can be written. You can skip this directory if you are not using it.
- **output_data**: Optional directory for all output data.

## (File) naming requirements

See description in [the main benchmarking workflow [README.md][readme-naming].

## Description of steps

### 1. Validation

- `input_file`: output file(s) from analysis workflow (predictions file) to be uploaded as input file to benchmark.
- Validation checks performed in [docker_recipes/validation/validation.py][validation-py]:

  - The prediction's file(s)have to start with `brain_` and end with `.nii.gz`. In this workflow, these files are compressed in one single 'tar' file, which is used as input file.
  - Spacing and Dimensions of untarred files from input file and of goldstandar_dir files have to match.

  > NOTE: Other validation operations should be considered when including other challenges in the same benchmarking event provided that these may use different input files (e.g. csv files) to make sure that the challenge you are invoking does match with the input data and the ground truth data.
  >
- Validation status can be 0/1. The workflow breaks if it fails to pass the validation step.

### 2. Metrics Computation

- Decompressed `input files` from tar file, and `groud truth files` loading.
- pair comparison among input and gold files to calculate the metrics.

  - `input_file`: predictions output file from analysis workflow in .nii.tar.gz format.
  - `gold standard`: ground truth nii.gz files.
- EuCanImage custom functions are defined in [docker_recipes/metrics/compute_metrics.py][metrics-py].
- The `assessment_results` file is obtained in the metrics computation step.

### 3. Results Consolidation

- Gathers *all* `validated_[participant].[challenge].[event].json` files from the validation step, *all* `assessments_[participant].[challenge].[event].json` files from the metrics computation step, and aggregation data (from 'minimal_aggregation_template.json' file).
- Outputs OEB compatible `consolidated_result.json` file for the tested participant.
- "aggregation" objects in the `consolidated_result.json` determine which metrics are to be plotted against each other on the OEB website.
- In order to specify which of the metrics present in the assessment objects should be plotted in OEB, the file `inputs/minimal_aggregation_template.json has to be modified accordingly.

## Usage

Please check out the sections on [building docker images][build-images] and [running the benchmarking workflow][run-workflow] in the main EuCanImage benchmarking workflow [README][readme].

[//]: #
[readme]: ./README.md
[readme-naming]: ../README.md#how-to-file-naming-requirements
[build-images]: ./README.md#7-build-images
[run-workflow]: ./README.md#8-test-run
[spec]: ./specification/
[validation-py]: ./docker_recipes/validation/validation.py
[metrics-py]: ./docker_recipes/metrics/compute_metrics.py
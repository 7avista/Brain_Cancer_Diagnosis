# EuCanImage Brain Segmentation: Docker declarations

**Brain Cancer Diagnosis Docker declarations** to be used with the [OEB compatible Nextflow benchmarking workflow][main-nf]*.
---

This README is a brief recap of the three OpenEBench (OEB) benchmarking workflow steps that are computed in their individual containers. For a general introduction to *OpenEBench benchmarking workflows* see the main benchmarking workflow [README][readme]. For a more detailed description on the example EuCanImage Brain Cancer Diagnosis Benchmarking Workflow see [Brain Cancer Diagnosis benchmarking workflow readme][bcd-bwf].

> **NOTE for developers:**
> In order to make the workflow containers reproducible and stable in the long-term, make sure to use specific versions in the container base image (e.g.*ubuntu:20.04*, NOT *ubuntu:latest*).

- [Structure](#structure)
- [Usage](#usage)

## Structure

This benchmarking workflow structure is composed of three docker images / steps:

1. [**Validation**](./validation):
   The input file format is checked and, if required, the content of the file is validated. The validation generates a participant dataset. 
2. [**Metrics**](./metrics):
   During metrics computation predictions are compared with the 'Gold Standards' provided by the community, resulting in a set of quality metrics that reflect the performance of the benchmarking participants. Those metrics are written into assessment datasets. 
3. [**Consolidation**](./consolidation):
   Validation results and assessment metrics of all completed challenges are gathered and written to `consolidated_result.json` (which may include these data from other paticipants as well as found in OEB database). By running this container offline the metrics to be plotted are collected in "aggregation" objects, as to whose specified in `minimal_aggregation_template.json`.

In order to create datasets with structure compatible with the [Elixir Benchmarking Data Model](https://github.com/inab/benchmarking-data-model), please follow the python module and JSON schema. Some example files can be found at [`specification`][spec]

## Usage

Make sure to rename the images (see bash command below) and adjust the paths in the `nextflow.config` accordingly.

### Build images

To build the docker images locally by either of the following two methods:

1. Go to the [`docker_recipes/`] directory and run the following
   (note: the `tag_id` should match the one in your [`nextflow.config`][nextflow-config])

```bash
./build.sh <tag_id>
```

2. Go to the specific docker directory for each step in `docker_recipes/`:

- `validation/`, `metrics/`, or `consolidation/`
  and run the following

```bash
docker build . -t [community]/[validation OR metrics OR consolidation]:<tag_id>
```

### Update images

If you want to update the docker containers, please remove your original images first:

```bash
docker image ls #look for the IMAGE_ID of your docker image
docker rmi [IMAGE_ID]
```

Then, you can rebuild the docker image locally (see above).

Always make sure you're using up to date versions of the images and check the `nextflow.config`.

[//]: #
[main-nf]: ../main.nf
[readme]: ../README.md
[bcd-bwf]: ../EuCanImage_BCD_Benchmarking_Workflow.md
[spec]: ../specification/example_files/
[nextflow-config]: ../nextflow.config
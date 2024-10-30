# EuCanImage Brain Segmentation: Docker declarations

*OpenEBench/EuCanImage benchmarking **Docker declarations** for the **Brain Cancer Diagnosis** benchmarking event; to be used with the [OEB compatible Nextflow benchmarking workflow][main-nf]*.

---

This README is a brief recap of the three OpenEBench (OEB) benchmarking workflow steps that are computed in their individual containers. For a general introduction to **OpenEBench benchmarking workflows** see [the main benchmarking workflow `README.md`][readme-bwf]. For a more detailed description on the example EuCanImage Brain Cancer Diagnosis Benchmarking Workflow see [EuCanImage Brain Cancer Diagnosis benchmarking workflow `README.md`][readme-bcd-bwf].

> **NOTE for developers:**
> In order to make the workflow containers reproducible and stable in the long-term, make sure to use specific versions in the container base image (e.g.*ubuntu:20.04*, NOT *ubuntu:latest*).

- [Structure](#structure)
- [Usage](#usage)

## Structure

This benchmarking workflow structure is composed of three docker images / steps:

1. [**Validation**](./validation):
   The input file format is checked and, if required, the content of the file is validated. The validation generates a participant dataset. In order to create datasets with structure compatible with the [Elixir Benchmarking Data Model][elixir-data-model], please use the following [python module and JSON schema][oeb-json].
2. [**Metrics**](./metrics):
   During metrics computation predictions are compared with the 'Gold Standards' provided by the community, resulting in a set of quality metrics that reflect the performance of the benchmarking participants. Those metrics are written into assessment datasets. In order to create datasets with structure compatible with the [Elixir Benchmarking Data Model][elixir-data-model], please use the following [python module and JSON schema][oeb-json].
3. [**Consolidation**](./consolidation):
   Validation results and assessment metrics of all completed challenges are gathered and written to `consolidated_result.json` (which may include these data from other paticipants in OEB database). By running this container offline the metrics to be plotted are collected in "aggregation" objects, as to whose specified in `minimal_aggregation_template.json`.

## Usage

Please check out the sections on [building docker images][build-images] and [running the benchmarking workflow][run-workflow] in the main [OpenEBench (OEB) Benchmarking Workflows README][readme-bwf].


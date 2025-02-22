# Deep Learning Model Extension Specification

[![hackmd-github-sync-badge](https://hackmd.io/lekSD_RVRiquNHRloXRzeg/badge)](https://hackmd.io/lekSD_RVRiquNHRloXRzeg?both)

- **Title:** Deep Learning Model Extension
- **Identifier:** <https://schemas.stacspec.org/v1.0.0-beta.3/extensions/dl-model/json-schema/schema.json>
- **Field Name Prefix:** dlm
- **Scope:** Item, Collection
- **Extension [Maturity Classification][stac-ext-maturity]:** Proposal
- **Owner**:
  [@sfoucher](https://github.com/sfoucher)
  [@fmigneault](https://github.com/fmigneault)
  [@ymoisan](https://github.com/ymoisan)

[stac-ext-maturity]: https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity

This document explains the Template Extension to the [SpatioTemporal Asset Catalog][stac-spec] (STAC) specification.
This document explains the fields of the STAC Deep Learning Model (dlm) Extension to a STAC Item. 
The main objective is to be able to build model collections that can be searched 
and that contain enough information to be able to deploy an inference service.
When Deep Learning models are trained using satellite imagery, it is important
to track essential information if you want to make them searchable and reusable:
1. Input data origin and specifications
2. Model base transforms
3. Model output and its semantic interpretation
4. Runtime environment to be able to run the model
5. Scientific references

[stac-spec]: https://github.com/radiantearth/stac-spec

Check the original technical report
[here](https://github.com/crim-ca/CCCOT03/raw/main/CCCOT03_Rapport%20Final_FINAL_EN.pdf) for more details.

![](https://i.imgur.com/cVAg5sA.png)

- Examples:
  - [Example with a UNet trained with thelper](examples/item.json)
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Item Properties and Collection Fields

| Field Name       | Type                                        | Description                                                            |
|------------------|---------------------------------------------|------------------------------------------------------------------------|
| dlm:data         | [Data Object](#data-object)                 | Describes the EO data compatible with the model.                       |
| dlm:inputs       | [Inputs Object](#inputs-object)             | Describes the transformation between the EO data and the model inputs. |
| dlm:architecture | [Architecture Object](#architecture-object) | Describes the model architecture.                                      |
| dlm:runtime      | [Runtime Object](#runtime-object)           | Describes the runtime environments to run the model (inference).       |
| dlm:outputs      | [Outputs Object](#outputs-object)           | Describes each model output and how to interpret it.                   |

In addition, fields from the following extensions must be imported in the item:
- [Scientific Extension Specification][stac-ext-sci] to describe relevant publications.
- [EO Extension Specification][stac-ext-eo] to describe eo data.
- [Version Extension Specification][stac-ext-ver] to define version tags.

[stac-ext-sci]: https://github.com/radiantearth/stac-spec/tree/v1.0.0-beta.2/extensions/scientific/README.md
[stac-ext-eo]: https://github.com/radiantearth/stac-spec/tree/v1.0.0-beta.2/extensions/eo/README.md
[stac-ext-ver]: https://github.com/radiantearth/stac-spec/tree/v1.0.0-beta.2/extensions/version/README.md

### Data Object

| Field Name      | Type                                       | Description                                                                                                                   |
|-----------------|--------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| process_level   | [Process Level Enum](#process-level-enum)  | Data processing level that represents the apparent variability of the data.                                                   |
| data_type       | [Data Type Enum](#data-type-enum)          | Data type (`uint8`, `uint16`, etc.) enum based on numpy base types for data normalization and pre-processing.                 |
| nodata          | integer \| string                          | Value indicating *nodata*, which could require special data preparation by the network (see [No Data Value](#no-data-value)). |
| number_of_bands | integer                                    | Number of bands used by the model                                                                                             |
| useful_bands    | \[[Model Band Object](#model-band-object)] | Describes bands by index in the relevant order for the model input.                                                           |

#### Process Level Enum

It is recommended to use the [STAC Processing Extension][stac-ext-proc]
to represent the `processing:level` of the relevant level `L0` for raw data up to `L4` for Analysis-Ready Data (ARD).

[stac-ext-proc]: https://github.com/stac-extensions/processing#suggested-processing-levels

#### Data Type Enum

It is recommended to use the [STAC Raster Extension - Raster Band Object][stac-ext-raster-band-obj]
in STAC Collections and Items that refer to a STAC Item using `dlm`'s `data_type`. The values should be one of the known
data types defined by `raster:bands`'s `data_type` as presented in [Data Types][stac-ext-raster-dtype].

If source imagery has different `data_type` values than the one specified by `dlm`'s `data_type` property,
this should provide
an indication that the source imagery might require a preprocessing step (scaling, normalization, conversion, etc.)
to adapt the samples to the relevant format expected by the described model.

[stac-ext-raster-dtype]: https://github.com/stac-extensions/raster/#data-types
[stac-ext-raster-band-obj]: https://github.com/stac-extensions/raster/#raster-band-object

#### No Data Value

It is recommended to use the [STAC Raster Extension - Raster Band Object](https://github.com/stac-extensions/raster/#raster-band-object)
in STAC Collections and Items that refer to a STAC Item using `dlm`'s `nodata`. This value should either map
to the `raster:bands`'s `nodata` property of relevant bands, or a classification label value representing
a "*background*" pixel mask (see [STAC Label Extension - Raster Label Notes][stac-ext-raster-label])
from a `label:type` defined as `raster` with the relevant `raster` asset provided.

If source imagery has different `nodata` values than the one specified by `dlm`'s `nodata` property, this should provide
an indication that the source imagery might require a preprocessing step to adapt the samples to the values expected by
the described model.

[stac-ext-raster-label]: https://github.com/stac-extensions/label#raster-label-notes

#### Model Band Object

Can be combined with `eo:bands`'s [`Band Object`][stac-ext-eo-band-obj].

[stac-ext-eo-band-obj]: https://github.com/stac-extensions/eo#band-object

| Field Name      | Type    | Description                              |
|-----------------|---------|------------------------------------------|
| index           | integer | **REQUIRED** Index of the spectral band. |
| name            | string  | Short name of the band for convenience.  |

### Inputs Object

| Field Name              | Type                            | Description                                                                                                                             |
|-------------------------|---------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| name                    | string                          | Python name of the input variable.                                                                                                      |
| input_tensors           | [Tensor Object](#tensor-object) | Shape of the input tensor ($N \times C \times H \times W$).                                                                             |
| scaling_factor          | number                          | Scaling factor to apply to get data within `[0,1]`. For instance `scaling_factor=0.004` for 8-bit data.                                 |
| normalization:mean      | list of numbers                 | Mean vector value to be removed from the data. The vector size must be consistent with `input_tensors:dim` and `selected_bands`.        |
| normalization:std       | list of numbers                 | Standard-deviation values used to normalize the data. The vector size must be consistent with `input_tensors:dim` and `selected_bands`. |
| selected_band           | list of integers                | Specifies the bands selected from the data described in dlm:data.                                                                       |
| pre_processing_function | string                          | Defines a python pre-processing function (path and inputs should be specified).                                                         |

#### Tensor Object

| Field Name | Type   | Description                         |
|------------|--------|-------------------------------------|
| batch      | number | Batch size dimension (must be > 0). |
| dim        | number | Number of channels  (must be > 0).  |
| height     | number | Height of the tensor (must be > 0). |
| width      | number | Width of the tensor (must be > 0).  |

### Architecture Object

| Field Name              | Type    | Description                                                 |
|-------------------------|---------|-------------------------------------------------------------|
| total_nb_parameters     | integer | Total number of parameters.                                 |
| estimated_total_size_mb | number  | The equivalent memory size in MB.                           |
| type                    | string  | Type of network (ex: ResNet-18).                            |
| summary                 | string  | Summary of the layers, can be the output of `print(model)`. |
| pretrained              | string  | Indicates the source of the pretraining (ex: ImageNet).     |

### Runtime Object

| Field Name        | Type                               | Description                                                                  |
|-------------------|------------------------------------|------------------------------------------------------------------------------|
| framework         | string                             | Used framework (ex: PyTorch, TensorFlow).                                    |
| version           | string                             | Framework version (some models require a specific version of the framework). |
| model_handler     | string                             | Inference execution function.                                                |
| model_src_url     | string                             | Url of the source code (ex: GitHub repo).                                    |
| model_commit_hash | string                             | Hash value pointing to a specific version of the code.                       |
| docker            | \[[Docker Object](#docker-object)] | Information for the deployment of the model in a docker instance.            |

#### Docker Object

| Field Name  | Type    | Description                                           |
|-------------|---------|-------------------------------------------------------|
| docker_file | string  | Url of the Dockerfile.                                |
| image_name  | string  | Name of the docker image.                             |
| tag         | string  | Tag of the image.                                     |
| working_dir | string  | Working directory in the instance that can be mapped. |
| run         | string  | Running command.                                      |
| gpu         | boolean | True if the docker image requires a GPU.              |

### Outputs Object

| Field Name               | Type                    | Description                                                                                                                                                                                                                                                                          |
|--------------------------|-------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| task                     | [Task Enum](#task-enum) | Specifies the Machine Learning task for which the output can be used for.                                                                                                                                                                                                            |
| number_of_classes        | integer                 | Number of classes.                                                                                                                                                                                                                                                                   |
| final_layer_size         | \[integer]              | Sizes of the output tensor as ($N \times C \times H \times W$).                                                                                                                                                                                                                      |
| class_name_mapping       | list                    | Mapping of the output index to a short class name, for each record we specify the index and the class name.                                                                                                                                                                          |
| dont_care_index          | integer                 | Some models are using a *do not  care* value which is ignored in the input data. This is an optional parameter.                                                                                                                                                                      |
| post_processing_function | string                  | Some models are using a complex post-processing that can be specified using a post processing function. The python package should be specified as well as the input and outputs type. For example:`my_python_module_name:my_processing_function(Tensor<BxCxHxW>) -> Tensor<Bx1xHxW>` |

#### Task Enum

It is recommended to define `task` with one of the following values:
- `regression`
- `classification`
- `object detection`
- `segmentation` (generic)
- `semantic segmentation`
- `instance segmentation`
- `panoptic segmentation`

This should align with the `label:tasks` values defined in [STAC Label Extension][stac-ext-label-props] for relevant
STAC Collections and Items employed with the model described by this extension.

[stac-ext-label-props]: https://github.com/stac-extensions/label#item-properties

## Relation types

The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

| Type           | Description                           |
|----------------|---------------------------------------|
| fancy-rel-type | This link points to a fancy resource. |

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct][stac-spec-code-conduct].
For contributions, please follow the
[STAC specification contributing guide][stac-spec-contrib-guide] Instructions
for running tests are copied here for convenience.

[stac-spec-code-conduct]: https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md
[stac-spec-contrib-guide]: https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md

### Running tests

The same checks that run as checks on PRs are part of the repository and can be run locally to verify
that changes are valid.
To run tests locally, you'll need `npm`, which is a standard part of any [node.js][nodejs] installation.

[nodejs]: https://nodejs.org/en/download/

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check Markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```

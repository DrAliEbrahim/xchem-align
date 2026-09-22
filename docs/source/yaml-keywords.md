# YAML Configuration and Keywords

## config.yaml keywords

This page describes the keywords available in the XChem Align `config.yaml` file.

The configuration file is validated against the built-in XChem Align configuration schema. The schema defines the structure and permitted keywords, while some additional requirements and default values are enforced by the XChem Align runtime.

The current configuration keyword hierarchy is as follows:
```
config.yaml
│
├── target_name
├── base_dir
├── copy_dir
├── extra_files_dir
├── ref_datasets
├── statuses
├── panddas_missing_ok
├── covalent
│
└── inputs
    ├── dir
    ├── type
    │   ├── model_building
    │   │   ├── soakdb
    │   │   ├── panddas_event_files
    │   │   └── sequences
    │   │       ├── dir
    │   │       ├── default
    │   │       └── variants
    │   │           ├── sequence
    │   │           └── crystals
    │   │
    │   └── manual
    │
    ├── code_prefix
    ├── code_prefix_tooltip
    └── exclude
 
```

> **Note:** Some keywords are optional in the schema but are required for particular workflows. Where this occurs, it is noted below.

| Keyword              | Type            | Required | Default                      |
| -------------------- | --------------- | -------- | ---------------------------- |
| `target_name`        | string          | Yes      | —                            |
| `base_dir`           | string          | Yes      | —                            |
| `copy_dir`           | string          | No       | —                            |
| `extra_files_dir`    | string          | No       | `upload-current/extra_files` |
| `ref_datasets`       | list of strings | No       | `[]`                         |
| `statuses`           | list of strings | No       | `["4", "5", "6"]`            |
| `panddas_missing_ok` | list of strings | No       | `[]`                         |
| `covalent`           | string          | No       | —                            |
| `inputs`             | list of objects | Yes      | —                            |
| `overrides`          | object          | No       | —                            |

---

### `target_name`

The name of the target being processed.

```yaml
target_name: Mpro
```

**Type:**

`string`

**Required:**

Yes.

**Restrictions**

The target name must:

* contain at least four characters;
* begin with a letter;
* contain only letters, numbers, `_` and `-`.

---

### `base_dir`

The base directory from which input paths are resolved.

```yaml
base_dir: data/inputs
```

**Type:**

`string`

**Required:**

Yes.

`base_dir` must exist and must be a directory.

Paths specified by `inputs.dir` are interpreted relative to `base_dir`.

For example:

```yaml
base_dir: data/inputs

inputs:
  - dir: experiment_1
```

refers to:

```yaml
data/inputs/experiment_1/
```

---

### `copy_dir`

Specifies a copy directory.

```yaml
copy_dir: data/inputs/copied
```

**Type:**

`string`

**Required:**

No.

> **Note**: `copy_dir` is retained in the current configuration schema but does not appear to control the output directory in the current collator implementation. The current output location is determined by the XChem Align working directory and its `upload-current` directory.

---

### `extra_files_dir`

Specifies a directory containing additional files that should be copied into the XChem Align output.

```yaml
extra_files_dir: data/extra_files
```

**Type:**

`string`

**Required:**

No.

**Default:**

If omitted, XChem Align looks for:

```yaml
upload-current/extra_files/
```

The contents of the directory are copied into the `extra_files` directory of the current upload version.

If the directory does not exist, XChem Align reports a warning.

---

### `ref_datasets`

Defines datasets that should be treated as reference datasets.

```yaml
ref_datasets:
  - Mpro-x0001
  - Mpro-x0002
```

**Type:**

`list[string]`

**Required:**

No.

Datasets listed here are marked as reference datasets in the generated metadata.

Reference datasets are also required when an assembly or crystal form in `assemblies.yaml` uses them as its `reference`.

---

### `statuses`

Controls which SoakDB refinement statuses are included when processing `model_building` inputs.

```yaml
statuses:
  - "4"
  - "5"
  - "6"
```

**Type:**

`list[string]`

**Required:**

No.

**Default:**

```yaml
["4", "5", "6"]
```

The standard XChem statuses are:

| Status | Meaning             |
| ------ | ------------------- |
| `1`    | Analysis Pending    |
| `2`    | PANDDA model        |
| `3`    | In Refinement       |
| `4`    | CompChem ready      |
| `5`    | Deposition ready    |
| `6`    | Deposited           |
| `7`    | Analysed & Rejected |

The values are matched against the beginning of the `RefinementOutcome` field in SoakDB.

Datasets listed in `ref_datasets` are included independently of this setting.

Datasets with status `7` are also handled separately as rejected datasets.

---

### `panddas_missing_ok`

Specifies datasets for which missing PanDDA event maps should not be treated as an error.

```yaml
panddas_missing_ok:
  - Mpro-x0123
  - Mpro-x0456
```

**Type:**

`list[string]`

**Required:**

No.

**Default:**

```yaml
[]
```

When XChem Align identifies a ligand but cannot find a corresponding PanDDA event map, the normal behaviour is to report an error.

If the crystal is included in `panddas_missing_ok`, the missing event map is permitted.

This option should therefore only be used for datasets where the absence of a PanDDA event map is intentional.

---

### `covalent`

Specifies covalent-processing information.

```yaml
covalent: ...
```

**Type:**

`string`

**Required:**

No.

**Note:**

`covalent` is present in the current XChem Align configuration schema, but the current collator implementation does not appear to consume this value.

It should therefore be considered a reserved or legacy configuration option unless a specific XChem Align workflow provides additional handling for it.

---

### `inputs`

The `inputs` section defines the data sources that XChem Align should process.

```yaml
inputs:
  - dir: experiment_1
    type: model_building
```

**Type:**

`list[object]`

**Required:**

Yes.

At least one input must be provided.

Multiple input sources can be specified:

```yaml
inputs:
  - dir: experiment_1
    type: model_building

  - dir: manually_added
    type: manual
```

Each input must have:

* `dir`
* `type`

The available input types are:

* `model_building`
* `manual`

---

#### `inputs.dir`

Specifies the directory containing the input data.

```yaml
inputs:
  - dir: dls/labxchem/data/2020/lb18145-153
    type: model_building
```

**Type:**

`string`

**Required:**

Yes.

The path must be relative to `base_dir`.

Absolute paths are not permitted.

For `model_building` inputs, XChem Align expects the model-building data beneath the input directory.

For `manual` inputs, XChem Align searches the specified directory directly for PDB, MTZ and CIF files.

---

#### `inputs.type`

Specifies the type of input data.

```yaml
type: model_building
```

**Type:**

`string`

**Required:**

Yes.

**Allowed values:**

| Value            | Description                                                                           |
| ---------------- | ------------------------------------------------------------------------------------- |
| `model_building` | Input generated from an XChem model-building workflow and associated SoakDB database. |
| `manual`         | Input consisting of manually supplied PDB, MTZ and CIF files.                         |

---

#### `inputs.code_prefix`

Specifies the prefix associated with datasets from this input.

```yaml
code_prefix: Mpro
```

**Type:**

`string`

**Required:**

No.

---

#### `inputs.code_prefix_tooltip`

Provides descriptive tooltip text shown in Fragalysis.

```yaml
code_prefix_tooltip: "SARS-CoV-2 Mpro fragment screen"
```

**Type:**

`string`

**Required:**

No.

**Usage:**

Required for `model_building` inputs.

The tooltip is stored in the generated metadata alongside the corresponding code prefix.

If the same prefix is used with different tooltips across multiple inputs, XChem Align reports a warning.

---

#### `inputs.soakdb`

Specifies the SoakDB SQLite database associated with a `model_building` input.

```yaml
soakdb: processing/database/soakDBDataFile.sqlite
```

**Type:**

`string`

**Required:**

Yes, with the `model_building` input.

**Usage:**

The path is interpreted relative to the input directory.

For example:

```yaml
base_dir: data/inputs

inputs:
  - dir: experiment_1
    type: model_building
    soakdb: processing/database/soakDBDataFile.sqlite
```

refers to:

```text
data/inputs/experiment_1/processing/database/soakDBDataFile.sqlite
```

For `model_building` inputs, the resulting database file **must exist**.

---

#### `inputs.exclude`

Specifies datasets that should not be processed.

```yaml
exclude:
  - Mpro-x0123
  - Mpro-x0456
```

**Type:**

`list[string]`

**Required:**

No.

**Usage:**

For `model_building` inputs, the values are matched against crystal names retrieved from SoakDB.

For `manual` inputs, the values are matched against the names of the input PDB files.

This is useful for excluding datasets that are present in the source data but should not be included in the final output.

---

#### `inputs.panddas_event_files`

Specifies one or more PanDDA event CSV files.

```yaml
panddas_event_files:
  - processing/analysis/pandda_1/analyses/pandda_inspect_events.csv
  - processing/analysis/pandda_2/analyses/pandda_inspect_events.csv
```

**Type:**

`list[string]`

**Required:**

No.

**Usage:**

The files provide PanDDA event information used to associate ligand observations with event maps.

Multiple event files may be supplied, allowing event information from multiple PanDDA runs to be considered.

This option is applicable to `model_building` inputs.

---

### `sequences`

Defines the protein sequences present in the crystallographic models.

`sequences` is available for `model_building` inputs.

```yaml
sequences:
  dir: processing/analysis/sequences
  default: default.fa
```

**Type:**

`object`

**Required:**

No.

The `sequences` section is optional during the collator workflow. Sequence information is subsequently important for workflows such as PDB deposition.

---

#### `sequences.dir`

Specifies the directory containing the FASTA files.

```yaml
sequences:
  dir: processing/analysis/sequences
```

**Type:**

`string`

**Required:**

No.

**Usage:**

The directory is relative to the input directory.

---

#### `sequences.default`

Specifies the default FASTA file.

```yaml
sequences:
  default: default.fa
```

**Type:**

`string`

**Required:**

No.

**Usage:**

The default sequence is used for crystals that do not have a sequence variant assigned to them.

---

#### `sequences.variants`

Defines alternative sequence files for specific crystals.

```yaml
sequences:
  variants:
    - sequence: variant_1.fa
      crystals:
        - Mpro-x0001
        - Mpro-x0002
```

**Type:**

`list[object]`

**Required:**

No.

Each variant associates one FASTA file with one or more crystal names.

---

#### `sequences.variants.sequence`

Specifies the FASTA file containing an alternative sequence.

```yaml
sequence: variant_1.fa
```

**Type:**

`string`

**Required:**

Only when a vairant is present.

The file is resolved relative to `sequences.dir`.

---

#### `sequences.variants.crystals`

Lists the crystals that use the corresponding sequence variant.

```yaml
crystals:
  - Mpro-x0001
  - Mpro-x0002
```

**Type:**

`list[string]`

**Required:**

The schema requires at least one crystal if the `crystals` property is supplied.

Each listed crystal is assigned the sequence from the corresponding variant FASTA file.

---

### Configuration example

A typical `model_building` configuration can therefore look like:

```yaml
target_name: Mpro

base_dir: data/inputs

ref_datasets:
  - Mpro-x0001

statuses:
  - "4"
  - "5"
  - "6"

panddas_missing_ok:
  - Mpro-x0123

inputs:
  - dir: dls/labxchem/data/2020/lb18145-153
    type: model_building

    code_prefix: Mpro
    code_prefix_tooltip: "SARS-CoV-2 Mpro fragment screen"

    soakdb: processing/database/soakDBDataFile.sqlite

    panddas_event_files:
      - processing/analysis/pandda_1/analyses/pandda_inspect_events.csv
      - processing/analysis/pandda_2/analyses/pandda_inspect_events.csv

    exclude:
      - Mpro-x0999

    sequences:
      dir: processing/analysis/sequences
      default: default.fa

      variants:
        - sequence: variant_1.fa
          crystals:
            - Mpro-x0100
            - Mpro-x0101
```
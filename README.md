# RT-Datasets
Datasets For Cross Band CSI Reconstruction by Sionna RT


This repository provides the ray-tracing (RT) subset used in our cross-band CSI
reconstruction experiments. The dataset contains raw multi-path component (MPC)
files for four RT scenes and five carrier frequencies. It is intended for
research on multi-band MIMO-OFDM channel modeling, cross-band CSI reconstruction,
and sparse target-pilot recovery.

The dataset files are hosted externally because they are larger than typical
GitHub repository files. This GitHub repository provides the dataset description,
metadata, configuration files, and citation information.

## Download

The raw RT channel files can be downloaded from SJTU Pan:

```text
Share name: RT_Channel_Datas
URL: https://pan.sjtu.edu.cn/web/share/7b6db27da53809e35968b5b3f8e71f10
```

After downloading, place the `.mat` files and `dataset_radio_config.json` under
one dataset root, for example:

```text
Data/
  dataset_radio_config.json
  channel_data_2.4GHz_BME.mat
  channel_data_3.5GHz_BME.mat
  ...
  channel_data_40.0GHz_LA_scene4.mat
```

## Dataset Overview

The RT subset contains four scenes:

| Scene key | Paper name | Source | Samples per band |
| --- | --- | --- | ---: |
| `BME` | BME | Shanghai Jiao Tong University scene, map extracted from OpenStreetMap | 56,021 |
| `D4` | D4 | Shanghai Jiao Tong University scene, map extracted from OpenStreetMap | 54,986 |
| `EMB` | EMB | Shanghai Jiao Tong University scene, map extracted from OpenStreetMap | 30,000 |
| `LA_scene4` | LA | Self-built LA scene | 62,482 |

For each scene, channels are provided at 2.4, 3.5, 15, 28, and 40 GHz. The
same sample index corresponds to the same user/base-station geometry across
available bands in a scene, which enables source-target frequency pairing.

All RT channels were generated with Sionna RT. BME, D4, and EMB use
OpenStreetMap-derived scene maps; LA is a self-built scene. When using this
dataset, please cite this dataset release and also acknowledge the upstream tools
and data sources listed in the citation section.

## File Layout

The release is organized as raw MATLAB files plus metadata:

```text
xband-rt-dataset/
  README.md
  dataset_radio_config.json
  data/
    channel_data_2.4GHz_BME.mat
    channel_data_3.5GHz_BME.mat
    channel_data_15.0GHz_BME.mat
    channel_data_28.0GHz_BME.mat
    channel_data_40.0GHz_BME.mat
    channel_data_2.4GHz_D4.mat
    ...
    channel_data_40.0GHz_LA_scene4.mat
```

The GitHub repository itself does not store the raw `.mat` files. Download them
from the link above and unpack them so that the `.mat` files and
`dataset_radio_config.json` are in the same dataset root.

## Raw Data Format

Each `.mat` file stores one scene at one carrier frequency. The file naming
pattern is:

```text
channel_data_<freq>GHz_<scene>.mat
```

Example:

```text
channel_data_28.0GHz_EMB.mat
```

The MATLAB files use the MPC format consumed by the accompanying code. Each file
contains at least the following arrays:

| Field | Shape | Description |
| --- | --- | --- |
| `mpc_flat` | `[N_paths_total, 6]` | Flattened MPC table. Columns are `[sample_id, tau, theta_t, phi_t, alpha_real, alpha_imag]`. |
| `sample_offsets` | `[N_samples + 1]` | Start/end offsets into `mpc_flat` for each sample. |
| `sample_table` | `[N_samples, 10]` | Per-sample metadata such as sample, BS, UE, and position identifiers. |
| `meta` | optional | Scene-level metadata, if present. |

The raw files store MPCs rather than precomputed dense CSI. The dense MIMO-OFDM
CSI matrices are synthesized from the MPCs using the RF configuration below.

## RF and CSI Configuration

The base station uses a uniform planar array (UPA). Antenna spacing is set to
half wavelength at the corresponding carrier frequency. The native CSI and PADS
configuration used in the paper is:

| Band | `Nc` | UPA | `Nt` | Subcarrier spacing | PADS grid | Max delay |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 2.4 GHz | 64 | 4 x 4 | 16 | 60 kHz | 256 x 256 | 16.67 us |
| 3.5 GHz | 64 | 4 x 4 | 16 | 90 kHz | 256 x 256 | 11.11 us |
| 15 GHz | 128 | 8 x 4 | 32 | 90 kHz | 256 x 256 | 11.11 us |
| 28 GHz | 128 | 8 x 8 | 64 | 180 kHz | 256 x 256 | 5.56 us |
| 40 GHz | 128 | 8 x 8 | 64 | 180 kHz | 256 x 256 | 5.56 us |

The configuration is also provided in `dataset_radio_config.json`. The paper
uses this configuration to synthesize dense CSI, build sparse target pilots, and
map CSI to a common power-angle-delay spectrum (PADS) grid.

## Using the Dataset with the Paper Code

Place the dataset root under `Data/` or pass its location through the command
line arguments used by the code.

Smoke test:

```bash
python dataset.py \
  --data-dir Data \
  --config Data/dataset_radio_config.json
```

Build a small debug CSI cache:

```bash
python build_csi_cache.py \
  --data-dir Data \
  --config Data/dataset_radio_config.json \
  --output-dir Data/CSI_cache_debug \
  --max-files 1 \
  --max-samples-per-file 64 \
  --shard-size 16
```

Build the full CSI cache:

```bash
python build_csi_cache.py \
  --data-dir Data \
  --config Data/dataset_radio_config.json \
  --output-dir Data/CSI_cache \
  --min-paths 1 \
  --max-paths-for-synthesis 128 \
  --shard-size 4096
```

The generated cache stores dense CSI as real and imaginary arrays with shape
`[N, Nc, Nt]`, together with sample identifiers and RF metadata. PADS conversion
can then be performed online during training or evaluation.

## How to Cite

In the paper, cite the dataset release when describing the RT subset, for
example:

```latex
The RT subset is released as the RT Cross-Band CSI Dataset~\cite{xband_rt_dataset}.
```

Use the following BibTeX entry after replacing the placeholders:

```bibtex
@misc{xband_rt_dataset,
  author       = {Hongpu Zhang, Xiangwen Gu, Shu Sun},
  title        = {{Cross Band RT Cross-Band CSI Dataset}},
  year         = {2026},
  version      = {v1.0.0},
  howpublished = {Online. Available: https://github.com/Zetahp/RT-Datasets},
  note         = {Data available at https://pan.sjtu.edu.cn/web/share/7b6db27da53809e35968b5b3f8e71f10. Accessed: 2026-06-14}
}
```


Please also cite the relevant upstream resources when appropriate:

```bibtex
@article{hoydis2023sionna_rt,
  author  = {J. Hoydis and F. Ait Aoudia and S. Cammerer and F. Euchner and M. Nimier-David and S. ten Brink and A. Keller},
  title   = {{Sionna RT}: Differentiable Ray Tracing for Radio Propagation Modeling},
  journal = {arXiv preprint arXiv:2303.11103},
  year    = {2023}
}

@article{haklay2008openstreetmap,
  author  = {M. Haklay and P. Weber},
  title   = {{OpenStreetMap}: User-Generated Street Maps},
  journal = {IEEE Pervasive Computing},
  volume  = {7},
  number  = {4},
  pages   = {12--18},
  year    = {2008}
}
```

## Attribution

```text
This dataset includes RT scenes constructed from OpenStreetMap-derived map data
for BME, D4, and EMB, and a self-built LA scene. Ray-tracing channels were
generated with Sionna RT.
```

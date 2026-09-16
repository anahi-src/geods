# GeoDS

A reproducible conda environment for geospatial data science in Python — GeoPandas, rasterio and the
scientific Python stack, plus JupyterLab, pinned in a single environment file.

> [!WARNING]
> **The environment files in this repository currently do not solve.** Several pins
> (`numpy=1.19.2`, `scipy=1.7`, `tensorflow==2.6`, …) predate `python=3.12` and have no
> matching build. See [Known issues](#known-issues) before filing a bug.

---

## Contents

- [Requirements](#requirements)
- [Which file do I use?](#which-file-do-i-use)
- [Quick start](#quick-start)
- [Installation](#installation) — [macOS](#macos) · [Windows](#windows) · [Linux](#linux)
- [Verify the installation](#verify-the-installation)
- [Using the environment](#using-the-environment)
- [Updating and removing](#updating-and-removing)
- [What's in the environment](#whats-in-the-environment)
- [Known issues](#known-issues)
- [Troubleshooting](#troubleshooting)

---

## Requirements

A conda-compatible package manager. **[Miniforge](https://github.com/conda-forge/miniforge) is
recommended** — it is a minimal installer that defaults to the `conda-forge` channel, which is where
all the geospatial packages in this environment come from. Mixing `conda-forge` with Anaconda's
`defaults` channel is [explicitly discouraged by conda-forge](https://conda-forge.org/docs/user/transitioning_from_defaults/),
and `defaults` carries Anaconda's commercial terms of service, which many institutions cannot accept.

Miniconda or a full Anaconda install will also work, but you should set channel priority first — see
[Troubleshooting](#troubleshooting).

<details>
<summary><b>New to conda? Start here</b></summary>

Conda is a package and environment manager. An *environment* is an isolated folder containing its own
Python interpreter and libraries, so a project's dependencies can't collide with another project's.

1. Download the Miniforge installer for your operating system from
   [github.com/conda-forge/miniforge](https://github.com/conda-forge/miniforge#download).
2. Run it and accept the defaults. On macOS and Linux this is a `.sh` script you run in the terminal;
   on Windows it is a `.exe`.
3. Close and reopen your terminal. On Windows, use the **Miniforge Prompt** that the installer adds to
   the Start menu.
4. Check it worked:

   ```bash
   conda --version
   ```

You do not need to install Python separately — conda provides it.

</details>

---

## Which file do I use?

| Platform | Architecture | File |
| --- | --- | --- |
| macOS | Apple silicon (M1/M2/M3/M4) | `geodsM1ARM.yml` |
| macOS | Intel | `geods.yml` |
| Windows | x86-64 | `geods.yml` |
| Linux | x86-64 / aarch64 | `geods.yml` |

Both files declare the same environment name, `geods`, and differ only in their TensorFlow pins: the
Apple silicon variant uses `tensorflow-macos` with the `tensorflow-metal` GPU plugin.

<details>
<summary><b>Not sure which Mac you have?</b></summary>

Run this in the terminal:

```bash
uname -m
```

`arm64` means Apple silicon — use `geodsM1ARM.yml`. `x86_64` means Intel — use `geods.yml`.

If you get `x86_64` on a Mac you believe is Apple silicon, your terminal is running under Rosetta.
</details>

---

## Quick start

```bash
git clone https://github.com/anahi-src/geods.git
cd geods

# Apple silicon Macs: swap geods.yml for geodsM1ARM.yml
conda env create --file geods.yml
conda activate geods
```

> [!NOTE]
> The command is `conda env create` — with **env** — not `conda create`. `conda create --file` expects a
> plain list of package specs and will fail on a YAML environment file.

---

## Installation

The steps are the same on all three platforms; the differences are noted below.

### macOS

```bash
# Apple silicon
conda env create --file geodsM1ARM.yml

# Intel
conda env create --file geods.yml

conda activate geods
```

<details>
<summary><b>macOS notes</b></summary>

- **Apple silicon.** Install the `arm64` Miniforge build. Running an `x86_64` conda under Rosetta will
  silently give you emulated, much slower NumPy and GDAL.
- **GPU acceleration.** `tensorflow-metal` routes TensorFlow onto the Apple GPU. It is optional — if it
  causes problems, remove it from the environment file and TensorFlow will fall back to the CPU.
- **Xcode command line tools** are not required; conda ships prebuilt binaries. If a `pip` dependency
  tries to compile from source, install them with `xcode-select --install`.
- **Gatekeeper** may block the first launch of a downloaded installer. Right-click the file and choose
  *Open* rather than double-clicking it.

</details>

### Windows

Open the **Miniforge Prompt** (Start menu) and run:

```bat
conda env create --file geods.yml
conda activate geods
```

<details>
<summary><b>Windows notes</b></summary>

- **Use the Miniforge Prompt**, not plain `cmd.exe` or PowerShell. If you want conda in PowerShell, run
  `conda init powershell` once, then reopen the shell. You may also need to allow local scripts:

  ```powershell
  Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
  ```

- **Long paths.** Conda environments nest deeply and can exceed the legacy 260-character path limit.
  Install Miniforge to a short path such as `C:\miniforge3`, and enable long paths:

  ```powershell
  # Run as Administrator
  New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" `
    -Name LongPathsEnabled -Value 1 -PropertyType DWORD -Force
  ```

- **Antivirus.** Real-time scanning can make environment creation take many minutes. Excluding your
  conda folder from scanning helps considerably.
- **TensorFlow GPU.** Native Windows GPU support was dropped after TensorFlow 2.10; GPU training on
  Windows now requires WSL2. CPU TensorFlow works normally.

</details>

### Linux

```bash
conda env create --file geods.yml
conda activate geods
```

<details>
<summary><b>Linux notes</b></summary>

- No system packages are needed. Conda provides GDAL, PROJ, GEOS and their dependencies inside the
  environment — do **not** `apt install gdal-bin` as well, as a system GDAL on the library path is a
  common source of segfaults in rasterio and Fiona.
- If `conda activate` reports that your shell is not configured, run `conda init bash` (or `zsh`,
  `fish`) once and reopen the terminal.
- On headless servers, set a non-interactive Matplotlib backend:

  ```bash
  export MPLBACKEND=Agg
  ```

- `aarch64` builds exist on conda-forge for the core geospatial stack, but not for every pinned
  package here. Expect to relax some pins.

</details>

---

## Verify the installation

```bash
conda activate geods
python -c "import geopandas, rasterio, sys; print(sys.version); print('geopandas', geopandas.__version__); print('rasterio', rasterio.__version__)"
```

A fuller smoke test that exercises the GDAL/PROJ stack, not just the imports:

```python
import geodatasets
import geopandas as gpd

gdf = gpd.read_file(geodatasets.get_path("geoda.chicago_commpop"))
print(gdf.shape, gdf.crs)                                  # (77, 9) EPSG:4326
print(round(gdf.to_crs(3857).geometry.area.sum() / 1e6))   # reprojection via PROJ
```

If the download, the GEOS-backed geometry operations and the PROJ reprojection all succeed, the
geospatial stack is wired up correctly.

List your environments at any time with:

```bash
conda env list
```

---

## Using the environment

### JupyterLab

```bash
conda activate geods
jupyter lab
```

### VS Code

With the environment created, open the command palette → *Python: Select Interpreter* → choose the
`geods` environment. VS Code discovers conda environments automatically; if it doesn't, point it at
the interpreter path shown by `conda info --envs`.

### Registering the kernel elsewhere

To use `geods` as a kernel from a Jupyter install that lives in a different environment:

```bash
conda activate geods
python -m ipykernel install --user --name geods --display-name "Python (geods)"
```

---

## Updating and removing

```bash
# Apply changes made to the environment file
conda env update --file geods.yml --prune

# Delete and start over
conda env remove --name geods

# Install under a different name (e.g. to test changes side by side)
conda env create --file geods.yml --name geods-test
```

To capture the exact environment you ended up with:

```bash
conda env export --from-history > geods-lock.yml
```

`--from-history` records only what you explicitly asked for, which keeps the file portable across
platforms. Dropping the flag produces a full, exactly-reproducible but platform-specific dump.

---

## What's in the environment

| Area | Packages |
| --- | --- |
| Geospatial | `geopandas`, `rasterio`, `geodatasets` |
| Data handling | `pandas`, `numpy`, `scipy`, `openpyxl`, `xlrd`, `tabulate`, `pyyaml`, `sqlite` |
| Statistics & ML | `scikit-learn`, `statsmodels`, `tensorflow`, `keras` |
| Text & matching | `rapidfuzz`, `nltk`, `gensim`, `beautifulsoup4`, `lxml`, `markdown2` |
| Visualisation | `matplotlib`, `seaborn`, `missingno`, `pillow`, `dataframe_image` |
| Notebooks | `jupyter`, `jupyterlab`, `notebook`, `nbconvert`, `nbval`, `pytest-notebook`, `nb_black`, `nb_conda` |
| Audio | `librosa`, `noisereduce` |
| Tooling | `black`, `ipython`, `jedi`, `tqdm`, `requests`, `pip` |

---

## Known issues

The environment files pin `python=3.12` alongside package versions released for Python 3.7–3.9. No
solver can satisfy this, so `conda env create` fails during dependency resolution.

| Pin | Highest Python supported | Suggested fix |
| --- | --- | --- |
| `numpy=1.19.2` | 3.8 | unpin, or `numpy>=2.0` |
| `scipy=1.7` | 3.9 (`<3.10` is declared in its metadata) | unpin |
| `pandas=1.3` | 3.9 | unpin, or `pandas>=2.2` |
| `matplotlib=3.4` | 3.9 | unpin |
| `scikit-learn=1.0` | 3.9 | unpin |
| `statsmodels=0.12`, `seaborn=0.11.0` | 3.8 | unpin |
| `tensorflow==2.6` / `tensorflow-macos==2.7` | 3.9 | `tensorflow>=2.16`; `tensorflow-macos` is frozen at 2.16.2 and plain `tensorflow` now ships macOS arm64 wheels |
| `keras==2.6` / `==2.7` | 3.9 | drop — Keras 3 is bundled with modern TensorFlow |
| `gensim==3.8.3` | 3.8 | `gensim>=4.3` |
| `nbconvert=5.6` | 3.7 | unpin; it also conflicts with `jupyterlab=3.1` |
| `protobuf==3.20` | 3.10 | drop the pin and let TensorFlow choose |
| `librosa=0.8`, `nltk=3.6`, `pillow=8.3`, `black=20` | 3.8–3.9 | unpin |
| `nb_conda`, `nb_black` | — | long unmaintained; drop them (JupyterLab handles kernel selection natively, and `black` can be run from a terminal) |

Two further points:

1. **The `defaults` channel should be removed** from the `channels:` list. `conda-forge` alone provides
   everything here, and mixing the two produces broken or unexpectedly old builds.
2. **The audio packages** (`librosa`, `noisereduce`) and some of the NLP packages look like leftovers
   from a different course template rather than geospatial dependencies. Removing them would cut the
   solve time and the number of conflicting pins substantially.

A pragmatic starting point is to pin only Python and the packages whose API you actually depend on,
and let conda resolve the rest.

---

## Troubleshooting

<details>
<summary><b><code>conda env create</code> hangs or fails to solve</b></summary>

Modern conda (23.10 and later) uses the fast `libmamba` solver by default. On an older install, enable
it explicitly:

```bash
conda update -n base conda
conda config --set solver libmamba
```

Then set channel priority so conda-forge wins:

```bash
conda config --add channels conda-forge
conda config --set channel_priority strict
```

</details>

<details>
<summary><b><code>CondaValueError: prefix already exists</code></b></summary>

An environment named `geods` already exists. Remove it, or install under a different name:

```bash
conda env remove --name geods
# or
conda env create --file geods.yml --name geods2
```

</details>

<details>
<summary><b><code>conda activate</code> doesn't work</b></summary>

Your shell hasn't been initialised. Run `conda init <your-shell>` once and open a new terminal.
On Windows, prefer the Miniforge Prompt.

</details>

<details>
<summary><b>rasterio or Fiona import errors, or segfaults on read</b></summary>

Almost always a GDAL clash with a system-installed copy. Check what's being loaded:

```bash
python -c "import rasterio; print(rasterio.__file__); print(rasterio.gdal_version())"
```

The path should be inside your conda environment. If it isn't, remove the system GDAL from your
library path, or recreate the environment with `channel_priority strict`.

</details>

<details>
<summary><b>The environment builds but a <code>pip</code> package fails</b></summary>

Conda installs its own dependencies first, then hands the `pip:` block to pip. A failure there leaves
a partially built environment. Fix the offending pin in the YAML and rerun:

```bash
conda env update --file geods.yml --prune
```

</details>

---

## Contributing

Issues and pull requests are welcome — particularly ones that bring the pins back in line with
`python=3.12`. Please note the platform and architecture you tested on.

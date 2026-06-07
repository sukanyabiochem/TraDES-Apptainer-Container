# TraDES Apptainer Container

Containerized build of [TraDES](https://trades.blueprint.org/) (TRAjectory DESign & Ensemble Sampler) — a protein conformation sampling tool — packaged in an [Apptainer](https://apptainer.org/) (formerly Singularity) container based on Ubuntu 14.04.

## Contents

| File | Description |
|------|-------------|
| `trades.def` | Apptainer definition file used to build the container |
| `trades.sif` | Pre-built Apptainer image |
| `120612b_TraDES_Source.tar.gz` | TraDES source archive (build 120612b) |
| `glu.faa` | Example FASTA input (glutamine) |

The remaining files (`seq2trj`, `trades`, `benchtraj`, `rotlib.bin.bz2`, etc.) are TraDES binaries and data files extracted from the container for direct host use.

---

## Prerequisites

- Ubuntu 22.04 / 24.04 / 26.04 (x86_64)
- `sudo` access
- Apptainer 1.4.5 (`.deb` files included or download from [GitHub releases](https://github.com/apptainer/apptainer/releases/tag/v1.4.5))

---

## Installation

### 1. Install Apptainer

Download the `.deb` packages (already present in this directory):

```bash
wget https://github.com/apptainer/apptainer/releases/download/v1.4.5/apptainer_1.4.5_amd64.deb
wget https://github.com/apptainer/apptainer/releases/download/v1.4.5/apptainer-suid_1.4.5_amd64.deb
```

If a conflicting `apptainer-dbgsym` package is already installed, remove it first:

```bash
sudo dpkg --purge apptainer-dbgsym
```

Install Apptainer (use `--ignore-depends` to work around the `libfuse3-3` → `libfuse3-4` rename on Ubuntu 24.04+):

```bash
sudo dpkg -i --ignore-depends=libfuse3-3 apptainer_1.4.5_amd64.deb apptainer-suid_1.4.5_amd64.deb
```

### 2. Fix libfuse3 on Ubuntu 24.04 / 26.04

Ubuntu 24.04+ ships `libfuse3-4` (`libfuse3.so.4`) instead of `libfuse3-3` (`libfuse3.so.3`). Apptainer 1.4.5 requires `libfuse3.so.3`:

```bash
sudo ln -s /usr/lib/x86_64-linux-gnu/libfuse3.so.4 /usr/lib/x86_64-linux-gnu/libfuse3.so.3
sudo ldconfig
```

> This step is not needed on Ubuntu 22.04.

### 3. Get the TraDES Source

```bash
wget https://us-east.manta.joyent.com/profhogue/public/TraDES-2/120612b_TraDES_Source.tar.gz
```

### 4. Build the Container

```bash
sudo singularity build trades.sif trades.def
```

The definition file (`trades.def`) pulls Ubuntu 14.04, installs all build dependencies, compiles TraDES from source, and places the binaries at `/TraDES/` inside the image.

---

## Usage

All commands are run via `singularity exec trades.sif <command>`. The working directory on the host is automatically bind-mounted, so input/output files are read from and written to your current directory.

### Verify installation

```bash
singularity exec trades.sif benchtraj
```

### Generate a trajectory from a FASTA sequence

```bash
singularity exec trades.sif seq2trj -f glu.faa -c T -o glu
```

| Flag | Meaning |
|------|---------|
| `-f` | Input FASTA file |
| `-c T` | Use coil/loop conformation type |
| `-o` | Output prefix |

This produces `glu.trj` (and supporting files).

### Sample conformations with TraDES

```bash
singularity exec trades.sif trades -f glu.trj -p T -b 5 -a T
```

| Flag | Meaning |
|------|---------|
| `-f` | Input trajectory file |
| `-p T` | Enable phi/psi sampling |
| `-b 5` | Generate 5 conformations |
| `-a T` | All-atom output |

Output PDB files are written to the current directory (e.g., `glu_0000001.pdb` … `glu_0000005.pdb`).

### Open an interactive shell inside the container

```bash
singularity shell trades.sif
```

---

## Container Definition Summary

```
Bootstrap: docker
From: ubuntu:14.04

Dependencies installed: build-essential, autoconf, g++, flex, libmotif-dev,
  freeglut3-dev, libpng-dev, ncbi-tools-bin, libncbi6-dev, and others.

Build script: nusi/build_scripts/build_Ubuntu_x64.sh
Binaries placed at: /TraDES/ (added to PATH via %environment)
```

---

## Troubleshooting

**`libfuse3.so.3: cannot open shared object file`**
→ Run the symlink fix in step 2 above.

**`Unable to open input file rotlib.bin.bz2`**
→ Run the command from the directory containing `rotlib.bin.bz2` (i.e., this directory), or copy the file there: `singularity exec trades.sif bash -c "cp /TraDES/* $(pwd)/"`.

**`apptainer-dbgsym` version conflict during install**
→ Run `sudo dpkg --purge apptainer-dbgsym` before installing.

**`get TraDES from WAYBACK MACHINE 2016`
→ https://web.archive.org/web/20161101000000*/https://us-east.manta.joyent.com/profhogue/public/TraDES-2/120612b_TraDES_Source.tar.gz 
---


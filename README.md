# taf-bcftools

TAFFISH wrapper for [BCFtools](https://samtools.github.io/bcftools/), a
command-line toolkit for variant calling, VCF/BCF manipulation, indexing,
statistics, filtering, normalization, annotation, and related genomic variant
workflows.

This repository packages upstream BCFtools 1.23.1 and HTSlib 1.23.1 as a
TAFFISH tool app. It exposes upstream `bcftools` through a versioned TAFFISH
entry point while keeping the runtime environment inside a portable Debian 12
container image.

## Installation

Install from the public TAFFISH Hub index:

```sh
taf update
taf install bcftools
```

Install the exact release:

```sh
taf install bcftools 1.23.1-r2
```

For local testing before the app is published to the public index:

```sh
taf install --from .
```

## Usage

Show TAFFISH app help:

```sh
taf-bcftools --help
```

Show upstream BCFtools help and version:

```sh
taf-bcftools bcftools --help
taf-bcftools bcftools --version
```

Run common BCFtools workflows:

```sh
taf-bcftools bcftools view input.vcf.gz
taf-bcftools bcftools stats input.vcf.gz > input.vchk
taf-bcftools bcftools norm -f reference.fa input.vcf.gz -Oz -o normalized.vcf.gz
taf-bcftools bcftools filter -i 'QUAL>=30' input.vcf.gz -Oz -o filtered.vcf.gz
taf-bcftools bcftools index filtered.vcf.gz
```

Because this is a command-mode TAFFISH tool, the first non-option argument is
treated as a command available inside the container image. Use explicit command
mode for BCFtools itself and for companion tools:

```sh
taf-bcftools bcftools view input.vcf.gz
taf-bcftools bgzip -c input.vcf > input.vcf.gz
taf-bcftools tabix -p vcf input.vcf.gz
taf-bcftools plot-vcfstats -P -p stats-plots input.vchk
```

BCFtools subcommands such as `view`, `stats`, `norm`, and `filter` are
subcommands of the `bcftools` executable, not standalone commands in the image.
Use `taf-bcftools bcftools <SUBCOMMAND> ...` for normal BCFtools workflows.

Do not use:

```sh
taf-bcftools view input.vcf.gz
```

`view` is not a standalone executable in the container image.

When passing an upstream option through the default wrapper, use `--` so the
option is handled by BCFtools instead of the TAFFISH wrapper:

```sh
taf-bcftools -- --help
taf-bcftools -- --version
```

For most day-to-day use, explicit command mode is clearer:

```sh
taf-bcftools bcftools --help
taf-bcftools bcftools --version
```

## Package

```text
name: bcftools
command: taf-bcftools
version: 1.23.1-r2
kind: tool
image: ghcr.io/taffish/bcftools:1.23.1-r2
```

## Container

The container image is built from `docker/Dockerfile`. It compiles HTSlib
1.23.1 and BCFtools 1.23.1 from upstream release tarballs in a builder stage,
then copies the installed BCFtools runtime into a slim runtime image.

The image includes:

```text
bcftools
bgzip
tabix
htsfile
plot-vcfstats
python
perl
```

BCFtools is built with GNU Scientific Library support so GSL-dependent commands
such as `bcftools polysomy` are available. The runtime image also includes a
small Python environment with `numpy` and `matplotlib` for BCFtools plotting
helpers such as `bcftools cnv -p`, `plot-roh.py`, and `plot-vcfstats`, plus
`gffutils` for `gff2gff.py`.

`plot-vcfstats -P` is supported for generating plot outputs without a summary
PDF. The default PDF mode of `plot-vcfstats` requires `pdflatex` or `tectonic`;
that TeX engine is not bundled in this image to avoid adding a large document
toolchain to the BCFtools runtime. If a final PDF is needed, run
`plot-vcfstats -P` first and create the PDF in a downstream document/TeX
environment.

The TAFFISH metadata declares a Docker smoke check:

```text
exist: bcftools, bgzip, tabix, plot-vcfstats, python, perl
test:  bcftools --help
test:  bcftools --version 2>&1 | grep -F 'bcftools 1.23.1' >/dev/null
test:  bcftools polysomy -h 2>&1 | grep -F 'bcftools polysomy' >/dev/null
test:  bcftools cnv -h 2>&1 | grep -F 'bcftools cnv' >/dev/null
test:  bcftools plugin -l >/dev/null
test:  plot-vcfstats -h >/dev/null
test:  python -c 'import gffutils, matplotlib, numpy'
test:  bgzip --help >/dev/null
test:  tabix --help >/dev/null
```

During TAFFISH Hub indexing, this smoke metadata is used to verify that the
published image can be inspected, that the upstream executable and companion
tools are available, that GSL-enabled BCFtools features are present, and that
the packaged BCFtools command reports version 1.23.1.

## License Boundary

The TAFFISH app packaging files are licensed under Apache-2.0. The packaged upstream BCFtools software is covered by: MIT/Expat or GPL. Bundled third-party components, datasets, models, and external resources keep their own license terms.

## Upstream

- Project: BCFtools
- Homepage: <https://samtools.github.io/bcftools/>
- Source: <https://github.com/samtools/bcftools>
- Release tags: <https://github.com/samtools/bcftools/releases>
- Documentation: <https://samtools.github.io/bcftools/bcftools.html>
- Upstream license: MIT/Expat; this packaged binary is distributed as GPL-3.0
  because it links against GNU Scientific Library to enable GSL-dependent
  functionality.

## Maintainer Notes

Useful checks before publishing:

```sh
taf check
taf compile -- --help
taf compile -- bcftools --version
taf publish --release --dry-run
docker build --check -f docker/Dockerfile .
docker build -t ghcr.io/taffish/bcftools:1.23.1-r2 -f docker/Dockerfile .
docker run --rm ghcr.io/taffish/bcftools:1.23.1-r2 bcftools --version
docker run --rm ghcr.io/taffish/bcftools:1.23.1-r2 bcftools plugin -l
```

The repository wrapper files are licensed under Apache-2.0. The packaged
BCFtools binary and third-party runtime components are distributed under their
own upstream licenses.

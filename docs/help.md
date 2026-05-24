taf-bcftools 1.23.1-r2

TAFFISH wrapper for BCFtools, a command-line toolkit for variant calling,
VCF/BCF manipulation, filtering, normalization, annotation, indexing, and
statistics.

Usage:
  taf-bcftools [TAF-APP-OPTION]
  taf-bcftools bcftools [BCFTOOLS-COMMAND] [COMMAND-ARGS...]
  taf-bcftools bgzip [BGZIP-ARGS...]
  taf-bcftools tabix [TABIX-ARGS...]
  taf-bcftools plot-vcfstats [PLOT-ARGS...]
  taf-bcftools -- [BCFTOOLS-OPTION...]
  taf-bcftools <COMMAND> [COMMAND-ARGS...]

TAF app options:
  -h, --help       Show this help text
  -v, --version    Show package and command version
  --compile        Print generated shell code instead of running it
  --               Stop parsing TAFFISH wrapper options

Upstream help:
  taf-bcftools bcftools --help
  taf-bcftools bcftools --version
  taf-bcftools bcftools plugin -l

Recommended BCFtools examples:
  taf-bcftools bcftools view input.vcf.gz
  taf-bcftools bcftools stats input.vcf.gz > input.vchk
  taf-bcftools bcftools norm -f reference.fa input.vcf.gz -Oz -o normalized.vcf.gz
  taf-bcftools bcftools filter -i 'QUAL>=30' input.vcf.gz -Oz -o filtered.vcf.gz
  taf-bcftools bcftools index filtered.vcf.gz
  taf-bcftools bcftools polysomy sample.vcf.gz

Companion command examples:
  taf-bcftools bgzip -c input.vcf > input.vcf.gz
  taf-bcftools tabix -p vcf input.vcf.gz
  taf-bcftools htsfile input.vcf.gz
  taf-bcftools plot-vcfstats -P -p stats-plots input.vchk

Common BCFtools subcommands:
  annotate
  call
  concat
  consensus
  convert
  csq
  filter
  index
  isec
  merge
  mpileup
  norm
  plugin
  query
  roh
  sort
  stats
  view

Notes:
  - This command runs BCFtools inside the TAFFISH container image.
  - Use "taf-bcftools bcftools <SUBCOMMAND> ..." for ordinary BCFtools
    workflows.
  - Use explicit command mode for companion tools available in the image, such
    as "bcftools", "bgzip", "tabix", "htsfile", and "plot-vcfstats".
  - BCFtools subcommands such as "view", "stats", "norm", and "filter" are not
    standalone executables in the container image. Do not use
    "taf-bcftools view ..."; use "taf-bcftools bcftools view ...".
  - Use "--" before upstream options when an option may be handled by the
    TAFFISH wrapper itself, such as "--help" or "--version".
  - The image includes GSL support for "bcftools polysomy".
  - The image includes Python, gffutils, numpy, matplotlib, and Perl for
    BCFtools helper scripts and plotting helpers.
  - "plot-vcfstats -P" works without a TeX engine. The default PDF mode of
    "plot-vcfstats" requires "pdflatex" or "tectonic", which is not bundled in
    this image.
  - Input and output paths should be accessible from the current working
    directory or from mounted user paths.
  - This image packages upstream BCFtools 1.23.1 with HTSlib 1.23.1 in a
    Debian 12 runtime image.

Container:
  image: ghcr.io/taffish/bcftools:1.23.1-r2
  supported backends: apptainer, podman, docker

License:
  TAFFISH app packaging: Apache-2.0.
  Upstream software: MIT/Expat or GPL.
  Bundled components, data, models, and external resources keep their
  own license terms.

Upstream:
  project: BCFtools
  homepage: https://samtools.github.io/bcftools/
  source:  https://github.com/samtools/bcftools

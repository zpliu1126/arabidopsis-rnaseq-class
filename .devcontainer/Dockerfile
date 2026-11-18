FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

USER root
RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        cutadapt=4.4-1build2 \
        fastqc=0.12.1+dfsg-3 \
        hisat2=2.2.1-4 \
        python3-venv \
        r-base-core=4.3.3-2build2 \
        r-base-dev=4.3.3-2build2 \
        r-bioc-deseq2=1.42.0+dfsg-1 \
        r-cran-biocmanager=1.30.22+dfsg-2 \
        r-cran-gdtools=0.3.5-1 \
        r-cran-ggplot2=3.4.4+dfsg-1 \
        r-cran-ggraph=2.1.0+dfsg-1 \
        r-cran-irkernel=1.3.2-2 \
        samtools=1.19.2-1build2 \
        subread=2.0.6+dfsg-2 \
        curl fonts-dejavu-core python3 util-linux \
        libcairo2-dev libcurl4-openssl-dev libssl-dev libxml2-dev \
    && rm -rf /var/lib/apt/lists/* \
    && dpkg --compare-versions "$(cutadapt --version)" ge 4.4 \
    && Rscript -e 'stopifnot(requireNamespace("DESeq2", quietly = TRUE))'

# Bioconductor 3.18 matches R 4.3. The CRAN snapshot matches ggplot2 3.4.
# Use C++14 for packages such as fgsea that request C++11 with newer Boost headers.
RUN printf '%s\n' 'CXX11STD = -std=gnu++14' > /tmp/rnaseq-Makevars \
    && R_MAKEVARS_USER=/tmp/rnaseq-Makevars Rscript --vanilla \
        -e 'options(repos = c(CRAN = "https://packagemanager.posit.co/cran/2024-02-29"), timeout = 600)' \
        -e 'options(BioC_mirror = "https://bioconductor.uib.no")' \
        -e 'BiocManager::install("clusterProfiler", version = "3.18", ask = FALSE, update = FALSE, Ncpus = 2L)' \
        -e 'stopifnot(packageVersion("clusterProfiler") == "4.10.1", packageVersion("ggplot2") == "3.4.4")' \
        -e 'stopifnot(requireNamespace("clusterProfiler", quietly = TRUE), requireNamespace("DESeq2", quietly = TRUE))' \
    && rm /tmp/rnaseq-Makevars

ENV VIRTUAL_ENV=/opt/rnaseq-python
ENV PATH="${VIRTUAL_ENV}/bin:${PATH}"

RUN /usr/bin/python3 -m venv "$VIRTUAL_ENV" \
    && python -m pip install --no-cache-dir 'jupyterlab==4.6.6' 'ipykernel==7.4.0' \
    && python -m ipykernel install --sys-prefix --name python3 --display-name 'Python 3 (RNA-seq)' \
    && Rscript --vanilla -e 'stopifnot(IRkernel::installspec(user = FALSE, name = "ir", displayname = "R 4.3.3 (RNA-seq)") == 0L)'

USER vscode
RUN python -m pip check \
    && python -c 'from jupyter_client.kernelspec import KernelSpecManager; assert {"python3", "ir"} <= KernelSpecManager().find_kernel_specs().keys()'

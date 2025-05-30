FROM rocker/rstudio:4.4.0

# Ensure virtual env on PATH so we're using the right venv binaries
#

# Set default locale
ENV LC_ALL=C

# Ensure virtual env is on the PATH so we're using the right venv binaries
ENV VIRTUAL_ENV=/opt/container-venv
ENV PATH=/opt/container-venv/bin:$PATH

# Ensure virtual env is on the PATH for LSF TO load the Package with a Specific Library Path
# This fixes the issue with the cc libraries
ENV R_LIBS_USER=/home/user/R/x86_64-pc-linux-gnu-library/4.4

#
# Ensure core system software is up to date and ready
#

# Install necessary certificates package
#RUN apt-get -y update && apt-get -y install ca-certificates
RUN apt-get -y update && apt-get -y install gdebi-core python3-pip python3-venv
# If the certificate works, better to use the following code of line (instead of breaking in two parts)
#RUN apt-get -y update && apt-get -y install ca-certificates gdebi-core python3-pip python3-venv

#
# Install Cloudflare certificates so Docker build isn't interrupted by
# certificate errors. Combination of certificates should be valid until 2029.
#

#COPY "./cloudflare_certs/original-certificate.pem" /usr/local/share/ca-certificates/2024-certificate.crt
#COPY "./cloudflare_certs/SJ-2025-2029-CF-WARP-certificate.pem" /usr/local/share/ca-certificates/2025-2029-certificate.crt
#RUN update-ca-certificates


#
# Set up virtual env; we need this to keep the existing numpy/scipy/pandas
# versions cohesive; otherwise, we get mistmatches with e.g. multiqc in the
# analysis pipelines. After activating the virtual env, we should have:
#
# (container-venv) Apptainer> pip3 list
# Package         Version
# --------------- -----------
# numpy           2.2.1
# pandas          2.2.3
# scipy           1.14.1
#
# Note: need to "source" the virtual env activate script, not just run it.
#

RUN python3 -m venv /opt/container-venv
RUN . /opt/container-venv/bin/activate


# Clean the APT Cache: Sometimes, the package cache may cause problems during installations. 
# You can clear the cache to ensure there are no broken cached files:
# RUN apt-get clean

# Prevent errors in dependencies for the following
#RUN apt-get update
#

# Avoids BiocManager errors in Rhtslib; "fatal error: lzma.h: No such file or directory" etc
RUN apt-get -y install liblzma-dev libbz2-dev

# Avoids BiocManager errors in bluster; "libglpk.so.40: cannot open shared object file: No such file or directory"
RUN apt-get -y install libglpk-dev

# Avoids BiocManager errors in infercnv: "libjags.so.4: cannot open shared object file: No such file or directory"
RUN apt-get update && \
    apt-get install -y r-cran-rjags

# Avoids errors in "Rscript -e 'devtools::install_github("welch-lab/RcppPlanc")'"
RUN apt-get -y install cmake libhdf5-dev

# Needed for pipeline stages
RUN apt-get update && \
    apt-get install -y openjdk-8-jre-headless fastqc curl

# Install MultiQC specific version (e.g., 1.25)
# Current multiqc in the container is v1.12 whereas previously the contained had v1.25. 
# The report style from v1.25 is preferable as it contains more information on overrepresented sequences.
RUN pip install multiqc==1.25

# Needed for the cluster-cell-calling stage etc. Pandas as per Antonia's request.
RUN pip3 install pandas leidenalg


#
#
# Install texlive full version
# This step can break easily. The code is correct, user needs to keep bulding until it passes this step (very time consuming).
# I do not know why it breaks, but there must be some issue from Cloudflare that interferes with permissions and proper installation.
#
ENV TEXLIVE_INSTALL_NO_CONTEXT_CACHE=1 \
    NOPERLDOC=1

# Install TexLive prerequisites
RUN apt-get update && \
    apt-get install -y wget unzip tar \
    make fontconfig perl openjdk-8-jre libgetopt-long-descriptive-perl \
    libdigest-perl-md5-perl libncurses5 \
    python3-pygments && \
    rm -rf /var/lib/apt/lists/*

    
# Install TexLive to the container
RUN wget http://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz -v && \
    tar xzf install-tl-unx.tar.gz -v && \
    #rm install-tl-unx.tar.gz && \
    cd install-tl* && \
    echo "selected_scheme scheme-full" > install.profile && \
    echo "tlpdbopt_install_docfiles 0" >> install.profile && \
    echo "tlpdbopt_install_srcfiles 0" >> install.profile && \
    echo "tlpdbopt_autobackup 0" >> install.profile && \
    echo "tlpdbopt_sys_bin /usr/bin" >> install.profile && \
    ./install-tl -profile install.profile && \
    cd .. && rm -rf install-tl*

# Symlink TexLive binaries to PATH
RUN /usr/local/texlive/2024/bin/x86_64-linux/tlmgr path add
#


#
# Use latest R sources to avoid e.g. "matrixStats" version too low for "MatrixGenerics"
#

RUN echo "options(repos = c(CRAN = 'https://packagemanager.posit.co/cran/__linux__/jammy/latest'))" >> /usr/local/lib/R/etc/Rprofile.site

#
# Install required R package managers
#
RUN Rscript -e 'install.packages("usethis")'
RUN Rscript -e 'install.packages(c("BiocManager", "devtools", "remotes"))'

# Debugging step: Verify installations
RUN Rscript -e 'if (!requireNamespace("BiocManager", quietly = TRUE)) stop("BiocManager not installed!")'
RUN Rscript -e 'if (!requireNamespace("devtools", quietly = TRUE)) stop("devtools not installed!")'
RUN Rscript -e 'if (!requireNamespace("remotes", quietly = TRUE)) stop("remotes not installed!")'

RUN Rscript -e 'BiocManager::install("celldex")'
RUN Rscript -e 'if (!requireNamespace("celldex", quietly = TRUE)) stop("celldex not installed!")'


# ###############################################################
# Problematic installation for the following - February 5, 2025 #
# START ########## ########## ########## ########## ########## ##
# Let's fix it!
#
# # Docker: Install infercnv
# https://github.com/broadinstitute/infercnv/blob/master/docker/Dockerfile
RUN apt-get update && apt-get -y install curl libssl-dev libcurl4-openssl-dev \
                                        libxml2-dev git python3 jags \
                                        r-cran-rjags time && \
                      apt-get clean && rm -rf /var/tmp/* \
                                          /tmp/* /var/lib/apt/lists/*
                                          
RUN Rscript -e 'BiocManager::install("infercnv")'
# Checkout and install infercnv
# update to version bump commit
RUN git clone https://github.com/broadinstitute/infercnv && cd infercnv && \
      git checkout master && git checkout 1b46b48303bac4a882bcb758e78fcf7f832fdefb && \
      R CMD INSTALL . 

ENV PATH=${PATH}:/infercnv/scripts
RUN Rscript -e 'if (!requireNamespace("infercnv", quietly = TRUE)) stop("infercnv not installed!")'
#

#
# # Docker: Install scDblFinder
# https://github.com/plger/scDblFinder/blob/devel/Dockerfile
# RUN Rscript -e "BiocManager::install('ensembldb'); BiocManager::install('Rtsne')"
# RUN Rscript -e "devtools::install('.', dependencies=TRUE, repos = BiocManager::repositories(), build_vignettes = TRUE)"
#RUN apt-get update && apt-get install -y \
#    libcurl4-openssl-dev \
#    libssl-dev \
#    libxml2-dev \
#    build-essential \
#    libgit2-dev \
#    && rm -rf /var/lib/apt/lists/*
    
#RUN Rscript -e 'BiocManager::install("scDblFinder")'
#RUN Rscript -e 'if (!requireNamespace("scDblFinder", quietly = TRUE)) stop("scDblFinder not installed!")'

# Or, to get the very latest version,
# RUN Rscript -e 'BiocManager::install("plger/scDblFinder")'
#

#
# # Docker: Install numbat
# https://github.com/kharchenkolab/numbat/blob/main/docker/Dockerfile
RUN apt-get update --yes && apt-get install --yes build-essential \
  libcurl4-gnutls-dev libxml2-dev libssl-dev libbz2-dev zlib1g-dev \
  libfontconfig1-dev libharfbuzz-dev libfribidi-dev \
  libncurses5-dev libncursesw5-dev liblzma-dev libgit2-dev \
  libfreetype6-dev libpng-dev libtiff5-dev libjpeg-dev \
  libglpk-dev git autoconf gettext libtool automake  \
  samtools sudo

RUN cd /usr/bin && \
  wget https://github.com/samtools/htslib/releases/download/1.15.1/htslib-1.15.1.tar.bz2 && \
  tar -vxjf htslib-1.15.1.tar.bz2 && cd htslib-1.15.1 && make && sudo make install

RUN Rscript -e 'install.packages(c("ragg", "pkgdown"))'

#RUN R -e 'chooseCRANmirror(ind=42); install.packages("ragg")'
#RUN R -e 'chooseCRANmirror(ind=42); install.packages("pkgdown")'

RUN R -e 'devtools::install_github("YuLab-SMU/ggtree", dependencies=TRUE)'

RUN R -e 'devtools::install_github("kharchenkolab/numbat", dependencies=TRUE)'

RUN git clone https://github.com/kharchenkolab/numbat.git

RUN mkdir -p /tmp && chmod 777 /tmp

RUN chmod 777 /numbat/inst/bin/pileup_and_phase.R 

RUN mkdir data

# hg 38
# RUN cd /data && wget -q https://sourceforge.net/projects/cellsnp/files/SNPlist/genome1K.phase3.SNP_AF5e2.chr1toX.hg38.vcf.gz && gzip -d genome1K.phase3.SNP_AF5e2.chr1toX.hg38.vcf.gz 

## Note: not run; the user can follow these commands if they want hg19
# hg19
## RUN cd /data && wget https://sourceforge.net/projects/cellsnp/files/SNPlist/genome1K.phase3.SNP_AF5e2.chr1toX.hg19.vcf.gz && gzip -d genome1K.phase3.SNP_AF5e2.chr1toX.hg19.vcf.gz

# hg38
# RUN cd /data && wget -q http://pklab.med.harvard.edu/teng/data/1000G_hg38.zip && unzip 1000G_hg38.zip 

# hg19
##RUN cd /data && wget http://pklab.med.harvard.edu/teng/data/1000G_hg19.zip && unzip 1000G_hg19.zip

RUN apt-get -y update

RUN git clone https://github.com/single-cell-genetics/cellsnp-lite.git && cd cellsnp-lite && \
  autoreconf -iv && ./configure && make && sudo make install    

#RUN apt-get -y update

RUN wget -q https://storage.googleapis.com/broad-alkesgroup-public/Eagle/downloads/Eagle_v2.4.1.tar.gz && cd .. && tar -xvzf Eagle_v2.4.1.tar.gz && cd /Eagle_v2.4.1 && cp eagle /usr/bin

# Debugging
RUN Rscript -e 'if (!requireNamespace("numbat", quietly = TRUE)) stop("numbat not installed!")'
#
# Problematic installation for R packages fixed - February 5, 2025 #
# THE END ########## ########## ########## ########## ########## ###
# ##################################################################

#
# This is a fairly time consuming section; lots built from source.
#

# This code solve the failing with installation
RUN apt-get -y update
RUN apt-get install -y libcurl4-openssl-dev libssl-dev libxml2-dev

RUN Rscript -e 'BiocManager::install("scDblFinder", ask = FALSE)'
RUN Rscript -e 'if (!requireNamespace("scDblFinder", quietly = TRUE)) stop("scDblFinder not installed!")'

RUN Rscript -e 'BiocManager::install("miQC")'
RUN Rscript -e 'if (!requireNamespace("miQC", quietly = TRUE)) stop("miQC not installed!")'

RUN Rscript -e 'BiocManager::install("scater")'
RUN Rscript -e 'if (!requireNamespace("scater", quietly = TRUE)) stop("scater not installed!")'

RUN Rscript -e 'BiocManager::install("SingleCellExperiment")'
RUN Rscript -e 'if (!requireNamespace("SingleCellExperiment", quietly = TRUE)) stop("SingleCellExperiment not installed!")'

RUN Rscript -e 'BiocManager::install("SingleR")'
RUN Rscript -e 'if (!requireNamespace("SingleR", quietly = TRUE)) stop("SingleR not installed!")'


#
# Some github-based R modules; RcppPlanc needs previously installed cmake
# and libhdf5-dev
#

RUN Rscript -e 'devtools::install_github("igordot/scooter")'
RUN Rscript -e 'if (!requireNamespace("scooter", quietly = TRUE)) stop("scooter not installed!")'

RUN Rscript -e 'devtools::install_github("welch-lab/RcppPlanc")'
RUN Rscript -e 'if (!requireNamespace("RcppPlanc", quietly = TRUE)) stop("RcppPlanc not installed!")'

# New R libraries for R shiny app
RUN Rscript -e 'devtools::install_github("SGDDNB/ShinyCell")'
RUN Rscript -e 'if (!requireNamespace("ShinyCell", quietly = TRUE)) stop("ShinyCell not installed!")'


#
# Removed "grid" from installs as triggered the warning message:
# "package 'grid' is a base package, and should not be updated"
# "grid" was then subsequently ignored by the installer anyway.
#

RUN Rscript -e 'install.packages(c("clustree", "cowplot", "data.table"))'
RUN Rscript -e 'install.packages(c("flexmix", "flextable", "forcats"))'
RUN Rscript -e 'install.packages(c("fs", "future", "GGally"))'
RUN Rscript -e 'install.packages(c("ggh4x", "ggplot2", "ggpmisc"))'
RUN Rscript -e 'install.packages(c("ggrepel", "ggthemes"))'
RUN Rscript -e 'install.packages(c("harmony", "igraph", "irlba"))'
RUN Rscript -e 'install.packages(c("knitr", "optparse"))'
RUN Rscript -e 'install.packages(c("patchwork", "purrr", "RColorBrewer"))'
RUN Rscript -e 'install.packages(c("remotes", "reshape2", "rliger"))'
RUN Rscript -e 'install.packages(c("rlist"))'
RUN Rscript -e 'install.packages(c("R.utils"))'
RUN Rscript -e 'install.packages(c("SeuratObject"))'
RUN Rscript -e 'install.packages(c("shiny", "SoupX", "stringr"))'
RUN Rscript -e 'install.packages(c("tidytext", "tidyverse", "tinytex", "yaml"))'

# New R libraries for R shiny app
RUN Rscript -e 'install.packages(c("shinyhelper", "DT", "ggdendro"))'

#

# Prevent errors in dependencies for the following
RUN apt-get -y update

#
# Was Rscript -e 'remotes::install_version("Seurat", "4.4.0", repos = c("https://packagemanager.posit.co/cran/__linux__/jammy/latest"))'
# but we specified the latest CRAN repo earlier so repo URL is redundant.
# Default version of Seurat is higher than 4.4.0 and is incompatible with
# input object format so install specific version. We also need to fix the
# SeuratWrappers, as the existing version is incompatible with 4.4.0:
#
# https://github.com/satijalab/seurat-wrappers/issues/169
#
# We need to install the Seurat wrappers first, as otherwise they will
# override the install of Seurat 4.4.0 with a later version of Seurat.
#
RUN Rscript -e 'remotes::install_github("satijalab/seurat-wrappers@community-vignette")'
RUN Rscript -e 'remotes::install_version("Seurat", version = "4.4.0")'

#RUN Rscript -e 'install.packages("Signac")'
RUN Rscript -e 'devtools::install_github("stuart-lab/signac", ref = "develop")'


# This code solve the failing with installation
RUN Rscript -e 'devtools::install_github("welch-lab/liger")'
#RUN Rscript -e 'if (!requireNamespace("liger", quietly = TRUE)) stop("liger not installed!")'


#
# Install CellRanger v8.0.1
#
# You will need to download the desired version from here: https://www.10xgenomics.com/support/software/cell-ranger/downloads/previous-versions
# Save on the same folder where Dockerfile is located
# You should also replace the link with the one from the most recent download as the url expires after 1-2 days
# Set environment variables
ENV PATH=/opt/cellranger-8.0.1/bin:${PATH}
ENV LD_LIBRARY_PATH=/opt/cellranger-8.0.1/lib

RUN curl -o cellranger-8.0.1.tar.gz "https://cf.10xgenomics.com/releases/cell-exp/cellranger-8.0.1.tar.gz?Expires=1738835653&Key-Pair-Id=APKAI7S6A5RYOXBWRPDA&Signature=HtHQnp-gnFU0b5lZlGj8uzZN29pVn5ij7Zoaf1YA6260lqfCqwfaWBH7dU0O8Zsi-C-gY~UPyYGRztoymb5fqR6h0o-FIyYG0KcX5WVAox2cPe1u0k4ZJlq4MrpUmTMbJQ7vpF3abinkB8~y~IzkinoKTuu5Ibvq6ZAZN1MkGBpRZ8hyu-y0dy2KyRru9e3vFWpzRkrEGeUMcx89OMBnM~-MBPRWtc7xrLrhDMAhD9WS4-YGPAQce3qsaq-8OXGnprhpG4hNbnkSzSKcj97p64HmSqW5pddObza7T4bRzLx~e3zYqrLv6bw-YZv~-cRZxgWr2d~fUYWKjEkcikQ32w__"
RUN tar -xvzf cellranger-8.0.1.tar.gz -C /opt
#

# Optional: Clean up to reduce image size
RUN rm -rf /var/lib/apt/lists/* && apt-get clean

#
# Set the working directory (optional)
# WORKDIR /home/user
#

# Run virtual end on container startup- adjust as needed.
ENTRYPOINT . /opt/container-venv/bin/activate


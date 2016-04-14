Error Pattern Analyzer User Guide
=================================

## Introduction

The error pattern analyzer is used to evaluate various models of spurious basecall and indel errors in sequencing data. This is an internal Strelka development tool and is not well supported for external users. This module is part of the Strelka code distribution because it (1) reuses many of Strelka's sequence handling libraries (2) potentially could be used to make error model decisions for the variant caller. Note that any usage of the pattern analyzer that directly impacts the variant calling model will be documented as part of the small variant caller itself.

## Installation

This module builds and installs with the Strelka variant caller by default. No special build steps are required.

## Method Overview

The error pattern analyzer is comprised of two primary steps.

(1) An error counting workflow analyzes BAMs to produce per-locus allele distributions over the genome for various allele types and context segmentations. Error counts are found for segments of the genome in parallel and merged to produce a single counts file for the sample.

(2) Given a counts file from the first step, various error models can be run on the data to evaluate model fit or to paramaterize a model to specific sequencing conditions.

More detailed methods documentation for both steps can be found [here](../methods/errorAnalysis/)

## Input requirements

The error counting process input requirements are identical to the Strelka small variant caller.

## Outputs

### Counts files

The primary output of the error counting workflow is a binary error counts file, currently written
to `${COUNTS_ANALYSIS_PATH}/results/variants/strelkaErrorCounts.bin`. This is a binary format, without running a model the easiest way to observe file details is to dump a summary of the file contents as follows:

    ${STRELKA_INSTALL_PATH}/libexec//DumpSequenceErrorCounts --counts-file ${COUNTS_ANALYSIS_PATH}/results/variants/strelkaErrorCounts.bin

### Error model output:

All error models applied to the counts file currently write to stdout, typically in csv format.

## Error counting workflow configuration and execution

Error counting is run in a two step procedure: (1) configuration and (2) workflow
execution. The configuration step is used to specify the input data and any
options pertaining to the error counting methods. The execution
step is used to specify any parameters pertaining to _how_ the workflow is executed
(such as the total number of cores or SGE nodes over which the jobs should be
parallelized). The second execution step can also be interrupted and restarted
without changing the final result of the workflow.

### Configuration

The workflow is configured with the script: `${STRELKA_INSTALL_PATH}/libexec/configureSequenceErrorCountsWorkflow.py`
. Running this script with no arguments will display all standard configuration
options to specify input alignment files, the reference sequence and the output run folder.
Note that all input alignment and reference sequence files must contain the same chromosome names
in the same order. Note that many downstream error modeling routines assume the counts are gathered from diploid regions, for now non-autosomes need to be filtered out crudely by explicitily listing all autosomes as region targets.

Example configuration for human autosomes:

    ${STRELKA_INSTALL_PATH}/libexec/configureSequenceErrorCountsWorkflow.py \
    --bam=sample.bam \
    --referenceFasta=hg19.fa \
    --runDir ${COUNTS_ANALYSIS_PATH}
    --region chr1 \
    --region chr2 \
    --region chr3 \
    --region chr4 \
    --region chr5 \
    --region chr6 \
    --region chr7 \
    --region chr8 \
    --region chr9 \
    --region chr10 \
    --region chr11 \
    --region chr12 \
    --region chr13 \
    --region chr14 \
    --region chr15 \
    --region chr16 \
    --region chr17 \
    --region chr18 \
    --region chr19 \
    --region chr20 \
    --region chr21 \
    --region chr22
        

On completion, the configuration script will create the workflow run script `${COUNTS_ANALYSIS_PATH}/runWorkflow.py`. This can be used to run the workflow in various parallel compute modes per the
instructions in the [Execution] section below.

### Execution

The configuration step creates a new workflow run script in the requested run directory:

`${COUNTS_ANALYSIS_PATH}/runWorkflow.py`

This script is used to control parallel execution of the workflow via the [pyFlow][2]
task engine. It can be used to parallelize analysis via one
of two modes:

1. Parallelized across multiple cores on a single node.
2. Parallelized across multiple nodes on an SGE cluster.

A running workflow can be interrupted at any time and resumed where it left
off. If desired, the resumed analysis can use a different running mode or total
core count.

For a full list of execution options, see:

`${COUNTS_ANALYSIS_PATH}/runWorkflow.py -h`

Example execution on a single node:

`${COUNTS_ANALYSIS_PATH}/runWorkflow.py -m local -j 8`

Example execution on an SGE cluster:

`${COUNTS_ANALYSIS_PATH}/runWorkflow.py -m sge -j 36`

#### Advanced execution options

These options are useful for workflow development and debugging:

* Stderr logging can be disabled with `--quiet` argument. Note this log is
  replicated to `${COUNTS_ANALYSIS_PATH}/workspace/pyflow.data/logs/pyflow_log.txt`
  so there is no loss of log information.
  
## Error Model evaluation/Parameter estimation 

Given a counts file, error estimation under a particular model can be fun via:

        ${STRELKA_INSTALL_PATH}/libexec/EstimateParametersFromErrorCounts --counts-file myCounts.bin --model 2
        
Note that the estimation process currenlty offers very little runtime flexibility or documentation, any user of this tool is assumed to be adding or modifying models by changes the estimator source code itself. As a minimal convenience to developers, the exact model choosen for analysis can be choosen at runtime by number, per the above aexample.


[2]: http://Illumina.github.io/pyflow/
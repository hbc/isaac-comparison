## Isaac and bwa-GATK comparison

Comparison between two calling approaches:

- Illumina's Isaac [aligner](https://github.com/sequencing/isaac_aligner) and
  [variant caller](https://github.com/sequencing/isaac_variant_caller)
- [bwa-mem](https://github.com/lh3/bwa) and [GATK HaplotypeCaller](https://www.broadinstitute.org/gatk/guide/best-practices)

### Running

We use [bcbio's AWS support](https://bcbio-nextgen.readthedocs.org/en/latest/contents/cloud.html)
to launch and configure instances.

Running the GATK comparison:

    mkdir -p ~/run/hiseqxten/work
    cd ~/run/hiseqxten/work
    bcbio_vm.py run s3://bcbio/hiseqxten-gatk/hiseqxten-gatk.yaml -n 32

See [isaac_run.md](https://github.com/hbc/isaac-comparison/blob/master/isaac_run.md)
for command lines for running Isaac.

### Results

- [Validation and resource usage plots](http://imgur.com/a/eUnOS)

## Isaac and bwa-GATK comparison

Compared two calling approaches:

- Illumina's Isaac [aligner](https://github.com/sequencing/isaac_aligner)
  (01.14.11.27) and
  [variant caller](https://github.com/sequencing/isaac_variant_caller) (1.0.6)
- [bwa-mem](https://github.com/lh3/bwa) (0.7.10) and
  [GATK HaplotypeCaller](https://www.broadinstitute.org/gatk/guide/best-practices) (3.3-0)

Uses [HiSeq X Ten data from the Garvan Institute, DNAnexus and AllSeq](http://allseq.com/x-ten-test-data)
with 800 million 150bp paired reads (40x coverage).

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

Run on a single [Amazon EC2 r3.8xlarge instance](http://aws.amazon.com/ec2/instance-types/) with
32 cores and 244Gb of memory, using an
[EBS provisioned SSD volume with 3000 IOPS](https://aws.amazon.com/ebs/details/#PIOPS).

|                         | Isaac | bwa + GATK |
|-------------------------+-------+------------|
| alignment               |  4:33 |       8:49 |
| callable regions        |       |       0:45 |
| variant calling         |  0:45 |       3:08 |
| variant post-processing |       |       0:46 |
|-------------------------+-------+------------|
| total                   |  5:28 |      13:28 |

- [Validation and resource usage plots](http://imgur.com/a/eUnOS)

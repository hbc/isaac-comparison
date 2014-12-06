Resources
---------

https://github.com/sequencing/isaac_aligner/blob/master/src/markdown/manual.md
https://github.com/sequencing/isaac_variant_caller
http://seqanswers.com/forums/archive/index.php/t-30915.html

Install iSAAC on a new machine
------------------------------

    sudo apt-get install git fabric ruby build-essential xsltproc
    cd install
    git clone https://github.com/chapmanb/cloudbiolinux.git
    cd cloudbiolinux
    fab -H localhost install_brew:htslib
    fab -H localhost install_brew:isaac-aligner
    fab -H localhost install_brew:isaac-variant-caller
    fab -H localhost install_brew:gvcftools

Build reference genome
----------------------
(7 hours -- took from [2014-12-05 02:58:00] [2014-12-05 09:44:25])


    mkdir -p ~/data
    cd ~/data
    gof3r get --no-md5 -b biodata -k prepped/GRCh37/GRCh37-seq.tar.gz | pigz -d -c | tar -xvp
    cd GRCh37
    isaac-sort-reference -j 32 -g seq/GRCh37.fa -o isaac
    isaac-pack-reference -j 32 -r isaac/sorted-reference.xml -o GRCh37-isaac.tar.gz
    gof3r put --no-md5 -b biodata -p GRCh37-isaac.tar.gz -k prepped/GRCh37/GRCh37-isaac.tar.gz -m x-amz-storage-class:REDUCED_REDUNDANCY -m x-amz-acl:public-read

Install reference genome
------------------------

    mkdir -p ~/data
    cd ~/data
    gof3r get --no-md5 -b biodata -k prepped/GRCh37/GRCh37-seq.tar.gz | pigz -d -c | tar -xvp
    cd GRCh37
    gof3r get --no-md5 -b biodata -k prepped/GRCh37/GRCh37-isaac.tar.gz
    mkdir isaac && cd isaac
    isaac-unpack-reference -j 32 -i ../GRCh37-isaac.tar.gz

Run alignment and calling
-------------------------

    mkdir -p ~/run/isaac-hiseq
    cd !$
    mkdir fastq
    gof3r get --no-md5 -b dnanexus-rnd -k NA12878-xten/reads/NA12878D_HiSeqX_R1.fastq.gz -p fastq/lane1_read1.fastq.gz
    gof3r get --no-md5 -b dnanexus-rnd -k NA12878-xten/reads/NA12878D_HiSeqX_R2.fastq.gz -p fastq/lane1_read2.fastq.gz

    echo "[`date`] alignment start" >> run.log
    isaac-align -j 32 -m 150 -r ~/data/GRCh37/isaac/sorted-reference.xml -n GRCh37 \
      -b fastq --base-calls-format fastq-gz --keep-unaligned back -o Aligned
    echo "[`date`] alignment finished" >> run.log
    cp /usr/local/etc/ivc_config_default.ini config.ini
    echo "[`date`] variant call start" >> run.log
    configureWorkflow.pl --bam Aligned/lane1.bam --ref ~/data/GRCh37/seq/GRCh37.fa --config=config.ini --output-dir=./Variants
    make -j 32
    echo "[`date`] variant call finished" >> run.log
    gzip -dc Variants/lane1.vcf.gz | extract_variants | awk '/^#/ || $7 == "PASS"' | bgzip -c > NA12878-isaac.vcf.gz

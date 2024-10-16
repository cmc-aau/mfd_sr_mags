## Command examples for short-read MAG recovery

### Assembly
* Shallow, short-read metagenomes were assembled with [Megahit](https://github.com/voutcn/megahit)
* Using `MFDxxxx` as an example, the command would be:
```
megahit -t 10 -1 reads/MFDxxxx_R1.fastq.gz -2 reads/MFDxxxx_R2.fastq.gz -o contigs/MFDxxxx --k-list 27,43,71,99,127 -m 0.7 --min-contig-len 1000
```

### Multi-sample coverage profiles
* To perform [multi-sample](https://www.nature.com/articles/s41592-023-01934-8) metagenomic binning of paired-end short reads listed in `MFD_forward.txt` and `MFD_reverse.txt`, overlapping samples were determined using [binchicken](https://github.com/AroneyS/binchicken)
* Sample overlaps are based on [SingleM profiling](https://github.com/wwood/singlem) in `SingleM_outputs`, about which more information is available in this GitHub [repo](https://github.com/cmc-aau/mfd_novelty_section)
* Using `MFDxxxx` as the sample to be binned and `MFDyyyy` as an additional sample with an overlapping microbial community:
```
binchicken coassemble --forward-list MFD_forward.txt --reverse-list MFD_reverse.txt --sample-singlem-dir SingleM_outputs --max-coassembly-samples 1 --max-recovery-samples 10 --singlem-metapackage 2.1.GTDB_r214.metapackage_20231006.smpkg.zb

minimap2 -t 10 -ax sr contigs/MFDxxxx/final.contigs.fa reads/MFDyyyy_R1.fastq.gz reads/MFDyyyy_R2.fastq.gz | samtools view --threads 5 -Sb -F 2308 - | samtools sort --threads 3 - > map/MFDxxxx_MFDyyyy.bam
jgi_summarize_bam_contig_depths map/MFDxxxx_MFDyyyy.bam --percentIdentity 97 --outputDepth cov/MFDxxxx_MFDyyyy.tsv
```
* After calculating coverage values, the additional datasets can be appended:
```
cut -f4,5 cov/MFDxxxx_MFDyyyy.tsv > cov/MFDxxxx_MFDyyyy_trunc.tsv
paste -d "\t" cov/MFDxxxx_MFDxxxx.tsv cov/MFDxxxx_MFDyyyy_trunc.tsv > cov/MFDxxxx.tsv
```

### Binning
* The contigs were binned into MAGs with [MetaBat2](https://bitbucket.org/berkeleylab/metabat)
* Using `MFDxxxx` as an example, the command would be:
```
metabat2 -i contigs/MFDxxxx/final.contigs.fa -a cov/MFDxxxx.tsv -o bins/MFDxxxx/MFDxxxx -t 10 -m 1500 -s 100000 --saveCls
```

[//]: # (Written by Mantas Sereika)

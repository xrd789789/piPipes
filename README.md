<img src="https://dl.dropboxusercontent.com/u/5238651/pipipe%20logo.jpg" align="middle" />
Piper
=====
<img src="https://dl.dropboxusercontent.com/u/5238651/Figure%201.jpg" align="middle" />

A pipeline collection developed in the [Zamore Lab](http://www.umassmed.edu/zamore) and [ZLab](http://zlab.umassmed.edu/zlab) to analyze piRNA/transposon from different Next Generation Sequencing (*small RNA-seq*, *RNA-seq*, *Genome-seq*, *ChIP-seq*, *CAGE/Degradome-Seq*).			

In order to achieve a generic interface in terms of the genome assembles it supports, **piper** also provides a installation pipeline to download ready-to-use genome annotation packages from [Illumina iGenome](https://support.illumina.com/sequencing/sequencing_software/igenome.ilmn) as well as [UCSC Genome Browser](http://genome.ucsc.edu/).			

For *small RNA-Seq*, *RNA-Seq* and *ChIP-Seq* pipelines, **piper** provides two modes: `a single library mode` and `dual library mode`, to analyze single library and pair-wise comparison between two samples respectively.     	

##INSTALL   

```bash
# get piper code from github
git clone https://github.com/bowhan/Piper.git
```

**piper** is written in Bash, C++, Perl, Python and R. It currently only works under Linux environment.

***
### C/C++
**Piper** comes with statically compiled linux x86_64 binaries for its own C++ scripts and the other tools written in C/C++. Ideally, the users don't need to do any compiling. But if the static versions do not work in your system, exemplified by the error message "kernel too old", please compile them from src and move the binaries to the `bin`. Please install **BEDtools**, using the source code in the `third_party` directory and rename it as `bedtools_piper` in the `bin` directory of `piper`. 			
Most of **piper's** C++ code utilizes *C++11* features and *Boost* library. It is recommended to install relatively new [GCC](http://gcc.gnu.org/) and [Boost](http://www.boost.org/users/download/) for compiling them.		 
***
### Python/Cython
**For MACS2 and HTSeq-count, the users will need to install them and make them available in their `$PATH`.**        
*We cannot find a good way to ship the ready-to-use Cython code. Without `htseq-count`, `piper rna/deg/cage` won't be able to make transcripts/transposon counting using genomic coordinates. But it will still perform other functions of the pipeline, including quantification using Cufflinks and eXpress. Without `macs2`, `piper chip/chip2` won't work at all*.
***
### R
For R packages that are unavailable in the user's system, the installation is performed during the `piper install` process. They will be installed in the same directory as the pipeline in case the user doesn't have write permission in the R installation directory.
***

### Genome Annotation
Due to the limitation on the size of the files on github, the genome sequence, most annotation files are to be downloaded from somewhere else and reformatted to accommodate the pipeline. **piper** uses [iGenome](http://support.illumina.com/sequencing/sequencing_software/igenome.ilmn) and provides `piper install` to download iGenome genomes and organize the files to be used by the pipeline (see below).		
***

**piper** uses the following public tools:

1. For alignment, **piper** uses [Bowtie](http://bowtie-bio.sourceforge.net/index.shtml), [Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml), [BWA](http://bio-bwa.sourceforge.net/),  [STAR](https://code.google.com/p/rna-star/) and [mrFast](http://mrfast.sourceforge.net/) for different purposes.		

2. For transcripts/transposons quantification, **piper** uses [Cufflinks](http://cufflinks.cbcb.umd.edu/), [HTSeq](http://www-huber.embl.de/users/anders/HTSeq/) and [eXpress](http://bio.math.berkeley.edu/eXpress) under different circumstances. 	

3. For transposon mobilization as well as other structural variants discovery, **piper** uses [TEMP](http://zlab.umassmed.edu/~zhuangj/TEMP/), [BreakDancer](http://gmt.genome.wustl.edu/breakdancer/current/), [RetroSeq](https://github.com/tk2/RetroSeq) and [VariationHunter](http://compbio.cs.sfu.ca/software-variation-hunter).    

4. For ChIP-Seq peaks calling, **piper** uses [MACS2](https://github.com/taoliu/MACS). For TSS/TES/metagene analysis, **piper** uses [bwtool](https://github.com/CRG-Barcelona/bwtool).    
 
5. Additionally, **piper** uses many tools from the [Kent Tools](http://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64.v287/), like `faSize`, `bedGraphToBigWig`.   

6. To wrap bash scripts for multi-threading, **piper** utilizes `ParaFly` from [Trinity](http://trinityrnaseq.sourceforge.net/). **piper** also learns the `touch` trick for job resuming from [Trinity](http://trinityrnaseq.sourceforge.net/).	    

7. To determine the version of FastQ, **piper** uses `SolexaQA.pl` from [SolexaQA](http://solexaqa.sourceforge.net/). **Piper** have modified it in a way that the program exits as soon as the version of FastQ has been determined. The modified code can be found in the `bin` directory.		

8. **piper** uses [BEDtools](https://github.com/bowhan/bedtools.git) to assign alignments to different genomic annotations (gene, transposon, piRNA cluster, et al.). In small RNA pipeline, we made a slightly modification on `intersectBed.cpp` to accommodate a special BED format used in the pipeline. The modified code can be found in the `src/third_party` folder. A statically compiled binary, which has been renamed as `bedtools_piper` to avoid confusion with the original one, can be found in the `bin` directory. 				 		

##USAGE
The pipeline finds almost everything under its own directory so please do not move the `zpipe` script. Use `ln  -s  $PATH_TO_piper/piper  $HOME/bin/piper` to create symbol link in your `$HOME/bin`; Or add `/path/to/piper` to your `$PATH`. 
**But please do NOT add the `/path/to/piper/bin` to your `$PATH`**

Call different pipelines using:		
```Bash
# genome installation pipeline
 # 1. to install genome and R packages in one step
 # the assembly that piper supports can be found in the common/iGenome_UTL.txt file
$PATH_TO_piper/piper	install -g dm3|mm9|hg19... 
 # 2. to only download the genome and R packages (if the machine/node is not appropriate to be used for heavy computing tasks, like building indexes); then run (1) on a powerful mechine/node.
$PATH_TO_piper/piper	install -g dm3|mm9|hg19 -D
 # 3. to download the iGenome from other explicitly specified location
$PATH_TO_piper/piper	install -g hg18 -l ftp://igenome:G3nom3s4u@ussd-ftp.illumina.com/Homo_sapiens/UCSC/hg18/Homo_sapiens_UCSC_hg18.tar.gz

# to run small RNA pipeline in single library mode; input fastq can be gzipped
$PATH_TO_piper/piper	small -i input.trimmed.fq[.gz] -g dm3 -c 24

# to run small RNA pipeline in dual library mode (need single library mode output for each sample first)
$PATH_TO_piper/piper	small2 -a directory_A -b directory_B -g dm3 -c 24
# to run small RNA pipeline in dual library mode, normalized to miRNA 
$PATH_TO_piper/piper	small2 -a directory_A -b directory_B -g dm3 -c 24 -N miRNA
# to run small RNA pipeline in dual library mode, normalized to siRNA (structural loci and cis-NATs), for oxidation sample of -fruitfly only-
$PATH_TO_piper/piper	small2 -a directory_A -b directory_B -g dm3 -c 24 -N siRNA

# to run RNASeq pipeline in single library mode, dUTP based method
$PATH_TO_piper/piper	rnaseq -l left.fq -r right.fq -g mm9 -c 8 -o output_dir

# to run RNASeq pipeline in single library mode, ligation based method
$PATH_TO_piper/piper	rnaseq -l left.fq -r right.fq -g mm9 -c 8 -o output_dir -L

# to run RNASeq pipeline in dual library mode (need single library mode been ran for each sample first)
$PATH_TO_piper/piper	rnaseq2 -a directory_A -b directory_B -g mm9 -c 8 -o output_dir -A w1 -B piwi

# to run Degradome/RACE/CAGE-Seq library 
$PATH_TO_piper/piper	deg -l left.fq -r right.fq -g dm3 -c 12 -o output_dir

# to run Degradome library to check ping-pong signature with a small RNA library (need the small RNA library ran first)
$PATH_TO_piper/piper	deg -l left.fq -r right.fq -g dm3 -c 12 -o output_dir -s /path/to/small_RNA_library_output

# to run ChIP Seq library in single library mode, for narrow peak, like transcriptional factor
$PATH_TO_piper/piper	chip -l left.IP.fq -r right.IP.fq -L left.INPUT.fq -R right.INPUT.fq -g mm9 -c 8 -o output_dir

# to run ChIP Seq library in single library mode, for broad peak, like H3K9me3
$PATH_TO_piper/piper	chip -l left.IP.fq -r right.IP.fq -L left.INPUT.fq -R right.INPUT.fq -g mm9 -c 8 -o output_dir -B

# to run ChIP Seq library in single library mode, only use unique mappers (otherwise Bowtie2 randomly choose one best alignment for each read)
$PATH_TO_piper/piper	chip -l left.IP.fq -r right.IP.fq -L left.INPUT.fq -R right.INPUT.fq -g mm9 -c 8 -o output_dir -Q 10

# to run ChIP Seq library in dual library mode (need single library mode been ran for each sample first)
$PATH_TO_piper/piper	chip2 -a directory_A -b directory_B -g mm9 -c 8 -o output_dir

# to run ChIP Seq library in dual library mode, extend up/down stream 5000 bp for TSS/TES/meta analysis (for bwtool)
$PATH_TO_piper/piper	chip2 -a directory_A -b directory_B -g mm9 -c 8 -o output_dir -x 5000

# to run Genome Seq library
$PATH_TO_piper/piper	dna -l left.fq -r right.fq -g dm3 -c 24 -D 100
```
Find more detailed information on [Wiki](https://github.com/bowhan/piper/wiki)

###*install* : to install genome assembly
Due to the limitation on the size of file by github, piper doesn't ship with the genome sequences and annotation. Alternatively, we provide scrips to download genome assemly files from iGenome project of illumina. Please make sure internet is available during this process.  **piper** provides an option to separate downloading from other processes, in case the machine/node with internet access is not appropriate for building index and other works.     
Except for the genome, this pipeline will also install unavailable R packages under the pipeline directory. The downloading and installation can be separated using -D option, in case the head node is not supposed to be used for heavy computational work, like building indexes.      
Currently, **piper** comes with annotation files for *Drosophila melanogaster (dm3)*, *Mus musculus (mm9)*, *Homo sapiens (hg19)*, *Danio rerio (danRer7)*, *Rattus norvegicus (rn5)*, and *Bos taurus (bosTau7)*. *Arabidopsis thaliana (TARI10)* is also included (but not rigorously tested), though no piRNA has been described in plants. 

A more detailed explanation can be found [here](https://github.com/bowhan/piper/wiki/installation).  

###*small* : small RNA pipeline
small RNA library typically clones 18–40nt small RNAs, including miRNA, siRNA and 
piRNA. This pipeline maps those reads to rRNA, microRNA hairpin, genome, repbase 
annotated  transposons, piRNA clusters with bowtie and uses bedtools to assign 
them to different annotations. For each feature, length distribution, nucleotide percentage, 
ping-pong score, et al.,  are calculated and graphed. Some microRNA analysis is also included. 
In the dual library mode, pair-wise comparison of miRNA and piRNAs will be done. We invented this balloon-plot to efficiently compare the heterogeneity of miRNA between two samples. piRNA for different transposon family is also compared. 
For small RNA tailing analysis, please use [Tailor](http://jhhung.github.io/Tailor/) and its associated pipeline. It is current not included in **piper**.			

A more detailed explanation can be found [here](https://github.com/bowhan/piper/wiki/smallRNA).

###*rnaseq* : RNASeq pipeline
RNASeq pipeline can be used for both dUTR or ligation based RNASeq. 
It uses bowtie2 to aligns paired-end reads to rRNA and STAR[4] to align the unmapped reads 
to genome; Then it uses Cufflinks for quantification of transcripts from the genomic coordinates. It
also use HTSeq-count to quantify genomic features using coordinates. It also directly aligns reads to 
transcriptome, repbase annotated transposon, piRNA clusters using Bowtie2. Quantification 
was done using eXpress. Library is normalized by gene transcriptome compatible reads, given by Cufflinks. Basic statistics and graphs will be given. 

A more detailed explanation can be found [here](https://github.com/bowhan/piper/wiki/RNASeq).  

###*cage/deg* : CAGE & Degradome pipeline
Both types of libraries are designed to gather the information of the 5' end of RNAs 
CAGE clones RNAs with Cap and Degradome clones RNAs with 5' monophosphate. 
The pipeline will align reads to rRNA with bowtie2, genome using STAR. 
Different from RNASeq, this pipeline emphasizes the accuracy of the 5' ends. Nucleotide
composition surrounding the 5' end of the reads are given, like in small RNA library.

A more detailed explanation can be found [here](https://github.com/bowhan/piper/wiki/DegradomeSeq).

###*chip* : ChIP-Seq pipeline
ChIP Seq pipeline aligns both input and ChIP data to genome with Bowtie2. Peak calling was done
using MASC2. Signal is normalized in three different methods (ppois, FE and logLR). TSS/TES/meta plots are drawn using bwtool. In the dual library mode, peak calling is redone for each sample without inter-library normalization, by differential peak calling algorithm of MACS2 directly. TSS/TES/meta plots are drawn for those loci using the normalized signal. 

A more detailed explanation can be found [here](https://github.com/bowhan/piper/wiki/ChIPSeq). 

###*dna* : Genomic Seq pipeline
Genomic Seq pipelines aligns the paired-end reads to genome with Bowtie2, BWA-MEM and mrFast. Variations
were called using different algorithms. 

A more detailed explanation can be found [here](https://github.com/bowhan/piper/wiki/GenomeSeq).

##Citing
* in preparation

##Contact
    Wei Wang (wei.wang2 `at` umassmed.edu)
    Bo W Han (bo.han `at` umassmed.edu, bowhan `at` me.com)
    
##LICENSE
**Piper** is released under the [GNU General Public License version 3](https://www.gnu.org/licenses/gpl.html).		

##References
```
[1] Li H and Durbin R. 2009. Fast and accurate short read alignment with Burrows-Wheeler transform. Bioinformatics 25: 1754-1760.
[2] Langmead B, Trapnell C, Pop M and Salzberg SL. 2009. Ultrafast and memory-efficient alignment of short DNA sequences to the human genome. Genome Biol 10: R25.
[3] Langmead B and Salzberg SL. 2012. Fast gapped-read alignment with Bowtie 2. Nat Methods 9: 357-359.
[4] Dobin A, Davis CA, Schlesinger F, Drenkow J, Zaleski C, Jha S, Batut P, Chaisson M and Gingeras TR. 2013. STAR: ultrafast universal RNA-seq aligner. Bioinformatics 29: 15-21.
[5] Trapnell C, Williams BA, Pertea G, Mortazavi A, Kwan G, van Baren MJ, Salzberg SL, Wold BJ and Pachter L. 2010.
[6] Transcript assembly and quantification by RNA-Seq reveals unannotated transcripts and isoform switching during cell differentiation. Nat Biotechnol 28: 511-515.
[7] Kent WJ, Zweig AS, Barber G, Hinrichs AS and Karolchik D. 2010. BigWig and BigBed: enabling browsing of large distributed datasets. Bioinformatics 26: 2204-2207.
[8] Zhang Y et al. 2008. Model-based analysis of ChIP-Seq (MACS). Genome Biol 9: R137.
[9] Roberts A and Pachter L. 2013. Streaming fragment assignment for real-time analysis of sequencing experiments. Nat Methods 10: 71-73.
[10] Grabherr MG et al. 2011. Full-length transcriptome assembly from RNA-Seq data without a reference genome. Nat Biotechnol 29: 644-652.
[11] Karolchik D, Hinrichs AS, Furey TS, Roskin KM, Sugnet CW, Haussler D and Kent WJ. 2004. The UCSC Table Browser data retrieval tool. Nucleic Acids Res 32: D493-D496.
[12] Cox MP, Peterson DA and Biggs PJ. 2010. SolexaQA: At-a-glance quality assessment of Illumina second-generation sequencing data. BMC Bioinformatics 11: 485.
[13] HTSeq: Analysing high-throughput sequencing data with Python. [http://www-huber.embl.de/users/anders/HTSeq/]
[14] Keane TM, Wong K and Adams DJ. 2013. RetroSeq: transposable element discovery from next-generation sequencing data. Bioinformatics 29: 389-390.
[15] Hormozdiari F, Hajirasouliha I, Dao P, Hach F, Yorukoglu D, Alkan C, Eichler EE and Sahinalp SC. 2010. Next-generation VariationHunter: combinatorial algorithms for transposon insertion discovery. Bioinformatics 26: i350-i357.
[16] Pohl, A. & Beato, M. bwtool: a tool for bigWig files. Bioinformatics (2014).
[17] Zhang, H., Meltzer, P. & Davis, S. RCircos: an R package for Circos 2D track plots. BMC Bioinformatics 14, 244 (2013).
```

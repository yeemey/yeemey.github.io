## [4 min. read](http://www.niram.org/read/)
## Some background...
About 10 months ago, armed with short-read sequences of experimentally evolved bacterial and archaeal populations, I decided that it would be "easier" to use haplotype reconstruction software to recover genotypes, than to either eyeball population-level variant frequencies over time (planning to write a paper on this - will link when it's in preprint), and then logically deduce which variants occurred on the same genetic background, or to sample genotypes from the population by selecting a number of clonal isolates for whole genome sequencing (planning a blog post on how to determine the number of isolates to appropriately sample a population with given amount of genetic diversity). I wasn't confident that I could do the former in a coherent, reproducible way, and in rejecting the latter, I was trying to avoid more lab work and sequencing costs.

To no one's surprise, the software route was not easy. Frustration got the better of me then, and I shelved the effort, but now I'm ready to give it another shot, and thought I should record the attempts here, mainly for my own reference, but also in the hopes that it'll be useful to someone else.

### My data features
The samples are populations of two microbial species, founded by clonal lab isolates - within-species genetic diversity is therefore not as high as say, an environmental metagenomics sample, or virus populations. The raw data are FASTQ files, obtained from Illumina HiSeq platform, paired end 2x150 bp reads.

Following adapter and quality trimming with [scythe](https://github.com/vsbuffalo/scythe) and [sickle](https://github.com/najoshi/sickle), I used [breseq](https://github.com/barricklab/breseq) for variant-calling. Breseq relies on [bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml) for read alignment to reference, and I'm going to use the resulting BAM and reference FASTA files for haplotype reconstruction trials.

## Begin tool trials!
### 1. [ShoRAH](https://github.com/cbg-ethz/shorah) v1.1.0
Zagordi, O., Bhattacharya, A., Eriksson, N., Beerenwinkel, N. [BMC Bioinformatics. 2011. 12:119](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-12-119)

#### Verdict
Not for (my) clonal, bacterial/archaeal population. It was [developed for](https://github.com/cbg-ethz/shorah/issues/40), and tested on highly diverse HIV sequences. ::sadface:: Nevertheless, a few lessons were learned:

#### Installation
Lesson from first attempt to install on a Linux Ubuntu server, on which I didn't have root permission: Use [conda](https://anaconda.org/) and the [bioconda channel to install ShoRAH](https://anaconda.org/bioconda/shorah) in a virtual environment to save yourself a headache (many thanks to [@DrYak for helping me resolve them](https://github.com/cbg-ethz/shorah/issues/60)).

Lesson from second attempt to install on a Linux, now CentOS, server (still not root): After creating a virtual environment, and downloading ShoRAH via the bioconda channel, the shared [libopenblas package](https://anaconda.org/anaconda/libopenblas) that ShoRAH needs isn't accessible from my virtual environment, so I just went ahead and conda-installed that myself in the environment.

#### Test data
I LOVE software that provides test data so I know how things should operate, and what the input/output files should look like. Since I used conda to install it in my virtual environment, the test data ended up here: ```anaconda3/envs/my_env/share/shorah/amplicon_test/```

As you can tell from the directory name though, the test data is only for the ```amplian.py``` module, which is odd since they warn you in the README that ```amplian.py``` is still under development. Anyway, the good news is that I had zero problems executing ```amplian.py``` on the test data!

I tried running `dec.py` on the test data as well, but it did not go well:
```
Traceback (most recent call last):
  File "~/anaconda3/envs/my_env/bin/dec.py", line 78, in <module>
    args.region, args.max_coverage, args.alpha, args.keep_files, args.seed)
  File "~/anaconda3/envs/my_env/lib/python3.5/site-packages/shorah_dec.py", line 401, in main
    r = list(aligned_reads.keys())[0]
IndexError: list index out of range
```

#### Real data
[As recommended](https://cbg-ethz.github.io/shorah/input.html), I used ```samtools sort``` on the BAM file I got previously from running variant call analysis with breseq. Starting with the default parameters on ```dec.py```, which does local haplotype analysis, I got the error message ```b2w run not successful```, which based on [this issue](https://github.com/cbg-ethz/shorah/issues/40), suggested that the genetic diversity in my sample was too low.

But I tried again, this time [setting the window size](https://cbg-ethz.github.io/shorah/local.html) to 51 to take into account the size of my reads (150 bp). Note that ```Window size must be divisible by win_shifts```, and win_shifts is set to 3, by default.

Still, ```b2w run not successful```.

Zagordi et al. suggested that for Illumina short reads, I should first use ```bam2msa.py``` to get a single analysis window from the multiple sequence alignment, and run ```diri_sampler``` on it. This is as opposed to how ```dec.py``` passes each of a SET of overlapping windows to ```diri_sampler```. However, I couldn't find ```bam2msa.py``` in my version of ShoRAH.

--

Having learned my lesson about using tools that aren't intended for, or validated on specific types of data, next time on Lab Notes, I'll cover [EVORhA](bioinformatics.intec.ugent.be/kmarchal/EVORhA/), "the first bacterial haplotype reconstruction method" for the analysis of "pooled sequence data obtained from populations of clonal organisms with a low mutation frequency and/or data obtained with a short-read based technology."

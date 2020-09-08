## [ 38 sec. read](http://www.niram.org/read/)
The [basic steps in a variant-calling workflow](https://datacarpentry.org/wrangling-genomics/02-quality-control/index.html) based on mapping short reads to reference genomes are so established in my mind that when I finally got to play with our nanopore sequencer, it took me about a week of fiddling* to realize that mapping long reads doesn't proceed in quite the same way.

Short reads mapping software like bowtie2 or bwa take advantage of the high coverage and quality scores (generally Q30 or better) to ... Besides the obvious difference of long reads being long (hundreds to thousands of bases, vs. tens to hundreds of bases), their quality scores are also not (yet) comparable, on average Q10 and up. Therefore, "mapping" long reads actually involves assembly first.



*searching on forums like the nanopore community and BioStars, and installing/using moar software.

# peak-calling-tutorial

Peak calling is a computational/statistical method used to identify areas in a genome that have been enriched with aligned reads as a consequence of performing a ChIP-sequencing
Peaks are defined as areas of enrichment where more sequences are aligned than would be expected by chance thus indicating locations of binding sites or histone modifications
Form ChIP-seq experiments, what we usually observe from the alignment files is a strand asymmetry with read densities on the +/- strand, centered around the binding site.
The distributions of these groups are then assessed using statistical measures and compared against background to determine if the site of enrichment is likely to be a real binding site.
![](/images/plos_chipseq_arrow.png)

___
### __Duplicate Removal:__
___
MACS offers several option to deal with duplicates we can use the auto option which is the most commonly used option tells MACS to calculate the maximum tags at the exact same location based on binomal distribution using a pvalue cutoff.
Another option is the all option  which keeps every tag. If a number is specified, then at most that many duplicates will be kept at the same location

___
### __Modeling the shift size:__
___
MACS first scans the whole dataset searching for highly significant enriched regions. Given a sonication size (bandwidth) and a high-confidence fold-enrichment (mfold),
MACS slides two bandwidth windows across the genome to find regions with tags more than mfold enriched.
the it randomly samples 1,000 of these high-quality peaks, separates their positive and negative strand tags,and aligns them by the midpoint between their centers. 
The distance between the modes of the two peaks in the alignment is defined as ‘d’ and represents the estimated fragment length

___
### __Peak detection:__
___
The read distribution along the genome can be modeled by a Poisson distribution. The Poisson is a one parameter model, where the parameter λ is the expected number of reads in that window.

![](/images/325px-Poisson_pmf.svg.png)

MACS uses a dynamic parameter, λlocal, defined for each candidate peak. The lambda parameter is estimated from the control sample and is deduced by taking the maximum value across various window sizes.


___
### __Data sets:__
___

our dataset is from ENCODE
https://www.encodeproject.org/experiments/ENCSR264RJX/
POU5F1 (hg38)
https://www.encodeproject.org/files/ENCFF720OTS/@@download/ENCFF720OTS.bam
https://www.encodeproject.org/files/ENCFF582ZOC/@@download/ENCFF582ZOC.bam

https://www.encodeproject.org/experiments/ENCSR061DGF/
NANOG (hg38)
https://www.encodeproject.org/files/ENCFF452BSY/@@download/ENCFF452BSY.bam
https://www.encodeproject.org/files/ENCFF138ZXJ/@@download/ENCFF138ZXJ.bam

https://www.encodeproject.org/experiments/ENCSR447FVP/
control (hg38)
https://www.encodeproject.org/files/ENCFF982GQU/@@download/ENCFF982GQU.bam
These are 2 transcription factors that are involved in stem cell pluripotency


___
### __Hands-on__
___

1. Get the tutorial material

        cp -r /cta/users/mayoubi/peak_calling_tutorial peak_calling_tutorial

2. copy a slurm_example.sh script
	
	cp /cta/share/slurm_example.sh

3. inside your slurm_example.sh loard your module and add the macs2 command for the datafiles

	nano slurm_example.sh
	module load macs2-2.2.6
	macs2 callpeak -t Nanog_Rep1.bam -c control.bam -f BAM -g 1.3e+8 -n Nanog-rep1 --outdir macs2
4. dispatch the slurm script
	
	sbatch slurm_example.sh

5.  if the code return with no errors do the same for the res of the data files

	macs2 callpeak -t Nanog_Rep2.bam -c control.bam -f BAM -g 1.3e+8 -n Nanog-rep1 --outdir macs2
	macs2 callpeak -t Nanog_Rep2.bam -c control.bam -f BAM -g 1.3e+8 -n Nanog-rep1 --outdir macs2
	macs2 callpeak -t Nanog_Rep2.bam -c control.bam -f BAM -g 1.3e+8 -n Nanog-rep1 --outdir macs2
	





___
### __Output__
___
1. `NAME_peaks.xls` is a tabular file which contains information about
   called peaks. You can open it in excel and sort/filter using excel
   functions. Information include:
   
    - chromosome name
    - start position of peak
    - end position of peak
    - length of peak region
    - absolute peak summit position
    - pileup height at peak summit
    - -log10(pvalue) for the peak summit (e.g. pvalue =1e-10, then
      this value should be 10)
    - fold enrichment for this peak summit against random Poisson
      distribution with local lambda,
    - -log10(qvalue) at peak summit
   


2. `NAME_peaks.narrowPeak` is BED6+4 format file which contains the
   peak locations together with peak summit, p-value, and q-value. You
   can load it to the UCSC genome browser. Definition of some specific
   columns are:
   
   - 5th: integer score for display. It's calculated as
     `int(-10*log10pvalue)` or `int(-10*log10qvalue)` depending on
     whether `-p` (pvalue) or `-q` (qvalue) is used as score
     cutoff. Please note that currently this value might be out of the
     [0-1000] range defined in [UCSC ENCODE narrowPeak
     format](https://genome.ucsc.edu/FAQ/FAQformat.html#format12). You
     can let the value saturated at 1000 (i.e. p/q-value = 10^-100) by
     using the following 1-liner awk: `awk -v OFS="\t"
     '{$5=$5>1000?1000:$5} {print}' NAME_peaks.narrowPeak`
   - 7th: fold-change at peak summit
   - 8th: -log10pvalue at peak summit
   - 9th: -log10qvalue at peak summit
   - 10th: relative summit position to peak start


3. `NAME_summits.bed` is in BED format, which contains the peak
   summits locations for every peak. The 5th column in this file is
   the same as what is in the `narrowPeak` file. The file
   can be loaded directly to the UCSC genome browser. Remove the
   beginning track line if you want to analyze it by other tools.

4. `NAME_peaks.broadPeak` is in BED6+3 format which is similar to
   `narrowPeak` file, except for missing the 10th column for
   annotating peak summits. This file and the `gappedPeak` file will
   only be available when `--broad` is enabled. 

5. `NAME_peaks.gappedPeak` is in BED12+3 format which contains both
   the broad region and narrow peaks. The 5th column is the score for
   showing grey levels on the UCSC browser as in `narrowPeak`. The 7th
   is the start of the first narrow peak in the region, and the 8th
   column is the end. The 9th column should be RGB color key.
 The 10th column tells how many blocks including the starting
   1bp and ending 1bp of broad regions. The 11th column shows the
   length of each block and 12th for the start of each block. 13th:
   fold-change, 14th: *-log10pvalue*, 15th: *-log10qvalue*. The file can
   be loaded directly to the UCSC genome browser. Refer to
   `narrowPeak` if you want to fix the value issue in the 5th column.

6. `NAME_model.r` is an R script which you can use to produce a PDF
   image of the model based on your data. Load it to R by:

   `$ Rscript NAME_model.r`

   Then a pdf file `NAME_model.pdf` will be generated in your current
   directory. Note, R is required to draw this figure.

7. The `NAME_treat_pileup.bdg` and `NAME_control_lambda.bdg` files are
   in bedGraph format which can be imported to the UCSC genome browser
   or be converted into even smaller bigWig files. The
   `NAME_treat_pielup.bdg` contains the pileup signals (normalized
   according to `--scale-to` option) from ChIP/treatment sample. The
   `NAME_control_lambda.bdg` contains local biases estimated for each
   genomic location from the control sample, or from treatment sample
   when the control sample is absent. 
___

### __Other useful links__

1. https://github.com/taoliu/MACS
___

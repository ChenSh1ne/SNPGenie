# SNPGenie

Perl software for estimating evolutionary parameters from pooled next-generation sequencing single-nucleotide polymorphism data. Just run **snpgenie-1.2.pl** in a directory containing the necessary input files, and we take care of the rest!

## Introduction

New applications of next-generation sequencing (NGS) use pooled samples containing DNA from multiple individuals to perform population genetic analyses. SNPGenie is a Perl program which can analyze the single-nucleotide polymorphism (SNP) calls from such data to calculate evolutionary parameters such as nucleotide diversity (including its nonsynonymous and synonymous partitions, πN and πS) and gene diversity. These calls are typically present in annotation tables and assume that the pooled nucleic acid sample is representative of the population of interest. For example, if one is interested in determining the nucleotide diversity of a virus population within a single host, it would be appropriate to sequence the pooled nucleic acid content of the viruses in a blood sample from that host. Comparing πN and πS for, say, a gene product, or comparing gene diversity at sites of different types, may help to dicepher instances of positive (Darwinian) selection, negative (purifying) selection, and random genetic drift. For additional background, see Nelson & Hughes (2015).

## SNPGenie Input

SNPGenie version 1.2 is a command-line interface application written in Perl. As such, it is limited only by the memory and processing capabilities of the local hardware. As input, it accepts:

1. One or more **reference sequence** files in **FASTA** format (.fa/.fasta); 
2. One file with CDS information in **Gene Transfer Format** (.gtf); and 
3. One or more tab-delimited (.txt) **SNP reports** in CLC or Geneious format. If you want another format included, just ask! 

Further details on input are below.

### Reference Sequence
The reference sequence must be present in a **FASTA** (.fa/.fasta) file. Providing only one reference sequence assumes...

### Gene Transfer Format

The Gene Transfer Format (.gtf) file is tab (\t)-delimited, and must include records for all CDS elements (i.e., open reading frames, or ORFs) present in your SNP Report(s). Note that SNPGenie expects every coding element to be labeled as type "CDS", and for its product name to follow the "gene_id" tag. (This name must match that present in the SNP report.) If a single coding element has multiple segments with different coordinates, simply enter one line for each segment, using the same product name. SNPGenie for CLC can currently handle 2 segments per ORF; if more are needed, just contact us! For more information about GTF, please visit <http://mblab.wustl.edu/GTF22.html>. A simple example follows:

	reference.gbk	CLC	CDS	5694	8369	.	+	0	gene_id "ORF1";
	reference.gbk	CLC	CDS	8203	8772	.	+	0	gene_id "ORF2";
	reference.gbk	CLC	CDS	1465	4485	.	+	0	gene_id "ORF3";
	reference.gbk	CLC	CDS	5621	5687	.	+	0	gene_id "ORF4";
	reference.gbk	CLC	CDS	7920	8167	.	+	0	gene_id "ORF4";
	reference.gbk	CLC	CDS	5395	5687	.	+	0	gene_id "ORF5";
	reference.gbk	CLC	CDS	7920	8016	.	+	0	gene_id "ORF5";
	reference.gbk	CLC	CDS	4439	5080	.	+	0	gene_id "ORF6";
	reference.gbk	CLC	CDS	5247	5549	.	+	0	gene_id "ORF7";
	reference.gbk	CLC	CDS	4911	5246	.	+	0	gene_id "ORF8";

### CLC Genomics Workbench SNP Reports

At minimum, a CLC SNP report must include the following 8 default column selections, with the unaltered CLC column headers: 

* **Reference Position**, which refers to the start site of the polymorphism within the reference FASTA sequence;
* **Type**, which refers to the nature of the record, usually the type of polymorphism, e.g., "SNV” for single-nucleotide variants;
* **Reference**, the reference nucleotide(s) at that site(s);
* **Allele**, the variant nucleotide(s) at that site(s);
* **Count**, the number of reads containing the variant;
* **Coverage**, the total number of sequencing reads at the site(s);
* **Frequency**, the frequency of the variant as a percentage, e.g., “14.6” for 14.60%; and
* **Overlapping annotations**, containing the name of the protein product or open reading frame (ORF), e.g., “CDS: ORF1”.

In addition to the aforementioned columns, the SNP report should ideally be free of thousand separators (,) in the Reference Position, Count, and Coverage columns (default format). The Frequency must remain a percentage (default format). Finally, the user should verify that the reading frame in the CLC output is correct. SNPGenie will produce various errors to indicate when these things are not so, e.g., by checking that all products begin with START and end with STOP codons, and checking for premature stop codons. Make sure to check the SNPGenie LOG file!

### Geneious SNP Reports

At minimum, the Geneious SNP report must include the following default column selections, with the unaltered Geneious column headers:

* **Minimum** and **Maximum**, which refer to the start and end sites of the polymorphism within the reference FASTA sequence, and will hold the same value for SNP records;
* **CDS Position**, with the coordinate of the site relative to the start cite of the relevant CDS annotation;
* **Type**, which refers to the nature of the record entry, e.g., “Polymorphism”;
* **Polymorphism Type**, which gives the type of polymorphism;
* **product**, containing the name of the protein product or open reading frame, e.g., ORF1; 
* **Change**, which contains the reference and variant nucleotides, e.g., "A -> G", and are always populated for SNP records;
* **Coverage**, containing the number of sequencing reads that include the site; and
* **Variant Frequency**, which contains the frequency of the nucleotide variant as a percentage, e.g., 14.60%.

As with CLC, the Geneious SNP report should ideally be free of extraneous characters such as thousand separators (,), but SNPGenie will do its best to adapt if they are present. Again, the Variant Frequency must remain a percentage (default format). Again, the user should verify that the reading frame in the Geneious output is correct. SNPGenie will produce various errors to indicate when these things are not so, e.g., by checking that all products begin with START and end with STOP codons, and checking for premature stop codons. Make sure to check the SNPGenie LOG file!

## Options

In case you want to alter the way snpgenie works, the following options (implemented using Perl's Getopt::Long module) may be used:

* **--minfreq**: optional floating point parameter specifying the minimum allele (SNP) frequency to include. Enter as a proportion/decimal (e.g., 0.01), **not** as a percentage (e.g., 1.0%). Default: 0.
* **--snpreport**: optional string parameter specifying the (one) SNP report to analyze. Default: auto-detect .txt and .csv file(s).
* **--fastafile**: optional string parameter specifying the (one) reference sequence. Default: auto-detect .fa and/or .fasta file(s).
* **--gtffile**: optional string parameter specifying the one file with CDS annotations. Default: auto-detect the .gtf file.
* **--sepfiles**: optional Boolean (flag) parameter specifying whether to product separate results (codon) files for each SNP report (all results already included together in the codon_results.txt file). Simply include in the command line to activate. Default: not included.
* **--slidingwindow**: optional integer parameter specifying the length of the sliding (codon) window used in the analysis. Default: 9 codons.
* **--ratiomode**: optional Boolean (flag) parameter specifying whether to include π values for each codon in the codon_results.txt file(s). This is usually inadvisable, as π values (especially πS) are subject to great stochastic error. Simply include in the command line to activate. Default: not included.
* **--sitebasedmode**: optional Boolean (flag) parameter specifying whether to include π values derived using a site-based (reference codon context only) approach in the codon_results.txt file(s). This is usually inadvisable, as π values will not reflect the true population pairwise comparisons. Simply include in the command line to activate. Default: not included.

For example, if you wanted to turn on the **sepfiles** option, specify a minimum allele frequency of 0.1 and specify your input files, you could enter the command:

	snpgenie-1.2.pl --sepfiles --minfreq=0.01 --snpreport=mySNPreport.txt --fastafile=myFASTA.fa --gtffile=myGTF.gtf

## How SNPGenie Works

Given the appropriate files, SNPGenie calculates nucleotide diversities for nonsynonymous and synonymous sites in a protein-coding sequence. Nucleotide diversity may be defined as the average number of nucleotide variants per nucleotide site for all pairwise comparisons. To distinguish between nonsynonymous and synonymous differences and sites, it is necessary to consider the codon context of each nucleotide in a sequence. This is why the user must submit the starting and ending sites of the coding regions in the .gtf file, along with the reference FASTA sequence file, to allow an accurate estimation of the number of nonsynonymous and synonymous sites for each codon by the Nei-Gojobori (1986) method. SNPGenie first splits the coding sequence into codons, each of which contains 3 sites. The software then determines the number of these sites which are nonsynonymous and synonymous by testing all possible changes at each site of every codon in the sequence. Because different nucleotide variants at the same site may lead to both nonsynonymous and synonymous polymorphisms, fractional sites occur frequently (e.g., only 2 of 3 possible nucleotide substitutions at the third position of AGA cause an amino acid change; thus, that site is considered 2/3 nonsynonymous and 1/3 synonymous). Next, the SNP report is consulted for the presence of variants to produce a revised estimate. Variants are incorporated through averaging weighted by their frequency. Although it is a rare occurrence, high levels of sequence variation may alter the number of nonsynonymous and synonymous sites in a particular codon, contributing to an altered picture of natural selection.

Next, SNPGenie calculates the number of nucleotide differences for each codon in each ORF specified in the .gtf file. Calculating nucleotide diversity codon-by-codon enables sliding-window analyses that may help to pinpoint important nucleotide regions subject to varying forms of natural selection. SNPGenie determines the average number of pairwise differences as follows: for every variant in the SNP Report, the number of variants is calculated as the product of the variant’s relative frequency and the coverage at that site. For each variant nucleotide (up to 3 non-reference nucleotides), the number of variants is stored, and their sum is subtracted from the coverage to yield the reference’s absolute frequency. Next, for each pairwise nucleotide comparison at the site, it is determined whether the comparison represents a nonsynonymous or synonymous change. If the former, the product of their absolute frequencies contributes to the number of nonsynonymous differences; if the latter, it contributes to the number of synonymous differences. When comparing codons with more than one nucleotide difference, all possible mutational pathways are considered, per the methods of Nei & Gojobori (1986). The sum of pairwise differences is divided by the total number of pairwise comparisons at the codon (i.e., cC2, where c = coverage) to yield the number of differences per site of each type. This is calculated separately for nonsynonymous and synonymous comparisons. For further background, see Nelson & Hughes (2015).

To run SNPGenie, first download the appropriate script and place it in your system’s PATH, or simply in your working directory. Next, place your SNP Report (.txt), FASTA (.fa/.fasta), and GTF (.gtf) files in your working directory (Figure 1). Open the command line prompt or terminal and navigate to the directory containing these files (Figure 2). Then simply execute SNPGenie (Figure 3).

SNPGenie for CLC automatically detects the number of FASTA files in your working directory. If there is one FASTA file, then SNPGenie enters its ONE-SEQUENCE MODE, where it is assumed that all SNP Reports refer to the same reference sequence, contained in the FASTA. This is the most common application. For example, one FASTA file might contain the reference (e.g., consensus) sequence for a virus population, while the GTF file contains the coordinates of the ORFs in the viral genome, and the SNP Reports contains SNPs called from different samples against the aforementioned reference.
There are some situations in which multiple FASTA sequences may be necessary, e.g., the segmented genome of an influenza virus. If there are multiple FASTA sequences in the working directory, SNPGenie for CLC will enter its MULTI-SEQUENCE MODE, and will look for a separate FASTA file for each ORF. These files must begin with the name of the respective ORF, followed by an underscore (“_”). For example, if one product was named ORF1, a FASTA file would need to exist by the name of ORF1_x.fa, where x is any string of alphanumeric characters.

Along with the aforementioned WARNINGS, SNPGenie will also alert you to such aberrations as missing FASTA files, incorrectly named ORFs, and reference sequences which do not match what is reported in the SNP Report. In most instances, SNPGenie will terminate. If this occurs, you may consult either the Terminal (command line) screen, or else the new SNPGenie_Results directory (created after running SNPGenie in your working directory) for the SNPGenie_WARNINGS.txt file. Also see section 5. Troubleshooting below.

## Output

SNPGenie creates a new folder called SNPGenie_Results within the working directory. This contains the following TAB-delimited results files:

1. **SNPGenie_parameters.txt**, containing the input parameters and file names.

2. **SNPGenie_LOG.txt**, documenting any peculiarities or errors encountered. Warnings are also printed to the Terminal (shell) window.

3. **site_results.txt**, providing results for all polymorphic sites. Columns are:
	* *file*. The SNP report analyzed.
	* *product*. The CDS annotation to which the site belongs; "noncoding" if none.
	* *site*. The site coordinate of the nucleotide in the reference sequence.
	* *ref_nt*. The identity of the nucleotide in the reference sequence.
	* *maj_nt*. The most common nucleotide in the population at this site.
	* *position_in_codon*. If present in a CDS annotation, the position of this site within its codon (1, 2, or 3).
	* *overlapping_ORFs*. The number of CDS annotations overlapping this site. For example, if the site is part of only one open reading frame, the value will be 0. If the site is part of two open reading frames, the value will be 1.
	* *codon_start_site*. The site coordinate of the relevant codon's first nucleotide in the reference sequence.
	* *codon*. The identity of the relevant codon.
	* *pi*. Nucleotide diversity at this site.
	* *gdiv*. Gene diversity at this site.
	* *class_vs_ref*. This site's classification, as compared to the reference sequence. For example, if the site contains only one SNP, and that SNP is synonymous, the site will be classified as Synonymous. Nonsynonymous or Synonymous.
	* *class*. This site's classification as compared to all sequences present in the population. For example, if the population contains both A and G residues at the third site of a GAA (reference) codon, then the site will be Synonymous, because both GAA and GAG encode Glu. On the other hand, if the population also contains a C at this site, the site will be Ambiguous, because GAC encodes Asp, meaning both nonsynonymous and synonymous polymorphisms exist at the site. Nonsynonymous, Synonymous, or Ambiguous.
	* *coverage*. The NGS read depth at the site.
	* *A*. The number of reads containing an A (adenine) nucleotide at this site. N.B.: may be fractional if the coverage and variant frequency given in the SNP report do not imply a whole number.
	* *C*. For C (cytosine), as for A.
	* *G*. For G (guanine), as for A.
	* *T*. For T (thymine), as for A.

4. **codon_results.txt**, providing results for all polymorphic sites. Columns are:
	* *file*. The SNP report analyzed.
	* *product*. The CDS annotation to which the site belongs; "noncoding" if none.
	* *site*. The site coordinate of the nucleotide in the reference sequence.
	* *codon*. The identity of the relevant codon.
	* *num_overlap_ORF_nts*. The number of nucleotides in this codon (up to 3) which overlap other ORFs (in addition to the current "product" annotation).
	* *mean_nonsyn_diffs*. The mean number of pairwise nucleotide comparisons in this codon which are nonsynonymous (i.e., amino acid-altering) in the pooled sequence sample. The numerator of πN.
	* *mean_syn_diffs*. The mean number of pairwise nucleotide comparisons in this codon which are synonymous (i.e., amino acid-conserving) in the pooled sequence sample. The numerator of πS.
	* *nonsyn_sites*. The mean number of sites in this codon which are nonsynonymous, given all sequences in the pooled sample. The denominator of πN and mean πN versus the reference.
	* *syn_sites*. The mean number of sites in this codon which are synonymous, given all sequences in the pooled sample. The denominator of πS mean πS versus the reference.
	* *nonsyn_sites_ref*. The number of sites in this codon which are nonsynonymous in the reference sequence.
	* *syn_sites_ref*. The number of sites in this codon which are synonymous in the reference sequence.
	* *mean_nonsyn_diffs_vs_ref*. This codon's mean number of nonsynonymous nucleotide differences from the reference sequence in the pooled sequence sample. The numerator of mean πN versus the reference.
	* *mean_syn_diffs_vs_ref*. This codon's mean number of synonymous nucleotide differences from the reference sequence in the pooled sequence sample. The numerator of mean πS versus the reference.
	* *mean_gdiv*. Mean gene diversity (observed heterozygosity) for this codon's nucleotide sites.
	* *mean_nonsyn_gdiv*. Mean gene diversity for this codon's nonsynonymous polymorphic sites.
	* *mean_syn_gdiv*. Mean gene diversity for this codon's synonymous polymorphic sites.

5. **\<SNP report name(s)\>_results.txt**, containing the information present in the codon_results.txt file, but separated by SNP report.

6. **product_results.txt**, providing results for all CDS elements present in the GTF file. Columns are:
	* *file*. The SNP report analyzed.
	* *product*. The CDS annotation to which the site belongs; "noncoding" if none.
	* *mean_nonsyn_diffs*. The sum over all codons in this product of the mean number of pairwise nucleotide comparisons which are nonsynonymous (i.e., amino acid-altering) in the pooled sequence sample. The numerator of πN.
	* *mean_syn_diffs*. The sum over all codons in this product of the mean number of pairwise nucleotide comparisons which are synonymous (i.e., amino acid-conserving) in the pooled sequence sample. The numerator of πS.
	* *mean_nonsyn_diffs_vs_ref*. The sum over all codons in this product of the mean number of nonsynonymous nucleotide differences from the reference sequence in the pooled sequence sample. The numerator of mean πN versus the reference.
	* *mean_syn_diffs_vs_ref*. The sum over all codons in this product of the mean number of synonymous nucleotide differences from the reference sequence in the pooled sequence sample. The numerator of mean πS versus the reference.
	* *nonsyn_sites*. The mean number of sites in this product which are nonsynonymous, given all sequences in the pooled sample. The denominator of πN and mean πN versus the reference.
	* *syn_sites*. The mean number of sites in this product which are synonymous, given all sequences in the pooled sample. The denominator of πS and mean πS versus the reference.
	* *piN*. The mean number of pairwise nonsynonymous differences per nonsynonymous site in this product.
	* *piS*. The mean number of pairwise synonymous differences per synonymous site in this product.
	* *mean_dN_vs_ref*. The mean number of nonsynonymous differences from the reference per nonsynonymous site in this product.
	* *mean_dS_vs_ref*. The mean number of synonymous differences from the reference per synonymous site in this product.
	* *mean_gdiv_polymorphic*. Mean gene diversity (observed heterozygosity) at all polymorphic nucleotide sites in this product.
	* *mean_gdiv_nonsyn*. Mean gene diversity at all nonsynonymous polymorphic nucleotide sites in this product.
	* *mean_gdiv_syn*. Mean gene diversity at all synonymous polymorphic nucleotide sites in this product.

7. **population_summary.txt**, providing summary results for each population's sample (SNP report). Columns are:
	* *file*. The SNP report analyzed.
	* *sites*. Total number of sites in the reference genome.
	* *sites_coding*. Total number of sites in the reference genome which code for a protein product, given the CDS annotations in the GTF file.
	* *sites_noncoding*. Total number of sites in the reference genome which do not code for a protein product, given the CDS annotations in the GTF file.
	* *pi*. Mean number of pairwise differences per site in the pooled sample across the whole genome.
	* *pi_coding*. Mean number of pairwise differences per site in the pooled sample across all coding sites in the genome.
	* *pi_noncoding*. Mean number of pairwise differences per site in the pooled sample across all noncoding sites in the genome.
	* *nonsyn_sites*. The mean number of sites in the genome which are nonsynonymous, given all sequences in the pooled sample. The denominator of πN and mean πN versus the reference.
	* *syn_sites*. The mean number of sites in the genome which are synonymous, given all sequences in the pooled sample. The denominator of πS and mean πS versus the reference.
	* *piN*. The mean number of pairwise nonsynonymous differences per nonsynonymous site across the genome of the pooled sample.
	* *piS*. The mean number of pairwise synonymous differences per synonymous site across the genome of the pooled sample.
	* *mean_dN_vs_ref*. The mean number of nonsynonymous differences from the reference per nonsynonymous site across the genome of the pooled sample.
	* *mean_dS_vs_ref*. The mean number of synonymous differences from the reference per synonymous site across the genome of the pooled sample.
	* *mean_gdiv_polymorphic*. Mean gene diversity (observed heterozygosity) at all polymorphic nucleotide sites in the genome of the pooled sample.
	* *mean_gdiv_nonsyn*. Mean gene diversity at all nonsynonymous polymorphic nucleotide sites in the genome of the pooled sample.
	* *mean_gdiv_syn*. Mean gene diversity at all synonymous polymorphic nucleotide sites in the genome of the pooled sample.
	* *mean_gdiv*. Mean gene diversity at all nucleotide sites in the genome of the pooled sample.
	* *sites_polymorphic*. The number of sites in the genome of the pooled sample which are polymorphic.
	* *mean_gdiv_coding_poly*. Mean gene diversity at all polymorphic nucleotide sites in the genome of the pooled sample which code for a protein product, given the CDS annotations in the GTF file.
	* *sites_coding_poly*. The number of sites in the genome of the pooled sample which are polymorphic and code for a protein product, given the CDS annotations in the GTF file.
	* *mean_gdiv_noncoding_poly*. Mean gene diversity at all polymorphic nucleotide sites in the genome of the pooled sample which do not code for a protein product, given the CDS annotations in the GTF file.
	* *sites_noncoding_poly*. The number of sites in the genome of the pooled sample which are polymorphic and do not code for a protein product, given the CDS annotations in the GTF file.

8. **sliding_window_length<Length>_results.txt**, containing codon-based results over a sliding window, with a default length of 9 codons.



OLD:

2. Nucleotide_diversity_results.txt. This file contains the nucleotide diversity results for all SNP Reports processed. The columns are:
	* File. The SNP Report analyzed.
	* Product. The protein product or ORF.
	* Site. The site of the first nucleotide in this row’s codon, relative to the reference sequence.
	* Codon. The codon to which this row applies.
	* Num Overlap ORF Nts. The number of nucleotides within this codon (ranging from 0 to 3) which overlap another ORF. For example, if a codon is contained within ORF1 and ORF2, this value will be 3.
	* Nonsyn Diffs. For the codons observed at this site, determined using variant and coverage data, this column gives the mean number of pairwise differences per site which are nonsynonymous (amino acid-altering). For codon comparisons in which ≥2 nucleotides differ, all possible mutational pathways are considered via an adaptation of the Nei & Gojobori (1986) method (Nelson & Hughes 2015).
	* Syn Diffs. For all possible codons observed at this site, determined using variant and coverage data, this column gives the mean number of pairwise differences per site which are synonymous (not amino acid-altering). For codon comparisons in which ≥2 nucleotides differ, all possible mutational pathways are considered via an adaptation of the Nei & Gojobori (1986) method (Nelson & Hughes 2015).
	* Nonsyn Sites. The number of nucleotide sites in the current codon which are nonsynonymous, determined using an adaptation of the Nei & Gojobori (1986) method (Nelson & Hughes 2015) which considers codons frequencies. The sum of this value and the value of Syn Sites for a codon sum to 3, as there are 3 nucleotide sites per codon.
	* Syn Sites. The number of nucleotide sites in the current codon which are synonymous, determined using an adaptation of the Nei & Gojobori (1986) method (Nelson & Hughes 2015) which considers codons frequencies. The sum of this value and the value of Nonsyn Sites for a codon sum to 3, as there are 3 nucleotide sites per codon. 
	* Nonsyn Sites (Ref). The number of nucleotide sites in the reference sequence of the current codon which are nonsynonymous., determined using the methods of Nei & Gojobori (1986). The sum of this value and the value of Syn Sites (Ref) will be 3.
	* Syn Sites (Ref). The number of nucleotide sites in the reference sequence of the current codon which are synonymous, determined using the methods of Nei & Gojobori (1986). The sum of this value and the value of Nonsyn Sites (Ref) will be 3.
	* πN. The mean number of nonsynonymous differences per nonsynonymous site in the population of sequences.
	* πS. The mean number of nonsynonymous differences per nonsynonymous site in the population of sequences.
	* Mean dN vs. Ref. Each individual sequence read can be compared to the reference sequence to yield the number of nonsynonymous differences. The Jukes-Cantor correction is applied to account for multiple mutations at the same site, yielding dN. The mean of all such comparisons is the value given in this column.
	* Mean dS vs. Ref. Each individual sequence read can be compared to the reference sequence to yield the number of synonymous differences. The Jukes-Cantor correction is applied to account for multiple mutations at the same site, yielding dS. The mean of all such comparisons is the value given in this column.



## Example

Suppose a blood sample is taken from an animal infected with an known influenza virus inoculum. This inoculum reference sequence is present in the file fluRef.fasta. Suppose Illumina sequencing has been performed on the pooled viral sample present in the blood sample, and Geneious has been used to generate a SNP report, fluPopSNPs.csv. Suppose further that we are interested in examining πN and πS in a protein product that is encoded by nucleotides 112 through 949. After consulting the SNP report to ensure it meets all format requirements, the following command is performed:

...

The sum of the mean number of nonsynonymous (or synonymous) pairwise differences may be divided by the sum of the number of nonsynonymous (or synonymous) sites to obtain πN (or πS).

...

## Additional Scripts

Some additional scripts are included to automate some common tasks when preparing SNPGenie input. These currently are:

* **snpgenie-gbk2gtf.pl**. At the command line, provide this script with one argument: a GenBank (.gbk) file. It will extract the coding element annotations to produce a GTF file ready for SNPGenie. Not working? Let us know, and we'll improve it! Here's an example:

	snpgenie-gbk2gtf.pl my_genbank_file.gbk

* **snpgenie-split_fasta.pl**. At the command line, provide this script with one argument: a FASTA (.fa or .fasta) file containing multiple sequences. This script will create multiple files in the working directory, each containing one of the sequences. Here's an example:

	snpgenie-split_fasta.pl my_multi_fasta_file.fasta

## Citation

When using this software, please refer to and cite:

	Nelson CW, Hughes AL (2015) Within-host nucleotide diversity of virus populations: 
	Insights from next-generation sequencing. Infection, Genetics and Evolution 30:1-7. 
	doi: 10.1016/j.meegid.2014.11.026

## Troubleshooting

* SNPGenie isn't executing? Try preceding the whole command line with "perl" to make sure SNPGenie is being treated as a script. For example:

	perl snpgenie-1.2.pl --sepfiles --minfreq=0.01 --snpreport=mySNPreport.txt --fastafile=myFASTA.fa --gtffile=myGTF.gtf	
* SNPGenie still isn't executing? You might also try making the script executable at the command line, as follows:

	chmod +x snpgenie-1.2.pl	

* Are (end-of-line) newline characters in Unix LF (\n) format? Although SNPGenie was also designed to accept Windows CRLF (\r\n) or Mac CR (\r) formats, these can sometimes introduce problems causing SNPGenie to crash or return all 0 values. Trying changing the newline character to Unix LF using a free program such a [TextWrangler] (http://www.barebones.com/products/textwrangler/).
* Are the FASTA files and/or CLC files tab (\t)-delimited?
* Are the Geneious files comma-separated?
* Do the SNP reports contain all necessary columns? (See sections on Input above.)
* Does the Frequency (CLC) or Variant Frequency (Geneious) SNP report column contain a percentage, not a decimal (e.g., 11.0% rather than 0.11)?
* Was the SNP calling frame correct (i.e., do the codons for a product in the SNP report begin with ATG and end with TAA, TAG, or TGA)?
* Are the product coordinates in the gtf file correct? (You might use a free program such as [MEGA] (http://www.megasoftware.net/) to check that the CDS coordinates begin with ATG and end with TAA, TAG, or TGA.)

## References

* Knapp EW, Irausquin SJ, Friedman R, Hughes AL (2011) PolyAna: analyzing synonymous and nonsynonymous polymorphic sites. Conserv Genet Resour 3:429-431.
* Nei M, Gojobori T (1986) Simple methods for estimating the numbers of synonymous and nonsynonymous nucleotide substitutions. Mol Biol Evol 3:418-26.
* Nelson CW, Hughes AL (2015) Within-host nucleotide diversity of virus populations: Insights from next-generation sequencing. Infection, Genetics and Evolution 30:1-7.

Copyright 2015

# Basic_Mirdeep2_tutorial_on_OSC

## Table of Contents
<a href="#Documentation">Documentation</a></br>
<a href="#Intro to using Mirdeep2">Intro to using Mirdeep2</a></br>
<a href="#Installing Mirdeep2">Installing Mirdeep2</a></br>
<a href="#Cleaning up the Genome for Use in Mirdeep2">Cleaning up the Genome for Use in Mirdeep2</a></br>
<a href="#Indexing the Genome">Indexing the Genome</a></br>
<a href="#Mapping">Mapping</a></br>
<a href="#Running Mirdeep2">Running Mirdeep2</a></br>
<a href="#Blasting Mirdeep2 results">Blasting Mirdeep2 results</a></br>
</br>

## <a name="Documentation">Documentation</a>
Emacs: https://www.gnu.org/software/emacs/manual/html_node/emacs/index.html

OSC: https://www.osc.edu/resources/technical_support/supercomputers/owens

Intro OSC tutorial: https://github.com/Stardust64/OSC_Basic_use_tutorial/blob/main/README.md

Mirdeep2: https://www.mdc-berlin.de/content/mirdeep2-documentation

## <a name="Intro to using Mirdeep2">Intro to using Mirdeep2</a>

### <a name="Installing Mirdeep2">Installing Mirdeep2</a>

To install Mirdeep2 go to working directory where you'd like it and enter the following code:
git clone https://github.com/rajewsky-lab/mirdeep2.git

(optionally you can follow the README instead)

#### Installation Method 1 (easy)

After it is installed go into new "mirdeep2" directory and enter the following code (this will take a few minutes to run):
perl install.pl

To check if installation was successful run "perl install.pl" again
It should say "already installed, nothing to do" next to the necessary packages
To further test installation, exit and re-enter the OSC cluster, and run tutorial script
cd /path/to/mirdeep2/directory
cd tutorial_dir
bash run_tut.sh

Congrats~! Now you can use Mirdeep2! To use mirdeep commands you'll have to enter the path to the command (see later scripts)

#### Installation Method 2 (hard)

Following the README instructions install each dependency individually.
Sources for dependency packages (make sure to grab right version)

Bowtie (bowtie-0.11.3-bin-linux-x86_64.zip): http://bowtie-bio.sourceforge.net/news.shtml#old

ViennaRNA (ViennaRNA-1.8.4.tar.gz): https://www.tbi.univie.ac.at/RNA/#old

Squid (squid-1.9g.tar.gz): http://eddylab.org/software/squid/

Randfold (randfold-2.0.tar.gz): http://bioinformatics.psb.ugent.be/supplementary_data/erbon/nov2003/

PDF-API2 (PDF-API2-0.73.tar.gz): http://search.cpan.org/~areibens/PDF-API2-0.71.001/lib/PDF/API2.pm


### <a name="Cleaning up the Genome for Use in Mirdeep2">Cleaning up the Genome for Use in Mirdeep2</a>

Something to keep in mind
Mirdeep2 is very finicky about certain characters. Make sure to check your reference genome file for odd characters.

If needed get rid of whitespaces, parenthesis, and change non-canonical letters to N
(Can make one script to address this, should take about 5 minutes to run)

Example code for cleaning up bad characters
Perl script to remove whitespaces
```{bash eval=FALSE}
perl -plane 's/\s+.+$//' < /path/to/reference/genome.fasta > /path/to/output/genome.fasta
```

Change confounding word (in this case NODE to node)
```{bash eval=FALSE}
sed -e 's/>NODE/>node/g' /path/to/reference/genome.fasta > /path/to/output/genome.fasta
```

sed to get rid of parenthesis in fasta headings
```{bash eval=FALSE}
sed 's/[)(]//g' /path/to/reference/genome.fasta > /path/to/output/genome.fasta
```
sed to change non-canonical letters to N
```{bash eval=FALSE}
sed -e '/^</! s/[RYSWKMBDHV]/N/g' /path/to/reference/genome.fasta > /path/to/output/genome.fasta
```

Example Script:
```{bash eval=FALSE}
#!/bin/bash
#SBATCH --account=PYS1234
#SBATCH --job-name=salt_index
#SBATCH --time=01:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mail-type=ALL
#SBATCH --mail-user=your.name@email.edu

date

cd /working/directory/

#Saltmarsh Sparrow
#Genome without whitespaces
#perl -plane 's/\s+.+$//' < /path/to/reference/genome.fasta > /path/to/reference/no_whitespace_genome.fasta

#sed to change non-canonical letters to N
#sed -e '/^</! s/[RYSWKMBDHV]/N/g' /path/to/reference/no_whitespace_genome.fasta > /path/to/reference/no_whitespace_and_noncon_letters_genome.fasta

date
```


### <a name="Indexing the Genome">Indexing the Genome</a>

```{bash eval=FALSE}
#!/bin/bash
#SBATCH --account=PYS1234
#SBATCH --job-name=salt_index
#SBATCH --time=05:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mail-type=ALL
#SBATCH --mail-user=your.name@email.edu

date

cd /working/directory

/path/to/mirdeep2/bin/bowtie-build /path/to/reference/no_whitespace_and_noncon_letters_genome.fasta cleaned_genome
ec=`echo $?`
if [ $ec != 0 ];then
        echo An error occured, exit code $ec
fi

date
```

### <a name="Mapping">Mapping</a>

Note: Option -k is for your adapter sequence You may have a different
adapter than the one shown here (ATCTCGTATGCCGTCTTCTGCTTG)

```{bash eval=FALSE}
#!/bin/bash
#SBATCH --account=PYS1234
#SBATCH --job-name=salt_mapper
#SBATCH --time=10:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mail-type=ALL
#SBATCH --mail-user=your.name@email.edu

date

module load mirdeep2/2.0.0.8

cd /working/directory

/path/to/mirdeep2/bin/mapper.pl /path/to/RNA/sequencing/reads/all_mirna.fastq -e -h -j -k ATCTCGTATGCCGTCTTCTGCTTG -l 17 -m -p cleaned_genome -s total_reads_collapsed.fa -t total_reads_collapsed_vs_genome.arf -v -n

date
```

### <a name="Running Mirdeep2">Running Mirdeep2</a>

```{bash eval=FALSE}
#!/bin/bash
#SBATCH --account=PYS1234
#SBATCH --job-name=salt_mirdeep2
#SBATCH --time=15:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --mail-type=ALL
#SBATCH --mail-user=your.name@email.edu

date

module load mirdeep2/2.0.0.8

cd /working/directory/for/mirdeep2/output

/path/to/mirdeep2/bin/miRDeep2.pl /path/to/total_reads_collapsed.fa /path/to/reference/no_whitespace_and_noncon_letters_genome.fasta /path/to/total_reads_collapsed_vs_genome.arf none none none

date
```

### <a name="Blasting Mirdeep2 results">Blasting Mirdeep2 results</a>

Now that you've gotten your miRNA results you can analyze them further and blast them against known miRNAs

Transfer mirdeep2 results to local computer using sftp or cyberduck
Download blast and move blastdbcmd to working directory

Make database for reference precursor sequences (In my case chicken miRNAs)
```{bash eval=FALSE}
~/ncbi-blast-2.9.0+/bin/makeblastdb -dbtype nucl -in Chicken_precursor.fa.fas -parse_seqids
```

blast mirdeep2 output against it
```{bash eval=FALSE}
~/ncbi-blast-2.9.0+/bin/blastn -db Chicken_precursor.fa.fas -query novel_pres_22_06_2020_t_23_21_59_score-50_to_na_copy.fa -out novel_801B_salt_vs_chicken_blastn.txt -word_size 11 -evalue 1000
```

the output txt file is what you start annotation with, go through each one individually and make notes, or make perl script to roughly go through them

Tab file version
```{bash eval=FALSE}
~/ncbi-blast-2.9.0+/bin/blastn -db ~/Desktop/801B_blast/Chicken_precursor.fa.fas -query ~/Desktop/801B_blast/novel_pres_22_06_2020_t_23_21_59_score-50_to_na_copy.fa -evalue 1000 -word_size 11 -out 801B_saltmarsh_vs_chicken_blastn_tab.tsv -outfmt 6
```

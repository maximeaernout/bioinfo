# Script Exam: FASTQ File Analysis

## 0. Data preparation

I start by creating the `data` and `work` folders, giving everyone full permissions on them, and checking the content of the directory.

```bash
mkdir data
mkdir work

chmod 777 data
chmod 777 work

ls -l
```

The `data` and `work` folders are created, and everyone can read, write and execute in them.

---

## Exercise 1: Take inventory of the dataset

### Question 1:

I display the beginning of `popA_ind01.fastq` to see how a read is structured.

```bash
head popA_ind01.fastq
```

```
@SRRpopA_ind01.1 GRC13_0027_FC:4:1:1:7 length=50
TGCAGGCATTGCGACGGGCCATTAATGTCGACCCTCTTGCTGTATTGAAG
+SRRpopA_ind01.1 GRC13_0027_FC:4:1:1:7 length=50
IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII@IIIIIIIIIIIIIII
@SRRpopA_ind01.2 GRC13_0027_FC:4:1:2:14 length=50
TGCATACTATTGAAAGATGAGGGGCAACCAGAATGATCTGCGTTAGCTAT
+SRRpopA_ind01.2 GRC13_0027_FC:4:1:2:14 length=50
IIIIIIIIIIIIIIII@IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII
@SRRpopA_ind01.3 GRC13_0027_FC:4:1:3:21 length=50
TGCAGACAGTGTTATTACGCCCGTCAGGAACCGCTGCGAATCATCTGGGC
```

Each read is made of 4 lines: the header, the sequence, the `+` line (separator) and the quality line.

### Question 2:

Then I count the number of lines of the file.

```bash
wc -l < popA_ind01.fastq
```

The file has 1200 lines.

Since a read is 4 lines, I divide the number of lines by 4 and store the result in a variable.

```bash
n=$(( $(wc -l < popA_ind01.fastq)/4))
echo $n
```

The file contains 300 reads.

### Question 3:

I create the script with `nano`. It prints, for each file given as an argument, the file name, its population and its number of reads.

```bash
nano ex1_count_reads.sh
```

```bash
#!/bin/bash

for file in "$@"; do
    filename=$(basename "$file")
    population="${filename%%_*}"
    lines=$(wc -l < "$file")
    n=$((lines / 4))
    printf "%s\t%s\t%s\n" "$filename" "$population" "$n"
done
```

I check the content of the script with `cat`.

```bash
cat ex1_count_reads.sh
```

The script is saved correctly.


I give execution permission to all users on the script.

```bash
chmod a+x ex1_count_reads.sh
```

The script can now be run with `./ex1_count_reads.sh`.

### Question 4:

I run the script on all the files of population A.

```bash
./ex1_count_reads.sh popA_*.fastq
```

```
popA_ind01.fastq	popA	300
popA_ind02.fastq	popA	250
popA_ind03.fastq	popA	200
```

The script prints one line per file: popA_ind01 has 300 reads, popA_ind02 has 250 reads and popA_ind03 has 200 reads.

I then run the script on all the files of both populations A and B.

```bash
./ex1_count_reads.sh pop*.fastq
```

```
popA_ind01.fastq	popA	300
popA_ind02.fastq	popA	250
popA_ind03.fastq	popA	200
popB_ind01.fastq	popB	400
popB_ind02.fastq	popB	275
popB_ind03.fastq	popB	225
```
The script prints one line per file: popA same as previously and popB_ind01 has 400 reads, popB_ind02 has 275 reads and popB_ind03 has 225 reads.

### Question 5:

I count the total number of reads of population A by concatenating all its files, and I do the same for population B.

```bash
n=$(cat popA_*.fastq | wc -l)
echo $((n/4))
```

Population A has 750 reads in total.

```bash
n=$(cat popB_*.fastq | wc -l)
echo $((n/4))
```

Population B has 900 reads in total.

### Question 6:

I compare the result of my script with the number of lines starting with `@`, counted by `grep`.

```bash
./ex1_count_reads.sh popA_*.fastq
```
```
popA_ind01.fastq  popA  300
```
I then run two different grep lines.
```bash
grep -c "^@" popA_ind01.fastq
grep -c "^@SRR" popA_ind01.fastq
```

`grep "^@"` gives 307 and `grep "^@SRR"` gives 300, whereas there are only 300 reads in the file.

This is explained by the fact that `grep -c "^@"` counts all the lines that start with `@`. 
However, a quality line can also start with `@`.
`grep "^@SRR"` assumes the read identifiers begin with `@SRR` and then correspond to the real number of reads in the file.

---

## Exercise 2: Which individuals look unusable?

### Question 1:

I count the sequences (the 2nd line of each read) of `popA_ind01.fastq` that do not contain any `N`.

```bash
awk 'NR%4==2' popA_ind01.fastq | grep -c -v "N"
```

There are 278 sequences without N.

### Question 2:

I compute the percentage of reads without N for this same file.

```bash
echo $(( 100* $(awk 'NR%4==2' popA_ind01.fastq | grep -c -v "N")/($(wc -l < popA_ind01.fastq)/4)))
```

92% of the reads do not contain any N.

### Question 3:

I create a script that prints, for each file, the number of reads, the number of reads without N and the corresponding percentage.

```bash
nano ex2_quality_N.sh
```

```bash
#!/bin/bash

printf "file\tn_reads\tn_reads_without_N\tpercent\n"

for f in "$@"; do
    lines=$(wc -l < "$f")
    n_reads=$(( lines / 4 ))
    n_without_N=$(awk 'NR%4==2' "$f" | grep -c -v "N")
    percent=$(( 100 * n_without_N / n_reads ))
    printf "%s\t%s\t%s\t%s\n" "$f" "$n_reads" "$n_without_N" "$percent"
done
```

I check the script with `cat` and make it executable.

```bash
cat ex2_quality_N.sh

chmod a+x ex2_quality_N.sh
```

The script is saved and can now be executed.

### Question 4:

I run the script on all the individuals of populations A and B to find a file of bad quality.

```bash
./ex2_quality_N.sh pop*.fastq
```

| file | n_reads | n_reads_without_N | percent |
|---|---|---|---|
| popA_ind01.fastq | 300 | 278 | 92 |
| popA_ind02.fastq | 250 | 217 | 86 |
| popA_ind03.fastq | 200 | 181 | 90 |
| popB_ind01.fastq | 400 | 364 | 91 |
| popB_ind02.fastq | 275 | 252 | 91 |
| popB_ind03.fastq | 225 | 58 | 25 |

The file to remove is `popB_ind03.fastq`, because only 25% of its sequences have no N, which is quite low. All the other files are between 86% and 92%.

---

## Exercise 3: Check the restriction site

### Question 1:

I count the sequences of `popA_ind01.fastq` that start with the motif `TGCA`.

```bash
awk 'NR%4==2' popA_ind01.fastq | grep -c "^TGCA"
```

There are 284 sequences that start with TGCA.

### Question 2: 

I create a script that takes a motif as its first argument, followed by a list of files. For each file, it prints the number and the percentage of reads starting with this motif.

```bash
nano ex3_cutsite.sh
```

```bash
#!/bin/bash

motif="$1"
shift

printf "file\tmotif\tn_reads_with_motif\tpercent"

for f in "$@"; do
    lines=$(wc -l < "$f")
    n_reads=$(( lines / 4 ))
    n_with_motif=$(awk 'NR%4==2' "$f" | grep -c "^$motif")
    percent=$(( 100 * n_with_motif / n_reads ))
    printf "%s\t%s\t%s\t%s\n" "$f" "$motif" "$n_with_motif" "$percent"
done
```

I check the script with `cat` and make it executable.

```bash
cat ex3_cutsite.sh

chmod a+x ex3_cutsite.sh
```

The script is saved and can now be executed.

### Question 3:

I run the script with the motif `TGCA` on all the files.

```bash
./ex3_cutsite.sh TGCA pop*.fastq
```

| file | motif | n_reads_with_motif | percent |
|---|---|---|---|
| popA_ind01.fastq | TGCA | 284 | 94 |
| popA_ind02.fastq | TGCA | 233 | 93 |
| popA_ind03.fastq | TGCA | 194 | 97 |
| popB_ind01.fastq | TGCA | 382 | 95 |
| popB_ind02.fastq | TGCA | 260 | 94 |
| popB_ind03.fastq | TGCA | 52 | 23 |

Most individuals have between 93% and 97% of reads starting with TGCA.

### Question 4:

I compare these results with the quality results of Exercise 2.

Most of the individuals have between 93% and 97% of reads starting with TGCA. However, `popB_ind03.fastq` only has 23% of reads containing the motif. This individual is clearly different from the others and could be considered of lower quality or potentially problematic for the analysis.

Both exercises identify the same individual as problematic. Combining these two quality indicators strengthens the argument that `popB_ind03` should be excluded or investigated before downstream genetic analyses.

---

## Exercise 4: A small test dataset, and archiving

### Question 1:

I create a `subsample` folder to store the subsampled files.

```bash
mkdir subsample
```

The `subsample` folder is created.

### Question 2:

I create a script that takes a number of reads as its first argument, followed by a list of files. For each file, it keeps only the first reads and saves them in the `subsample` folder. Since a read is 4 lines, the number of lines to keep is the number of reads multiplied by 4.

```bash
nano ex4_subsample.sh
```

```bash
#!/bin/bash

n_reads="$1"
shift

n_lines=$(( n_reads * 4 ))

for f in "$@"; do
    head -n "$n_lines" "$f" > "subsample/$f"
done
```

I check the script with `cat`, make it executable and run it to keep 100 reads per file.

```bash
cat ex4_subsample.sh

chmod a+x ex4_subsample.sh
```

I then run it.

```bash
./ex4_subsample.sh 100 ../pop*.fastq
```

The subsampled files are created in the `subsample` folder.

### Question 3:

I check that each file contains exactly 100 reads, which means 400 lines.

```bash
wc -l *.fastq
```

Yes, it matches what I asked for: I obtain 400 lines. It is logical because a read is 4 lines and I asked for 100 reads, so 100 × 4 = 400.

### Question 4:

I archive and compress all the fastq files of the folder into `subsample.tar.gz`.

```bash
tar -czf subsample.tar.gz *.fastq
```

The tar file has just been created, and I verify it with:

```bash
tar -tzf subsample.tar.gz
```

It gives me:

```
popA_ind01.fastq
popA_ind02.fastq
popA_ind03.fastq
popB_ind01.fastq
popB_ind02.fastq
popB_ind03.fastq
```

So in the tar.gz, I do have the fastq files of each individual of populations A and B.

### Question 5:

I compare the size of the folder with the size of the archive. As I'm already in the `subsample` folder, I don't need to put `subsample` back in the command line. Here is what I write:

```bash
du -h
```

It gives me `172K`.

Then, to look at the size of `subsample.tar.gz`, I write:

```bash
ls -lh subsample.tar.gz
```

And it gives me:

```
-rw-r--r-- 1 e22609595 2026LBISM2 19K nov.  16 10:39 subsample.tar.gz
```

The archive (19K) is much lighter than the folder (172K).

### Question 6:

I compress one file with `gzip` (I chose `popA_ind01.fastq`) and compare the sizes before and after.

```bash
gzip -c ../popA_ind01.fastq > popA_ind01.fastq.gz
```

The compressed file `popA_ind01.fastq.gz` is created.

Then, to check the sizes, I run:

```bash
ls -lh ../popA_ind01.fastq popA_ind01.fastq.gz
```

And I obtain:

```
-rw-r--r-- 1 e22609595 2026LBISM2 62K nov.  13 14:53 ../popA_ind01.fastq
-rw-r--r-- 1 e22609595 2026LBISM2 11K nov.  16 10:49 popA_ind01.fastq.gz
```

The `.fastq.gz` is lighter than the `.fastq`.

I also run the following command to get the compression ratio:

```bash
gzip -l popA_ind01.fastq.gz
```

And I obtain:

```
         compressed        uncompressed  ratio uncompressed_name
              10813               63454  83.0% popA_ind01.fastq
```

The compression ratio is 83%, which means that the compressed file only occupies 17% of the original storage. This compression is particularly useful with many individuals (like 500) because it significantly reduces storage requirements. For example, if an individual generates 1 GB of raw fastq data, the total would be 500 GB. If we use compression, then 500 GB × 17% = 85 GB. The total storage goes from 500 GB to 85 GB.

---

## Note:

I stopped there because I didn't have time to finish all exercises.

Maxime Aernout
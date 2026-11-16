#!/bin/bash

#0. Data preparation
mkdir data
mkdir work

chmod 777 data
chmod 777 work

ls -l


#Exercise 1

#Question 1
head popA_ind01.fastq

#Question 2
wc -l < popA_ind01.fastq

n=$(( $(wc -l < popA_ind01.fastq)/4))
echo $n

#Question 3
nano ex1_count_reads.sh

#!/bin/bash

for file in "$@"; do
    filename=$(basename "$file")
    population="${filename%%_*}"
    lines=$(wc -l < "$file")
    n=$((lines / 4))
    printf "%s\t%s\t%s\n" "$filename" "$population" "$n"
done

cat ex1_count_reads.sh

chmod a+x ex1_count_reads.sh

#Question 4
./ex1_count_reads.sh popA_*.fastq

./ex1_count_reads.sh pop*.fastq

#Question 5
n=$(cat popA_*.fastq | wc -l)
echo $((n/4))

n=$(cat popB_*.fastq | wc -l)
echo $((n/4))

#Question 6
./ex1_count_reads.sh popA_*.fastq

grep -c "^@" popA_ind01.fastq
grep -c "^@SRR" popA_ind01.fastq


#Exercise 2

#Question 1
awk 'NR%4==2' popA_ind01.fastq | grep -c -v "N"

#Question 2
echo $(( 100* $(awk 'NR%4==2' popA_ind01.fastq | grep -c -v "N")/($(wc -l < popA_ind01.fastq)/4)))

#Question 3
nano ex2_quality_N.sh

#!/bin/bash

printf "file\tn_reads\tn_reads_without_N\tpercent\n"

for f in "$@"; do
    lines=$(wc -l < "$f")
    n_reads=$(( lines / 4 ))
    n_without_N=$(awk 'NR%4==2' "$f" | grep -c -v "N")
    percent=$(( 100 * n_without_N / n_reads ))
    printf "%s\t%s\t%s\t%s\n" "$f" "$n_reads" "$n_without_N" "$percent"
done

cat ex2_quality_N.sh

chmod a+x ex2_quality_N.sh

#Question 4
./ex2_quality_N.sh pop*.fastq


#Exercise 3

#Question 1
awk 'NR%4==2' popA_ind01.fastq | grep -c "^TGCA"

#Question 2
nano ex3_cutsite.sh

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

cat ex3_cutsite.sh

chmod a+x ex3_cutsite.sh

#Question 3
./ex3_cutsite.sh TGCA pop*.fastq


#Exercise 4

#Question 1
mkdir subsample

#Question 2
nano ex4_subsample.sh

#!/bin/bash

n_reads="$1"
shift

n_lines=$(( n_reads * 4 ))

for f in "$@"; do
    head -n "$n_lines" "$f" > "subsample/$f"
done

cat ex4_subsample.sh

chmod a+x ex4_subsample.sh

./ex4_subsample.sh 100 ../pop*.fastq

#Question 3
wc -l *.fastq

#Question 4
tar -czf subsample.tar.gz *.fastq

tar -tzf subsample.tar.gz

#Question 5
du -h

ls -lh subsample.tar.gz

#Question 6
gzip -c ../popA_ind01.fastq > popA_ind01.fastq.gz

ls -lh ../popA_ind01.fastq popA_ind01.fastq.gz

gzip -l popA_ind01.fastq.gz

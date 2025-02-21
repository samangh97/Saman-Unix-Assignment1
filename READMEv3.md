# Unix Assignment

## Data Inspection

### Attributes of `fang_et_al_genotypes`

```bash
$ wc fang_et_al_genotypes.txt
$ du -h fang_et_al_genotypes.txt
$ awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
```
By inspecting this file, I learned that:

This file has 2783 lines, 2744038 words, and 11051939 characters.

The size of the file is 6.7 Mbytes.

This file has 986 columns.

###Attributes of snp_position.txt
```bash
$ wc snp_position.txt
$ du -h snp_position.txt
$ head -n 1 snp_position.txt
$ awk -F "\t" '{print NF; exit}' snp_position.txt
```
By inspecting this file, I learned:
1.This file has 984 lines, 13198 words, and 82763 characters.

2.The size of this file is 49K.

3.This file has 15 columns.

##Data Processing
###Transposing the Data
First, I transposed fang_et_al_genotypes.txt so that I could join it to snp_position.txt later.
```bash
$ awk -f transpose.awk fang_et_al_genotypes.txt > transposed_genotypes.txt
```
fang_et_al_genotypes.txt had 2 rows more than snp_position.txt, which would make the joining of the columns problematic. Thus, I inserted 2 empty rows in the snp_position.txt file to match the sample IDs in both files.
```bash
$ awk 'NR == 1 {print; print ""; print ""; next} {print}' snp_position.txt > snp_position_mod.txt
```
###Maize Data
In this code, I scanned the 3rd row of transposed_genotypes.txt and only put the columns in a new maize.txt file that contained the values "ZMMIL", "ZMMLR", or "ZMMMR". That way, the maize.txt file only has the maize chromosome information that we're interested in.
```bash
# Define the input and output files
input_file="transposed_genotypes.txt"
output_file="maize.txt"
# Use awk to process the file
awk '
BEGIN { FS=OFS="\t" }  # Set the field separator to tab (adjust if needed)
NR == 3 {              # Process only the 3rd row
    for (i = 1; i <= NF; i++) {  # Loop through all columns
        if ($i == "ZMMLR" || $i == "ZMMIL" || $i == "ZMMMR") {  # Check for the desired values
            cols[i] = 1  # Mark this column to keep
        }
    }
}
{                       # Process all rows
    for (i in cols) {   # Loop through marked columns
        printf "%s%s", $i, (i == length(cols) ? ORS : OFS)  # Print the column values
    }
}
' "$input_file" > "$output_file"
```
Next, I joined columns 1, 3, and 4 (SNP ID, chromosome, and position) from snp_position_mod.txt to transposed_genotypes.txt.
```bash
$ paste -d$'\t' <(cut -f1,3,4 snp_position_mod.txt) maize.txt > maize_joined.txt
```
Next, I sorted the maize_joined.txt file based on its second column, which contained the chromosome data from 1 to 10, and I cut each chromosome's data and put it in its own separate file named maize_sorted_${i} (where i is the chromosome number) for later processing.
```bash
# Sort maize data based on their chromosome value
sort -k2,2n -t$'\t' maize_joined.txt > maize_sorted.txt

# Loop through chromosomes 1 to 10
for i in {1..10}; do
    awk -v value="$i" -F$'\t' '$2 == value' maize_sorted.txt > "maize_sorted_${i}.txt"
done
```
I made a new directory called maize_chromosome to put all the chromosome data.
```bash
mkdir -p maize_chromosome
# Move all maize_sorted_${i} files into the directory
mv maize_sorted_*.txt maize_chromosome/
```
##Sorting Maize Data
###Sort Version 1: Increasing Order with "?" for Missing Data
```bash
# Loop through chromosomes 1 to 10
for i in {1..10}; do
    # Define the input file name
    input_file="maize_chromosome/maize_sorted_${i}.txt"
    
    # Replace empty cells in the 3rd column with "?"
    awk -F$'\t' 'BEGIN {OFS=FS} {$3 = ($3 == "" ? "?" : $3); print}' "$input_file" > "${input_file}.tmp"
    
    # Sort the file numerically by the 3rd column
    sort -k3,3n -t$'\t' "${input_file}.tmp" > "maize_chromosome/maize_sortv1_${i}.txt"
    
    # Remove the temporary file
    rm "${input_file}.tmp"
done
```
###Sort Version 2: Decreasing Order with "-" for Missing Data
```bash
# Loop through chromosomes 1 to 10
for i in {1..10}; do
    # Define the input file name
    input_file="maize_chromosome/maize_sorted_${i}.txt"
    
    # Replace empty cells in the 3rd column with "-"
    awk -F$'\t' 'BEGIN {OFS=FS} {$3 = ($3 == "" ? "-" : $3); print}' "$input_file" > "${input_file}.tmp"
    
    # Sort the file numerically by the 3rd column in decreasing order
    sort -k3,3nr -t$'\t' "${input_file}.tmp" > "maize_chromosome/maize_sortv2_${i}.txt"
    
    # Remove the temporary file
    rm "${input_file}.tmp"
done
```
###Handling Unknown and Multiple Chromosomes
The following script processes maize_sorted.txt and scans the second column for values of "multiple" and "unknown" and puts them in two new files:
```bash
# Define input and output files
input_file="maize_sorted.txt"
unknown_file="maize_unknown_chromosomes.txt"
multiple_file="maize_multiple_chromosomes.txt"

# Clear the output files (if they already exist)
> "$unknown_file"
> "$multiple_file"

# Process the input file
awk -F$'\t' 'BEGIN {OFS=FS} {
    if ($2 == "unknown") {
        print > "'"$unknown_file"'"
    } else if ($2 == "multiple") {
        print > "'"$multiple_file"'"
    }
}' "$input_file"
```
##Teosinte Data
I found the columns containing "ZMPBA", "ZMPIL", and "ZMPJA" and put them in a new file called teosinte.txt.
```bash
# Define the input and output files
input_file="transposed_genotypes.txt"
output_file="teosinte.txt"

# Use awk to process the file
awk '
BEGIN { FS=OFS="\t" }  # Set the field separator to tab (adjust if needed)
NR == 3 {              # Process only the 3rd row
    for (i = 1; i <= NF; i++) {  # Loop through all columns
        if ($i == "ZMPBA" || $i == "ZMPIL" || $i == "ZMPJA") {  # Check for the desired values
            cols[i] = 1  # Mark this column to keep
        }
    }
}
{                       # Process all rows
    for (i in cols) {   # Loop through marked columns
        printf "%s%s", $i, (i == length(cols) ? ORS : OFS)  # Print the column values
    }
}
' "$input_file" > "$output_file"
```
Next, I joined columns 1, 3, and 4 (SNP ID, chromosome, and position) from snp_position_mod.txt to transposed_genotypes.txt.
```bash
$ paste -d$'\t' <(cut -f1,3,4 snp_position_mod.txt) teosinte.txt > teosinte_joined.txt
```
Next, I sorted the teosinte_joined.txt file based on its second column, which contained the chromosome data from 1 to 10, and I cut each chromosome's data and put it in its own separate file named teosinte_sorted_${i} (where i is the chromosome number) for later processing.
```bash
# Sort teosinte data based on their chromosome value
sort -k2,2n -t$'\t' teosinte_joined.txt > teosinte_sorted.txt

# Loop through chromosomes 1 to 10
for i in {1..10}; do
    awk -v value="$i" -F$'\t' '$2 == value' teosinte_sorted.txt > "teosinte_sorted_${i}.txt"
done
```
I made a new directory called teosinte_chromosome to put all the chromosome data.
```bash
mkdir -p teosinte_chromosome
# Move all teosinte_sorted_${i} files into the directory
mv teosinte_sorted_*.txt teosinte_chromosome/
```
##Sorting Teosinte Data
###Sort Version 1: Increasing Order with "?" for Missing Data
```bash 
# Loop through chromosomes 1 to 10
for i in {1..10}; do
    # Define the input file name
    input_file="teosinte_chromosome/teosinte_sorted_${i}.txt"
    
    # Replace empty cells in the 3rd column with "?"
    awk -F$'\t' 'BEGIN {OFS=FS} {$3 = ($3 == "" ? "?" : $3); print}' "$input_file" > "${input_file}.tmp"
    
    # Sort the file numerically by the 3rd column
    sort -k3,3n -t$'\t' "${input_file}.tmp" > "teosinte_chromosome/teosinte_sortv1_${i}.txt"
    
    # Remove the temporary file
    rm "${input_file}.tmp"
done
```
###Sort Version 2: Decreasing Order with "-" for Missing Data
```bash 
# Loop through chromosomes 1 to 10
for i in {1..10}; do
    # Define the input file name
    input_file="teosinte_chromosome/teosinte_sorted_${i}.txt"
    
    # Replace empty cells in the 3rd column with "-"
    awk -F$'\t' 'BEGIN {OFS=FS} {$3 = ($3 == "" ? "-" : $3); print}' "$input_file" > "${input_file}.tmp"
    
    # Sort the file numerically by the 3rd column in decreasing order
    sort -k3,3nr -t$'\t' "${input_file}.tmp" > "teosinte_chromosome/teosinte_sortv2_${i}.txt"
    
    # Remove the temporary file
    rm "${input_file}.tmp"
done
```
###Handling Unknown and Multiple Chromosomes
The following script processes teosinte_sorted.txt and scans the second column for values of "multiple" and "unknown" and puts them in two new files:
```bash
# Define input and output files
input_file="teosinte_sorted.txt"
unknown_file="teosinte_unknown_chromosomes.txt"
multiple_file="teosinte_multiple_chromosomes.txt"

# Clear the output files (if they already exist)
> "$unknown_file"
> "$multiple_file"

# Process the input file
awk -F$'\t' 'BEGIN {OFS=FS} {
    if ($2 == "unknown") {
        print > "'"$unknown_file"'"
    } else if ($2 == "multiple") {
        print > "'"$multiple_file"'"
    }
}' "$input_file"
```

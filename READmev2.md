#Unix Assignment 
## Data inspection
###attributes of fang_et_al_genotypes
$ wc fang_et_al_genotypes.txt
$ du -h fang_et_al_genotypes.txt
$ awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
By inspecting this file I learned that:
1.	This file has 2783 lines, 2744038 words and 11051939 characters.
2.	 The size of the file is 6.7 Mbytes.
3.	This file has 986 columns
###attributes of snp_position.txt
$ wc snp_position.txt
$ du -h snp_position.txt
$ head -n 1 snp_position.txt
$ awk -F "\t" '{print NF; exit}' snp_position.txt

SNP_ID  cdv_marker_id   Chromosome      Position        alt_pos mult_positions  amplicon        cdv_map_feature.name   gene     candidate/random        Genaissance_daa_id      Sequenom_daa_id count_amplicons count_cmf       count_gene
By inspecting this file I learned:
1.	This file has 984 lines, 13198 words, and 82763 characters.
2.	The size of this file is 49K
3.	This file has 15 columns
## Data Processing
First I transposed fang_et_al_genotypes.txt so that I could join it to snp_position.txt later.
Transposing the data:
$ awk -f transpose.awk fang_et_al_genotypes.txt > transposed_genotypes.txt
fang_et_al_genotypes.txt had 2 rows more than snp_position.txt which would make the joining of the columns problematic, thus I inserted 2 empty rows in the snp_position.txt file to match the sample IDs in both files.
   - **Command**:
     ```bash
awk 'NR == 1 {print; print ""; print ""; next} {print}' snp_position.txt > snp_position_mod.txt
     ```
### Maize Data
In this code I scanned the 3rd row of the transposed_genotypes.txt and only put the columns in a new maize.txt file that contained the value “ZMMIL”, “ZMMLR” or “ZMMMR”. That way the maize.txt file only has the maize chromosome information that we’re interested in.

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
Next, I joined columns 1,3 and 4 (SNP ID, chromosome, and position) from snp_position.txt to transposed_genotpes.txt. 
paste -d$'\t' <(cut -f1,3,4 snp_position_mod.txt) maize.txt > maize_joined.txt
Next I sorted the maize_joined.txt file based on its second column which contained the chromosome data from 1 to 10, and I cut each chromosome data and put it in its own separate file named as maize_sorted_${i} ( Here i is the chromosome number) for later processing.
#sort maize and teosinte data based on their chromosome value
sort -k2,2n -t$'\t' maize_joined.txt > maize_sorted.txt

bin/bash

sort -k2,2n -t$'\t' maize_joined.txt > maize_sorted.txt
for i in {1..10}; do awk -v value="$i" -F$'\t' '$2 == value' sorted.txt > "maize_sorted_${i}.txt" done

I made a new directory called maize_chromosome to put all the chromosome data. 
#!/bin/bash

# Create the directory if it doesn't already exist
mkdir -p maize_chromosome

# Move all teosinte_sorted_$[i} files into the directory
mv maize_sorted_*.txt maize_chromosome/

### sorting all the maize_sorted_${i}.txt files based on increasing values of 3rd column and inserting “?” where there is no data in the 3rd column (sort v1)
# Loop through i from 1 to 10
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
### sorting based on decreasing values of the 3rd column and replacing “-“ where there is no data. (sort v2)

#!/bin/bash

# Loop through i from 1 to 10
for i in {1..10}; do
    # Define the input file name
    input_file="maize_chromosome/maize_sorted_${i}.txt"        
        # Replace empty cells in the 3rd column with "-"
        awk -F$'\t' 'BEGIN {OFS=FS} {$3 = ($3 == "" ? "-" : $3); print}' "$input_file" > "${input_file}.tmp"
        
        # Sort the file numerically by the 3rd column in decreasing order
        sort -k3,3nr -t$'\t' "${input_file}.tmp" > "maize_chromosome/maize_sortv2_${i}.txt"
        
        # Remove the temporary file
        rm "${input_file}.tmp"
    else
done
### putting all unknown chromosomes and multiple chromosomes in the same file. 
The following script processes maize_sorted.txt and scans the second column for values of “multiple” and “unknown” and puts them in two new files:
#!/bin/bash

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

###Teosinte Data
I found the columns containing ZMPBA, ZMPIL and ZMPJA and put them in a new file called teosinte.txt.
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

Next, I joined columns 1,3 and 4 (SNP ID, chromosome, and position) from snp_position.txt to transposed_genotpes.txt. 
paste -d$'\t' <(cut -f1,3,4 snp_position_mod.txt) teosinte.txt > teosinte_joined.txt
Next I sorted the teosinte_joined.txt file based on its second column which contained the chromosome data from 1 to 10, and I cut each chromosome data and put it in its own separate file named as teosinte_sorted_${i} ( Here i is the chromosome number) for later processing.
#sort teosinte data based on their chromosome value
sort -k2,2n -t$'\t' teosinte_joined.txt > teosinte_sorted.txt

#!/bin/bash
sort -k2,2n -t$'\t' teosinte_joined.txt > teosinte_sorted.txt
for i in {1..10}; do
    awk -v value="$i" -F$'\t' '$2 == value' teosinte_sorted.txt > "teosinte_sorted_${i}.txt"
done
I made a new directory called teosinte_chromosome to put all the chromosome data. 
#!/bin/bash

# Create the directory if it doesn't already exist
mkdir -p teosinte_chromosome

# Move all teosinte_sorted_$[i} files into the directory
mv teosinte_sorted_*.txt teosinte_chromosome/

### sorting all the teosinte_sorted_${i}.txt files based on increasing values of 3rd column and inserting “?” where there is no data in the 3rd column (sort v1)
# Loop through i from 1 to 10
for i in {1..10}; do
    # Define the input file name
    input_file="teosinte_chromosome/ teosinte _sorted_${i}.txt"
            
        # Replace empty cells in the 3rd column with "?"
        awk -F$'\t' 'BEGIN {OFS=FS} {$3 = ($3 == "" ? "?" : $3); print}' "$input_file" > "${input_file}.tmp"
        
        # Sort the file numerically by the 3rd column
        sort -k3,3n -t$'\t' "${input_file}.tmp" > " teosinte _chromosome/maize_sortv1_${i}.txt"
        
        # Remove the temporary file
        rm "${input_file}.tmp"
done
### sorting based on decreasing values of the 3rd column and replacing “-“ where there is no data. (sort v2)

#!/bin/bash

# Loop through i from 1 to 10
for i in {1..10}; do
    # Define the input file name
    input_file=" teosinte _chromosome/ teosinte _sorted_${i}.txt"        
        # Replace empty cells in the 3rd column with "-"
        awk -F$'\t' 'BEGIN {OFS=FS} {$3 = ($3 == "" ? "-" : $3); print}' "$input_file" > "${input_file}.tmp"
        
        # Sort the file numerically by the 3rd column in decreasing order
        sort -k3,3nr -t$'\t' "${input_file}.tmp" > " teosinte _chromosome/ teosinte _sortv2_${i}.txt"
        
        # Remove the temporary file
        rm "${input_file}.tmp"
    else
done
### putting all unknown chromosomes and multiple chromosomes in the same file. 
The following script processes teosinte_sorted.txt and scans the second column for values of “multiple” and “unknown” and puts them in two new files:
#!/bin/bash

# Define input and output files
input_file="teosinte_sorted.txt"
unknown_file=" teosinte _unknown_chromosomes.txt"
multiple_file=" teosinte _multiple_chromosomes.txt"

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


# Organize Mitofinder and GetOrganelle results by gene
These scripts will use the outputs from Mitofinder or GetOrganelle to create new directories with the mitochondrial sequence data organized by each gene.

## How Run the Scripts
The outputs from the Mitofinder and GetOrganelle analyses using the Hydra job files from this GitHub page will create directories called "Final_Genes" for Mitofinder results and "Final_XXX" for GetOrganelle results. These directories contain a fasta file for each sample with all the mitogenes for that sample. 

To reorganize this data into directories with sequences grouped by each mitochondrial gene, run the following job files in Hydra for either Mitofinder or GetOrganelle results.

The scrips are set up to run within the "_Final_Genes" and "_Final_xxxx" diretctories of Mitofinder and GetOrganelle, respectively. However, if you would like to run the job files from a different directory in Hydra, paste the full path the corresponding results directory after "prodir=" in the job file.

### Mitofinder
```

#!/bin/bash

#THIS SCRTPT WILL PULL ALL SEQUENCES FROM THE "_Final_Genes" OUTPUT FILES FROM MITOFINDER RUNS (HHW JOB FILE)
#TO RUN SCRIPT - File must be within the "Final_Genes" results directory.

# 1. Configuration
prodir="."
outdir="${prodir}/sequences_by_gene"
mtgenes="ATP8 COX1 rrnL ND1 CYTB ND2 ND6 ATP6 ND4 rrnS COX3 COX2 ND4L ND3 ND5 mutS"
summary_file="${outdir}/sequences_by_gene_summary.txt"

# 2. Create directories
mkdir -p "$outdir"
for gene in $mtgenes; do
    mkdir -p "${outdir}/${gene}"
    # Empty the file if it already exists
    > "${outdir}/${gene}/${gene}_all_samples.fasta"
done

# 3. Extraction Logic
echo "Starting extraction from MitoFinder files..."

# Loop through every NT.fasta file in the directory
for file in "${prodir}"/*.fasta; do
    # Check if files actually exist
    [ -e "$file" ] || continue
    echo "Processing: $(basename "$file")"
    
    for gene in $mtgenes; do
        # Use AWK to find the header ending in @GENE and grab all sequence lines until next >
        awk -v g="$gene" '
            BEGIN { RS = ">"; ORS = "" }
            $1 ~ "@"g"$" { print ">" $0 }
        ' "$file" >> "${outdir}/${gene}/${gene}_all_samples.fasta"
    done
done

# 4. Summary Report
# We use { ... } > "$summary_file" to save the output, 
# and "tee" to still show it on your screen.
{
    echo "------------------------------------------"
    echo "Extraction complete. Summary of sequences:"
    for gene in $mtgenes; do
        outfile="${outdir}/${gene}/${gene}_all_samples.fasta"
        if [ -s "$outfile" ]; then
            count=$(grep -c ">" "$outfile")
            echo " - ${gene}: ${count} sequences"
        else
            echo " - ${gene}: 0 sequences found"
        fi
    done
} | tee "$summary_file"

echo "------------------------------------------"
echo "Summary saved to: $summary_file"

```


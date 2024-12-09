
# AgriPlex_Genotype2VCF  
A script to convert AgriPlex genotype data into usable VCF files.  --Minghao 

## Overview  
This repository provides a streamlined process for converting AgriPlex genotype data, typically provided in `.xlsx` format, into standardized VCF files for downstream analysis.

## Input Data  
The genotype data provided by AgriPlex often contains unnecessary formatting, pictures, and multiple sheets. For this script:  
1. Focus on the first sheet, **"Genotypes"**.  
2. Save this sheet as a **CSV file**.  
3. Remove unrelated columns and format the data as follows:  
   - Rename the `allele1` and `allele2` columns to `REF` and `ALT`, respectively.  
   - Ensure the data resembles the example below:  

| Sample_ID     | Customer Marker ID | LSU_diversity_2 | LSU_diversity_11 |  
|---------------|---------------------|------------------|------------------|  
|               | REF                | C                | C                |  
|               | ALT                | T                | T                |  
| 3EB03_P3_1C   |                    | C                | T                |  
| 3EB03_P3_2C   |                    | C                | T                |  
| 3EB03_P3_3C   |                    | C                | C                |  
| 3EB03_P4_1C   |                    | C                | C                |  
| 3EB03_P4_2C   |                    | C                | C                |  
| 3EB03_P5_1C   |                    | C                | C / T            |  

## Conversion Process  
After formatting the data, use the provided script to generate the VCF file.

### Key Features of the Conversion:  
- Genotypes matching the reference allele (`REF`) are denoted as `0`.  
- Genotypes matching the alternative allele (`ALT`) are denoted as `1`.  
- Entries with `FAIL` are represented as a period (`.`).  

### Python Script  

```python
import pandas as pd
import argparse

def convert_to_vcf(input_file, output_file):
    # Load the CSV file
    geno_data = pd.read_csv(input_file)

    # Extract REF and ALT rows and sample data
    ref_row = geno_data.iloc[0, 2:]
    alt_row = geno_data.iloc[1, 2:]
    sample_data = geno_data.iloc[2:, :]

    # Prepare VCF header
    vcf_header = [
        "##fileformat=VCFv4.2",
        "##source=AgriPlexConversion",
        "##FORMAT=<ID=GT,Number=1,Type=String,Description=\"Genotype\">",
        "#CHROM\tPOS\tID\tREF\tALT\tQUAL\tFILTER\tINFO\tFORMAT"
    ]

    # Add sample IDs to the header
    sample_ids = sample_data['Sample_ID'].tolist()
    vcf_header[-1] += "\t" + "\t".join(sample_ids)

    # Generate VCF body
    vcf_rows = []
    chrom = "1"  # Using a placeholder chromosome number
    for idx, marker in enumerate(ref_row.index):
        pos = idx + 1  # Incremental position as placeholder
        ref = ref_row[marker]
        alt = alt_row[marker]

        # Convert genotype calls
        genotypes = []
        for _, row in sample_data.iterrows():
            call = row[marker]
            if call == ref:
                genotypes.append("0/0")
            elif call == alt:
                genotypes.append("1/1")
            elif call == "FAIL":
                genotypes.append("./.")
            else:
                genotypes.append("0/1")  # Assuming heterozygous if different from both REF and ALT

        genotypes_str = "\t".join(genotypes)
        vcf_rows.append(f"{chrom}\t{pos}\t{marker}\t{ref}\t{alt}\t.\tPASS\t.\tGT\t{genotypes_str}")

    # Combine header and body
    vcf_content = "\n".join(vcf_header + vcf_rows)

    # Save the VCF to a file
    with open(output_file, 'w') as vcf_file:
        vcf_file.write(vcf_content)

if __name__ == "__main__":
    # Set up argparse
    parser = argparse.ArgumentParser(description="Convert AgriPlex genotype data to a usable VCF file.")
    parser.add_argument("input_file", help="Path to the cleaned CSV file containing genotype data.")
    parser.add_argument("output_file", help="Path to save the generated VCF file.")
    args = parser.parse_args()

    # Run the conversion
    convert_to_vcf(args.input_file, args.output_file)
```

---

## Usage Instructions

To use this script:

1. Save it as `genotype2vcf.py`.

2. Run the script using the command line:

   ```bash
   python genotype2vcf.py <input_file> <output_file>
   ```

   Replace `<input_file>` with the path to your cleaned CSV file, and `<output_file>` with the desired path for the output VCF file.

### Example

If your input file is located at `/mnt/data/cleaned_UoA_Rice_geno_2023.csv` and you want to save the output VCF file to `/mnt/data/Converted_Rice_Genotypes.vcf`, use the command:

   ```bash
   python genotype2vcf.py /mnt/data/cleaned_UoA_Rice_geno_2023.csv /mnt/data/Converted_Rice_Genotypes.vcf
   ```

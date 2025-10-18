# GDC MMRF-COMMPASS CNV Analysis Toolkit

This repository contains Python scripts designed for the automated analysis of Copy Number Variation (CNV) data from the **MMRF-COMMPASS** project available on the **Genomic Data Commons (GDC)** portal.

The workflow is divided into sequential steps:
1.  **Download:** Programmatic listing and downloading of public `.seg` CNV files from the GDC API.
2.  **Combine:** Merging all individual segment files (`.seg`) into a single master file.
3.  **Classify:** Applying simple or detailed classification thresholds to the segment mean data.
4.  **Report:** Generating frequency reports for the most recurrent CNV segments, including gene annotation (using the Ensembl REST API) and a placeholder for querying population frequencies (e.g., gnomAD/DGV).

## 🚀 Prerequisites

Before running the scripts, ensure you have the following installed:

1.  **Python 3.x**
2.  **Required Libraries:**
    ```bash
    pip install requests pandas
    ```
3.  **GDC Data Transfer Tool (Optional but Recommended):** For the most robust and efficient download (demonstrated in Block 4). The necessary shell commands for installation are included in the script.

## 📦 Project Structure

The workflow is contained within a single script that can be executed block-by-block, typical of a Jupyter or Colab notebook environment.

| File | Description |
| :--- | :--- |
| `cópia_de_cnv_mmrf_commpass.py` | The main Python script containing the entire multi-step workflow. |
| `mmrf_cnv_data/` | (Created upon execution) Directory where all downloaded `.seg` files will be stored. |
| `combined_cnvs.txt` | (Created upon execution) The aggregated master CNV segment file. |
| `cnv_frequency_report_with_genes.txt` | (Created upon execution) The final output report detailing the top recurrent CNVs. |

## ⚙️ Workflow Steps

The original script is broken down into multiple blocks, each corresponding to a major step in the analysis pipeline.

### Step 1: Download CNV Files from GDC (Blocks 1-4)

This section contains four variations for listing and downloading files. **Block 4** is the most robust as it uses the official GDC Data Transfer Tool.

* **Block 1:** Simple listing and sequential download of the first 1000 files using Python `requests`.
* **Block 2:** Handles pagination to list *all* files and downloads them sequentially.
* **Block 3:** Refined version of Block 2, checking if files already exist before downloading.
* **Block 4 (Recommended):** Generates a GDC manifest file (`gdc_manifest_mmrf_cnv.txt`) and uses the `gdc-client` tool for parallel and resumable downloading.

**Key Parameters:**
* `PROJECT_ID = "MMRF-COMMPASS"`
* `DATA_CATEGORY = "Copy Number Variation"`
* `DOWNLOAD_DIR = "mmrf_cnv_data"`

### Step 2: Combine Segment Files (Blocks 5-7)

Individual `.seg` files downloaded from GDC must be combined into a single dataset for analysis.

* **Block 5/6:** Iterates through the files in `mmrf_cnv_data/`, concatenating the contents and ensuring the header is written only once. The output is a tab-separated file (`combined_cnvs.txt`).
* **Block 7:** Estimates the total number of unique samples found in the combined dataset.

### Step 3: Classify CNV Segments (Blocks 8-9)

The **Segment_Mean** value (log2 ratio) in the `.seg` file must be converted into a human-readable CNV type (e.g., Deletion, Gain).

* **Block 8 (Simple Classification):** Applies a basic set of thresholds (e.g., $\text{log}_2\text{Ratio} \le -1.5$ for Homozygous Deletion).
* **Block 9 (Detailed Classification):** Uses refined, more stringent thresholds to categorize CNVs into 6 types (e.g., Homozygous Deletion, 1-Copy Loss, Neutral, 1-Copy Gain, 2+ Copy Gain). This adds the column `CNV_Type_Detalhada` to the dataframe.

### Step 4: Generate Frequency Report and Annotation (Blocks 10-14)

This final section analyzes the recurrence of CNV segments across the cohort.

* **Block 10/11/12/13:** Groups the classified data by genomic segment (`SegmentID`) and CNV type to calculate the recurrence count and frequency (in %). This generates reports like `cnv_frequency_report.txt` and `cnv_frequency_report3.txt`.
* **Block 14 (Expanded Report):**
    * Calculates the top 5 most frequent CNVs.
    * Uses the **Ensembl REST API** to annotate the regions with overlapping **Gene Symbols**.
    * Includes a **Placeholder Function** (`query_cnv_frequency_public_db`) for users to integrate real-world population frequency data (e.g., gnomAD CNV, DGV) if they implement the necessary API query logic.

## 📝 Key Variables and Column Names

| Variable/Column | Description |
| :--- | :--- |
| `Segment_Mean` | The $\text{log}_2\text{Ratio}$ value indicating copy number change. |
| `CNV_Type` | Simple classification (e.g., "Homozygous Deletion"). |
| `CNV_Type_Detalhada` | Detailed classification (e.g., "1-Copy Gain (3 Copies)"). |
| `SegmentID` | Unique identifier created from `Chromosome:Start-End`. |
| `Count` | Number of samples in the cohort showing the specific CNV in that segment. |
| `Frequency_pct` | Percentage of the total cohort segments represented by this specific CNV. |

---

## ⚠️ Important Considerations

1.  **CNV Calling Thresholds:** The classification in Blocks 8 and 9 uses **idealized log2 ratio thresholds** (e.g., $\text{log}_2(1/2) \approx -1$, $\text{log}_2(3/2) \approx 0.58$). For highly accurate clinical CNV calling in tumor samples, these thresholds must be **adjusted based on tumor purity and ploidy**.
2.  **API Rate Limits:** Running the gene annotation in Block 14 for many segments quickly will hit the Ensembl API rate limits. The current script only queries the top 5 for demonstration purposes.
3.  **Placeholder Functions:** The `query_cnv_frequency_public_db` functions are **placeholders** and must be replaced with custom code to connect to external databases (like gnomAD's GraphQL API) to retrieve actual population frequency data.

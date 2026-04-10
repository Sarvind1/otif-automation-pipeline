# OTIF Automation Pipeline

A comprehensive supply chain automation system for calculating and monitoring On-Time In-Full (OTIF) performance metrics, vendor performance, and turnaround time (TAT) analytics. This system automates data ingestion from multiple sources, performs complex transformations, and generates insights for supply chain optimization.

## Features

- **Data Ingestion Pipeline**: Automated ingestion from Redshift databases and Excel/CSV sources
- **OTIF Calculation Engine**: Computes on-time and in-full delivery metrics with customizable business rules
- **TAT (Turnaround Time) Calculator**: Sophisticated turnaround time calculation system supporting:
  - Multi-stage process flow tracking
  - Dynamic expression evaluation
  - Configurable stage dependencies
  - Delay analysis and reporting
- **Vendor & Supplier Analytics**: Performance tracking across vendors, suppliers, and marketplace channels
- **Compliance & Blocker Tracking**: Identifies and tracks compliance issues and process blockers
- **SharePoint Integration**: Automated upload of results to SharePoint for stakeholder access
- **Report Generation**: Multiple export formats (Excel, JSON, CSV) with detailed analytics

## Tech Stack

- **Core**: Python 3.x with pandas, numpy
- **Database**: AWS Redshift (via redshift-connector)
- **Cloud**: AWS S3 (boto3)
- **Integration**: Microsoft SharePoint (msal), xlsxwriter, openpyxl
- **Data Processing**: pandas, numpy
- **Configuration**: Pydantic, JSON
- **UI**: xlwings for Excel automation

## Project Structure

```
otif_autom/
├── app.py                      # Main orchestration pipeline
├── main.py                     # Core OTIF calculation logic
├── ingestion_tables.py         # Redshift data ingestion
├── ingestion_excels.py         # Excel file ingestion
├── sharepoint.py               # SharePoint integration
├── dod.py                      # Day-over-day analysis
├── imports.py                  # Centralized imports
├── requirements.txt            # Python dependencies
├── tat-calculator/             # TAT calculation subsystem
│   ├── tat_calculator.py       # Core TAT engine
│   ├── tat_processor.py        # TAT orchestration
│   ├── run_tat_calculation.py  # TAT runner script
│   ├── stage_calculator.py     # Stage-level TAT calculations
│   ├── delay_calculator.py     # Delay analysis
│   ├── models_config.py        # TAT configuration models
│   ├── stages_config.json      # Stage configuration
│   ├── outputs/                # Generated reports and data
│   └── requirements.txt        # TAT-specific dependencies
├── static/                     # Static reference data
│   ├── default_mappings.xlsx   # Business rule mappings
│   └── asin_static_payment_status.csv
└── outputs/                    # Pipeline output artifacts
```

## Setup Instructions

### Prerequisites

- Python 3.8+
- Access to:
  - AWS Redshift cluster
  - AWS S3 (for credential storage)
  - Microsoft SharePoint instance
  - Excel/CSV data sources

### Installation

1. **Clone the repository** (or extract the archive):
   ```bash
   cd otif_autom
   ```

2. **Create a virtual environment**:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   ```

3. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   cd tat-calculator && pip install -r requirements.txt && cd ..
   ```

4. **Configure credentials**:
   - Copy `.env.example` to `.env`
   - Fill in all required values:
     - Redshift connection parameters
     - AWS credentials
     - SharePoint tenant and client credentials
   
   Alternatively, create a `creds.txt` file with format:
   ```
   REDSHIFT_HOST=your_host
   REDSHIFT_USER=your_user
   REDSHIFT_PASSWORD=your_password
   ...
   ```

5. **Prepare data sources**:
   - Ensure required Excel mapping files are in place (`static/default_mappings.xlsx`)
   - Verify Redshift database connectivity
   - Confirm SharePoint library paths are accessible

## Usage

### Running the Main OTIF Pipeline

```bash
python app.py
```

This will:
1. Load credentials from `.env` or `creds.txt`
2. Ingest data from Redshift tables
3. Ingest and process Excel mapping files
4. Calculate OTIF metrics and KPIs
5. Generate day-over-day analysis
6. Upload results to SharePoint

**Output**: Generates pickle cache files for efficient re-runs, OTIF calculations in Excel format, and logs in `outputs/`

### Running TAT Calculator

```bash
cd tat-calculator
python run_tat_calculation.py
```

Configuration via `stages_config.json`:
```json
{
  "stages": {
    "stage_1": {
      "name": "Purchase Order Creation",
      "actual_timestamp": "po_date",
      "lead_time": 1,
      ...
    }
  }
}
```

**Output**: TAT calculations in JSON and Excel formats in `outputs/tat_results/`, delay analysis in `outputs/delay_results/`

### Example: Processing Custom Data

```python
from app import *

# Load data sources
creds = load_creds('creds.txt')
dfs_tables = ingestion_tables_main(creds)
dfs_excels = ingestion_excels_main(creds)

# Run OTIF calculation
final_df = cal_main(dfs_tables, dfs_excels)

# Export results
final_df.to_excel('otif_results.xlsx', index=False)
```

## Configuration

### Mapping Files

Place Excel mapping files in `static/` directory:
- `default_mappings.xlsx`: Core business rule mappings
- Custom vendor, supplier, and status mappings

### TAT Configuration

Edit `tat-calculator/stages_config.json` to:
- Define process stages
- Set lead times and dependencies
- Configure fallback calculations
- Specify team ownership

### Environment Variables

See `.env.example` for all required and optional variables.

## Output Files

- `outputs/logs/tat_calculation.log`: Detailed execution logs
- `outputs/tat_results/`: JSON files with complete TAT calculations
- `outputs/delay_results/`: Delay analysis and anomalies
- `outputs/excel_exports/`: Excel reports for stakeholders
- `outputs/csv_files/`: Processed data in CSV format
- `*.pkl` cache files: For efficient pipeline re-runs

## Performance Notes

- First run ingests all data from Redshift and processes Excel files (slower)
- Subsequent runs use cached pickle files for faster execution
- Large datasets (millions of rows) may require optimization in SQL queries
- TAT calculation performance scales with number of stages and records

## Troubleshooting

### Redshift Connection Issues
- Verify credentials in `.env` or `creds.txt`
- Check network connectivity to Redshift endpoint
- Ensure Redshift user has SELECT permissions on required tables

### SharePoint Upload Failures
- Verify MSAL credentials and tenant configuration
- Check site URL and library path are correct
- Ensure user has write permissions to target library

### Memory Issues with Large Files
- Reduce batch size in ingestion modules
- Process files in chunks instead of loading entirely
- Increase system RAM or use streaming operations

### Missing Mapping Data
- Verify all Excel files in `static/` directory exist
- Check for typos in sheet names referenced in code
- Regenerate mappings if source data has changed

## Dependencies

### Core
- `pandas >= 1.0`: Data manipulation and analysis
- `numpy >= 1.18`: Numerical computing
- `redshift-connector`: Redshift database access
- `boto3`: AWS S3 operations
- `requests`: HTTP client for APIs

### Integration
- `msal >= 1.0`: Microsoft authentication
- `xlsxwriter`: Excel file generation
- `openpyxl`: Excel file manipulation
- `xlwings`: Excel automation

### TAT Subsystem
- `pydantic`: Data validation and configuration
- `pandas >= 1.0`: Data processing
- Additional dependencies in `tat-calculator/requirements.txt`

## Maintenance

### Regular Tasks
- Monitor log files for errors
- Update data source credentials quarterly
- Review and update mapping files as business rules change
- Archive old output files to manage storage

### Performance Optimization
- Profile slow queries in Redshift ingestion
- Consider incremental instead of full data loads
- Optimize pandas operations for large datasets

## License

[Specify your license here]

## Contact

For questions or issues regarding this pipeline, contact the supply chain analytics team.

---

**Last Updated**: 2026-04-10

# KIŠIB.Hasty-JSON Importer

A web-based tool for converting [Hasty.ai](https://app.hasty.ai) annotation exports into the KIŠIB project database format.

## Overview

This application transforms JSON exports from Hasty.ai (an image annotation platform) into CSV tables compatible with the KIŠIB database. The tool processes annotated image data and generates two standardized tables:
- `3_pictureelement` - Contains pictorial elements (annotations) with bounding boxes and metadata
- `3_pictelattributes` - Contains attributes and classifications for each pictorial element

The importer automatically normalizes terminology, maps vocabulary IDs, and highlights missing data for manual review.

## Inspiration

This app is based on the approach developed in [matanat](https://github.com/matanat)'s [acawai script](https://github.com/matanat/acawai), reimplemented in JavaScript for easier deployment and cross-platform compatibility.

## Features

- **Upload Hasty.ai JSON exports** and process them directly in the browser
- **Automatic vocabulary normalization** - converts Hasty terminology to KIŠIB standards where necessary
- **ID mapping** - automatically links class names and attributes to vocabulary IDs
- **Missing data highlighting** - cells with placeholders are visually marked for review
- **Configurable ID ranges** - set custom start IDs for database records
- **CSV export** - generates semicolon-delimited CSV files ready for database import
- **No server-side processing** - all conversion happens client-side for data security

## Requirements

### To Run the Application

- A modern web browser (Chrome, Firefox, Safari, Edge)
- A local web server (e.g., XAMPP, WAMP, Python's SimpleHTTPServer) or hosting on a web server

### Required Data Files

The application needs three CSV reference files in the same directory as `index.html`:

1. `_4_illustration__202507040911.csv` - Image/seal mapping data
2. `_9_picvocabulary__202507091642.csv` - Vocabulary definitions
3. `_9_picvocabcategory__202507011635.csv` - Vocabulary category definitions

These files are used to map image filenames to seal IDs and to resolve vocabulary terms to database IDs. They should always represent the most updated exports from the KIŠIB database.

## Installation

### Option 1: Local Web Server (Recommended)

1. Clone or download this repository
2. Ensure the three required CSV files are in the same directory as `index.html`
3. Start a local web server in the project directory:
   
   **Using Python:**
   ```bash
   # Python 3
   python -m http.server 8000
   
   # Python 2
   python -m SimpleHTTPServer 8000
   ```
   
   **Using PHP:**
   ```bash
   php -S localhost:8000
   ```
   
   **Using Node.js (with http-server):**
   ```bash
   npx http-server -p 8000
   ```

4. Open your browser and navigate to `http://localhost:8000`

### Option 2: Web Server Deployment

1. Upload `index.html` and the three CSV files to your web server
2. Ensure all files are in the same directory
3. Access the application via your server URL

### Option 3: XAMPP/WAMP

1. Copy the project folder to your XAMPP/WAMP `htdocs` directory
2. Start Apache via the XAMPP/WAMP control panel
3. Navigate to `http://localhost/[your-folder-name]/`

## Usage

### Basic Workflow

1. **Export from Hasty.ai**
   - Export your annotated project as JSON from Hasty.ai
   - Save the JSON file to your local machine

2. **Open the Application**
   - Navigate to the application URL in your browser

3. **Upload JSON File**
   - Click "JSON-Datei auswählen" and select your Hasty export file

4. **Set ID Start Values**
   - `Startwert für pictureelement ID`: Starting ID for picture elements (default: 1)
   - `Startwert für pictelattributes ID`: Starting ID for picture attributes (default: 1)

5. **Process the Data**
   - Click "Verarbeiten" to convert the JSON data
   - Two tables will be generated showing the converted data
   - Cells with missing or placeholder data are highlighted in red

6. **Optional: Correct missing vocabulary normalization**
   - If vocabulary normalization is still missing (marked in red) the code (at the moment) has to be adapted.
   - If changes have been made, redo steps 2 - 5 until no further normalization is required.

7. **Export CSV Files**
   - Click "Export 3_pictureelement" to download the picture elements table
   - Click "Export 3_pictelattributes" to download the attributes table
   - Both files use semicolon (`;`) as delimiter for compatibility with European Excel/database systems

### Data Normalization

The application automatically normalizes terminology from Hasty annotations to match KIŠIB vocabulary:

**Class Name Corrections:**
- `person` → `human being`
- `lion-griffin` → `lion-dragon`
- `unclassified` → `undetermined`
- `vessel_with straw(s)` → `vessel with straw(s)`

**Vocabulary Category Corrections:**
- `rank` → `pictorial rank`
- `personal attributes` → `personal attribute`
- `face orientation` → `orientation` (with specification: `face`)
- `body orientation` → `orientation` (with specification: `body`)

All normalization changes are logged in the `comments` column for transparency.

## Output Format

### Table 1: `3_pictureelement`

| Column | Description |
|--------|-------------|
| `id` | Unique identifier for picture element (format: `pic00000001`) |
| `seal_id` | Reference to seal/image in KIŠIB database |
| `class_name` | Normalized classification (e.g., `human being`, `bull`) |
| `type_class` | Vocabulary ID for the class |
| `hasty_id` | Original annotation ID from Hasty.ai |
| `bbox` | Bounding box coordinates (JSON array) |
| `polygon` | Polygon coordinates if available (JSON array) |
| `mask` | Mask data if available |
| `sequence` | Z-index/layering information |
| `comments` | Notes about normalization changes |

### Table 2: `3_pictelattributes`

| Column | Description |
|--------|-------------|
| `id` | Unique identifier for attribute (format: `picAtt00000001`) |
| `picElID` | Reference to corresponding picture element |
| `vocabCategory` | Attribute category (e.g., `orientation`, `personal attribute`) |
| `vocabCategoryID` | Vocabulary ID for category |
| `vocabAttribute` | Specific attribute value (e.g., `right`, `seated`) |
| `vocabAttributeID` | Vocabulary ID for attribute |
| `vocabSpecification` | Additional specification if applicable |
| `vocabSpecificationID` | Vocabulary ID for specification |
| `hasty_id` | Original annotation ID from Hasty.ai |
| `comments` | Notes about normalization changes |

## Troubleshooting

### File Access Errors
**Problem:** Browser shows "Failed to fetch" errors for CSV files

**Solution:** You must run the application through a web server. Opening `index.html` directly from the file system will cause CORS errors. Use one of the installation methods above.

### Missing Data (Red Cells)
**Problem:** Many cells are highlighted in red with placeholder values

**Solution:** This indicates missing mappings in the reference CSV files:
- Verify that `seal_id` exists in the illustration CSV for each image
- Check that all class names and attributes exist in the vocabulary CSV
- Manually review and correct these entries before database import

### JSON Parsing Errors
**Problem:** "Error processing data" message appears

**Solution:**
- Ensure the JSON file is a valid Hasty.ai export
- Check that the JSON is not corrupted or truncated
- Verify the file is actually JSON format (not CSV or other format)

### Wrong CSV Delimiter
**Problem:** Imported CSV appears in a single column

**Solution:** The application exports with semicolon (`;`) delimiter. When importing:
- In Excel: Use "Data > From Text" and specify semicolon as delimiter
- In database tools: Set delimiter to semicolon in import settings

## Technical Details

- **Technology:** Pure HTML5, CSS3, and vanilla JavaScript
- **No Dependencies:** No external libraries required
- **Browser Compatibility:** Works in all modern browsers (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- **File Processing:** Client-side only - no data is sent to any server
- **CSV Format:** Semicolon-delimited with UTF-8 encoding

## Project Context

This tool is part of the KIŠIB project, which focuses on the documentation and analysis of ancient seal impressions. For more information about the KIŠIB project, please contact the project team.

## Author

Sebastian Hageneuer

## License

GPL-3.0 license

## Contributing

For questions, bug reports, or feature requests, please open an issue in this repository.

---

**Note:** This tool is specifically designed for the KIŠIB project workflow. Adaptation for other projects may require modifications to the vocabulary mappings and output format.

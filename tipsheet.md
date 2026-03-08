# NICAR tip sheet: Google Sheets importing + data prep

## What you'll learn

How to get messy data into Google Sheets from:

- Text files (CSV/TSV and other delimited formats)
- Web tables (live data via `IMPORTHTML`)
- PDFs (native vs scanned, and what to do about each)

Plus a quick starter workflow for cleaning without breaking your source data.

## First things first

Log into your Google account, then open this Google Sheet [bit.ly/ire-sheets-importing](https://bit.ly/ire-sheets-importing) and make a copy. We'll do all of the hands-on work in this document.

### Useful gear

- A plain text editor to inspect raw files (any will do)
- Google account + Drive space
- Optional (but great): [Tabula](https://tabula.technology/), DocumentCloud, [OpenRefine](https://openrefine.org/)

## Data in this session (example files)

- `import_contributions.csv` (campaign contributions, delimited text)
- `warnreport.pdf` (California WARN report)
- U.S. Census 2024 population estimates press release table:
  - https://www.census.gov/newsroom/press-releases/2025/vintage-2024-popest.html

## 1) Importing delimited text (CSV/TSV/custom)

Try not to get in the habit of working with CSVs and other delimited text files by double-clicking them to open in Excel. Excel will usually open the file just fine, but spreadsheet programs can silently "help" you by changing the data in harmful ways.

- ZIP codes can lose leading zeros (`01234 -> 1234`)
- Dates may get auto-converted (and sometimes misread)
- Long IDs might get turned into scientific notation

### Quick recipe: import a CSV correctly

1. In your Google Sheets file, create a new tab named `Raw_Contributions`.
2. Go to `File -> Import -> Upload` and select `import_contributions.csv`.
3. In the import dialog:
   - Import location: `Replace current sheet`
   - Separator type: `Comma` (or whatever delimiter the file uses)
   - Turn off: `Convert text to numbers, dates and formulas`

## 2) Importing an HTML table (live data)

### When to use it

Great for:

- Frequently updated public tables (agency dashboards, "latest" stats)
- Quick one-off scraping without writing code

### Recipe: `IMPORTHTML`

1. Make a new tab named `Raw_Census` (or similar).
2. Click into cell `A1`.
3. Use:

```gs
=IMPORTHTML("https://www.census.gov/newsroom/press-releases/2025/vintage-2024-popest.html","table",1)
```

### What each part means

- `"URL"`: the web page containing the table
- `"table"`: you can also use `"list"` for list-style HTML content
- `1`: which table on the page (first, second, third)

### How to find the right table number

- Open the webpage
- Right click -> Inspect
- Search for `<table>`
- Count tables in the HTML (or trial-and-error by changing `1` to `2`, `3`, etc.)

### Reality check: it will need cleaning

Common issues:

- Extra header rows
- Footnotes mixed into data
- Numbers stored as text
- Missing or merged cells

### Don't lose your data

Google Sheets will periodically re-scrape your data. To keep your data from changing, make a static version by selecting the data, copying it into the clipboard, and then using "Paste special" to paste only "values" into a new tab or spreadsheet.

## 3) Extracting data from PDFs

### Step 1: figure out what kind of PDF you're working with

- Native PDF: text is selectable/searchable (usually easier)
- Image/scanned PDF: text is not selectable (needs OCR)

### Fast workflow (web tools)

1. Open `warnreport.pdf` and identify the table you want.
2. Use [Tabula](https://tabula.technology/) to capture the tables you need.
3. Export to CSV, then import into Sheets (using the same "don't auto-convert" approach).
4. If it's scanned:
   - Run OCR first (DocumentCloud can do this; many tools can)
   - Then extract

### What to watch for

- `1` becomes `I` or `L`
- Columns drift (especially multi-line company names/addresses)
- Summary tables at the bottom get merged into the main table

### Minimum QA for PDF extraction

- Pick 5 rows across the document and compare against the PDF manually.
- Check totals against the PDF's totals (if provided).
- If the table has an "as-of" date, capture it in your sheet.

## 4) Prepping your data

### Keep raw, build clean

A simple tab structure that scales:

- `Raw_Contributions` (untouched import)
- `Clean_Contributions` (your working copy / formulas)
- `Notes` (source URL, date pulled, assumptions, QA checks)

### Quick "what's dirty?" scan

- Pivot table to summarize categories and find typos (for example, `CA` vs `Calif`)

## Key cleaning example: ZIP codes (5-digit vs 9-digit)

### Problem

ZIP codes can appear as:

- `94103`
- `94103-1234`
- `941031234`

### Goal

Create a consistent 5-digit ZIP column for grouping and mapping.

### Recipe: `LEFT()`

1. Insert a new column next to the ZIP column.
2. Name it `zip_clean`.
3. Format the new column as `General` (not `Text`).
4. In the first data row (example uses ZIP in column `H`):

```gs
=LEFT(H2,5)
```

5. Fill down.

## Handy functions for data cleaning

- `LEFT()`, `RIGHT()`, `MID()` - slice strings
- `TRIM()` - remove extra spaces
- `SUBSTITUTE()` - replace characters (great for cleaning `$` and commas)
- `VALUE()` - convert numbers stored as text into numbers
- `DATEVALUE()` / `TO_DATE()` - convert date strings (careful)
- `SPLIT()` - break delimited text inside a cell
- `REGEXEXTRACT()` - pull patterns out of messy strings (power tool)

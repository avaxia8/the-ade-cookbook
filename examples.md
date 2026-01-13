# Code Examples

## Table of Contents
- [1. Parse](#1-parse) - Basic document parsing
- [2. Split a Merged File](#2-split-a-merged-file) - Process from URLs
- [3. Page-by-Page Processing](#3-page-by-page-processing) - Handle multi-page documents
- [4. Table Extraction with Confidence](#4-table-extraction-with-confidence) - Extract tables with confidence scores
- [5. Async Processing for Large Files](#5-async-processing-for-large-files) - Handle large documents asynchronously
- [Common Patterns](#common-patterns) - Reusable code patterns
  - [Error Handling](#error-handling)
  - [Batch Processing](#batch-processing)
  - [Dynamic Schema](#dynamic-schema)

## 1. Parse

```python
import os
from pathlib import Path
from landingai_ade import LandingAIADE

client = LandingAIADE(
    apikey=os.environ.get("VISION_AGENT_API_KEY"),  # This is the default and can be omitted
    # defaults to "production".
    environment="eu",
)

response = client.parse(
    # use document= for local files, document_url= for remote URLs
    document=Path("path/to/file"),
    model="dpt-2-latest", 
    save_to="./output_folder",  # optional: saves as {input_file}_parse_output.json
)
print(response.chunks)
```

## 2. Extract

# Step 2: Extract (if needed)
class Invoice(BaseModel):
    invoice_number: str
    vendor_name: str
    total_amount: float

extraction = client.extract(
    schema=Invoice.model_json_schema(),
    markdown=parse_result.markdown
)

print(extraction.extraction)
# {'invoice_number': 'INV-2025-001', 'vendor_name': 'Acme Corp', 'total_amount': 1250.00}
```

### The Output of the parse can be then extracted or split

## 2. Split a Merged File

```python
# Parse from URL
parse_result = client.parse(
    document_url="https://example.com/document.pdf",  # Remote file
    model="dpt-2-latest",
    save_to="./parsed_output"  # Auto-saves results
)

print(f"Found {len(parse_result.chunks)} chunks")
print(f"Markdown length: {len(parse_result.markdown)} characters")

# Extract if needed
class ReportData(BaseModel):
    title: str
    executive_summary: str
    key_metrics: dict
    recommendations: list[str]

extraction = client.extract(
    schema=ReportData.model_json_schema(),
    markdown=parse_result.markdown
)

print(extraction.extraction['title'])
```

## 3. Page-by-Page Processing

```python
# Parse with page splitting
result = client.parse(
    document=Path("multi_page_doc.pdf"),
    model="dpt-2-latest",
    split="page",
    save_to="./output"  # Saves each page separately
)

print(f"Document has {len(result.splits)} pages")

# Process each page differently
for i, page in enumerate(result.splits):
    if i == 0:
        # First page - extract header info
        header = client.extract(
            schema={"properties": {"title": {"type": "string"}}},
            markdown=page.markdown
        )
    else:
        # Other pages - extract data
        data = client.extract(
            schema={"properties": {"content": {"type": "array"}}},
            markdown=page.markdown
        )
```

## 4. Table Extraction with Confidence

```python
from typing import List, Dict

class FinancialTable(BaseModel):
    headers: List[str]
    rows: List[Dict[str, float]]
    total_revenue: float
    total_expenses: float

# Parse first
parse_result = client.parse(
    document=Path("financial_statement.pdf"),
    model="dpt-2-latest",
    save_to="./output"
)

# Extract with confidence scores
result = client.extract(
    schema=FinancialTable.model_json_schema(),
    markdown=parse_result.markdown,
    include_confidence=True
)

# Check confidence
for field, confidence in result.confidence_scores.items():
    if confidence < 0.8:
        print(f"⚠️ Low confidence for {field}: {confidence:.2f}")

# Access data
data = result.extraction
print(f"Revenue: ${data['total_revenue']:,.2f}")
print(f"Expenses: ${data['total_expenses']:,.2f}")
```

## 5. Async Processing for Large Files

```python
import asyncio
from pathlib import Path

async def process_large_document(file_path: Path):
    """Handle documents > 10MB asynchronously"""
    
    # Start async job
    job = await client.parse_async(
        document=file_path,
        model="dpt-2-latest",
        save_to="./async_output"
    )
    
    print(f"Job started: {job.id}")
    
    # Poll for completion
    while job.status != "completed":
        await asyncio.sleep(2)
        job = await client.get_job_status(job.id)
        print(f"Status: {job.status}")
    
    # Get results
    result = await client.get_job_result(job.id)
    
    # Extract data
    extraction = await client.extract_async(
        schema=YourSchema.model_json_schema(),
        markdown=result.markdown
    )
    
    return extraction.extraction

# Run async
data = asyncio.run(process_large_document(Path("large_file.pdf")))
```

## Common Patterns

### Error Handling

```python
try:
    result = client.extract(document="file.pdf", schema=schema)
except Exception as e:
    print(f"Extraction failed: {e}")
    # Fallback logic
```

### Batch Processing

```python
from pathlib import Path

files = Path("documents").glob("*.pdf")
results = []

for file in files:
    try:
        # Parse each document
        parse_result = client.parse(
            document=file,
            model="dpt-2-latest",
            save_to=f"./batch_output/{file.stem}"
        )
        
        # Extract if needed
        extraction = client.extract(
            schema=InvoiceSchema.model_json_schema(),
            markdown=parse_result.markdown
        )
        results.append({"file": file.name, "data": extraction.extraction})
    except Exception as e:
        results.append({"file": file.name, "error": str(e)})
```

### Dynamic Schema

```python
# Create schema based on document type
def get_schema_for_doc(doc_type: str):
    schemas = {
        "invoice": InvoiceSchema,
        "form": FormSchema,
        "report": ReportSchema
    }
    return schemas[doc_type].model_json_schema()

# Parse document first
parse_result = client.parse(
    document=Path(file_path),
    model="dpt-2-latest",
    save_to="./output"
)

# Detect type and extract
doc_type = detect_document_type(parse_result.markdown)
schema = get_schema_for_doc(doc_type)
result = client.extract(schema=schema, markdown=parse_result.markdown)
```

## Next Steps

- **[Recipes](recipes.md)** - Production-ready patterns
- **[Concepts](concepts.md)** - Understand the internals
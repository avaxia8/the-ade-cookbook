# Production Recipes

## Table of Contents
1. [Invoice Processing](#1-invoice-processing) - Extract vendor details, line items, and totals
2. [Form Extraction with Checkboxes](#2-form-extraction-with-checkboxes) - Handle forms with various input types
3. [Batch Document Processing](#3-batch-document-processing) - Process multiple documents efficiently
4. [Document Classification](#4-document-classification) - Auto-categorize and route documents
5. [Multi-Page Table Extraction](#5-multi-page-table-extraction) - Handle tables spanning pages
6. [Visual Grounding - Save Chunks as Images](#6-visual-grounding---save-chunks-as-images) - Visualize extracted regions
7. [Error Recovery & Validation](#7-error-recovery--validation) - Handle errors gracefully
8. [Production Pipeline](#8-production-pipeline) - Complete pipeline with logging

## 1. Invoice Processing

Extract vendor details, line items, and totals from invoices.

```python
import os
from pathlib import Path
from landingai_ade import LandingAIADE
from pydantic import BaseModel, Field
from typing import List

class LineItem(BaseModel):
    description: str
    quantity: float
    unit_price: float
    total: float

class Invoice(BaseModel):
    invoice_number: str = Field(description="Invoice ID or number")
    vendor_name: str
    invoice_date: str
    line_items: List[LineItem]
    subtotal: float
    tax_amount: float
    total_amount: float

client = LandingAIADE(
    api_key=os.environ.get("VISION_AGENT_API_KEY"),  # This is the default and can be omitted
)

# Parse invoice first
parse_result = client.parse(
    document=Path("invoice.pdf"),
    model="dpt-2-latest",
    save_to="./output"  # Saves as invoice_parse_output.json
)

# Extract structured data
result = client.extract(
    schema=Invoice.model_json_schema(),
    markdown=parse_result.markdown,
    include_confidence=True
)

# Validate totals
data = result.extraction
calculated_total = sum(item['total'] for item in data['line_items'])
if abs(calculated_total - data['subtotal']) > 0.01:
    print("Warning: Line items don't match subtotal")

print(f"Invoice {data['invoice_number']}: ${data['total_amount']:.2f}")
```

## 2. Form Extraction with Checkboxes

Extract data from forms including text fields, checkboxes, and signatures.

```python
class ApplicationForm(BaseModel):
    # Text fields
    full_name: str
    email: str
    phone: str
    
    # Address
    street_address: str
    city: str
    state: str
    zip_code: str
    
    # Checkboxes
    is_citizen: bool = Field(description="Checked if US citizen")
    has_criminal_record: bool
    agrees_to_terms: bool
    
    # Signature
    signature_present: bool = Field(description="True if signature exists")
    signature_date: str

def process_form(file_path: str):
    # Parse form
    parse_result = client.parse(
        document=Path(file_path),
        model="dpt-2-latest",
        save_to="./forms_output"
    )
    
    # Check for form chunks
    form_chunks = [c for c in parse_result.chunks if c.type == "chunkForm"]
    print(f"Found {len(form_chunks)} form sections")
    
    # Extract data
    extraction = client.extract(
        schema=ApplicationForm.model_json_schema(),
        markdown=parse_result.markdown
    )
    
    # Validate
    data = extraction.extraction
    if not data['signature_present']:
        raise ValueError("Form requires signature")
    
    return data
```

## 3. Batch Document Processing

Process multiple documents efficiently.

```python
from pathlib import Path
import json
from concurrent.futures import ThreadPoolExecutor

def process_document(file_path: Path):
    """Process single document"""
    try:
        # Parse first
        parse_result = client.parse(
            document=file_path,
            model="dpt-2-latest",
            save_to=f"./batch_output/{file_path.stem}"
        )
        
        # Then extract
        result = client.extract(
            schema=InvoiceSchema.model_json_schema(),
            markdown=parse_result.markdown
        )
        return {
            'file': file_path.name,
            'status': 'success',
            'data': result.extraction
        }
    except Exception as e:
        return {
            'file': file_path.name,
            'status': 'error',
            'error': str(e)
        }

def batch_process(folder: Path, max_workers: int = 4):
    """Process folder of documents"""
    files = list(folder.glob("*.pdf"))
    print(f"Processing {len(files)} documents...")
    
    results = []
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        results = list(executor.map(process_document, files))
    
    # Save results
    with open("batch_results.json", "w") as f:
        json.dump(results, f, indent=2)
    
    # Summary
    success = sum(1 for r in results if r['status'] == 'success')
    print(f"Processed: {success}/{len(files)} successful")
    
    return results
```

## 4. Document Classification

Automatically categorize and route documents.

```python
from typing import Literal

class DocumentType(BaseModel):
    document_type: Literal["invoice", "receipt", "contract", "form", "report"]
    confidence: float = Field(ge=0, le=1)
    key_indicators: List[str] = Field(description="Why this classification")

def classify_and_route(file_path: str):
    """Classify document and route to appropriate processor"""
    
    # Quick parse for classification
    parse_result = client.parse(
        document=Path(file_path),
        model="dpt-2-fast",
        save_to="./classified"
    )
    
    # Classify
    classification = client.extract(
        schema=DocumentType.model_json_schema(),
        markdown=parse_result.markdown[:1000]  # Use first 1000 chars
    )
    
    doc_type = classification.extraction['document_type']
    confidence = classification.extraction['confidence']
    
    print(f"Document type: {doc_type} (confidence: {confidence:.2f})")
    
    # Route to appropriate processor
    processors = {
        'invoice': process_invoice,
        'receipt': process_receipt,
        'contract': process_contract,
        'form': process_form,
        'report': process_report
    }
    
    if confidence > 0.7:
        return processors[doc_type](file_path)
    else:
        print("Low confidence - needs manual review")
        return None
```

## 5. Multi-Page Table Extraction

Handle tables that span multiple pages.

```python
def extract_multipage_table(pdf_path: str):
    """Extract tables across pages"""
    
    # Parse with page splits
    result = client.parse(
        document=Path(pdf_path),
        model="dpt-2-latest",
        split="page",
        save_to="./multipage_output"
    )
    
    all_rows = []
    table_started = False
    
    for page_num, page in enumerate(result.splits):
        # Find tables on this page
        table_chunks = [c for c in page.chunks if c.type == 'chunkTable']
        
        if table_chunks:
            # Extract table data
            table_data = client.extract(
                schema={
                    "properties": {
                        "headers": {"type": "array", "items": {"type": "string"}},
                        "rows": {"type": "array", "items": {"type": "object"}}
                    }
                },
                markdown=table_chunks[0].markdown
            )
            
            rows = table_data.extraction.get('rows', [])
            
            # Skip header row if continuing table
            if table_started and rows:
                rows = rows[1:]  # Skip header
            
            all_rows.extend(rows)
            table_started = True
        else:
            table_started = False
    
    return all_rows
```

## 6. Visual Grounding - Save Chunks as Images

Visualize and save extracted regions from documents.

```python
from PIL import Image, ImageDraw
import pymupdf
from pathlib import Path
from datetime import datetime

def save_chunks_as_images(parse_response, document_path, output_base_dir="groundings"):
    """Save each parsed chunk as a separate image file."""
    
    # Create timestamped output directory
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    document_name = Path(document_path).stem
    output_dir = Path(output_base_dir) / f"{document_name}_{timestamp}"
    output_dir.mkdir(parents=True, exist_ok=True)
    
    if document_path.suffix.lower() == '.pdf':
        pdf = pymupdf.open(document_path)
        
        for page_num in range(len(pdf)):
            page = pdf[page_num]
            pix = page.get_pixmap(matrix=pymupdf.Matrix(2, 2))
            img = Image.frombytes("RGB", [pix.width, pix.height], pix.samples)
            
            # Save chunks for this page
            img_width, img_height = img.size
            page_dir = output_dir / f"page_{page_num}"
            page_dir.mkdir(exist_ok=True)
            
            for chunk in parse_response.chunks:
                if chunk.grounding.page != page_num:
                    continue
                
                box = chunk.grounding.box
                
                # Convert normalized coords to pixels
                x1 = int(box.left * img_width)
                y1 = int(box.top * img_height)
                x2 = int(box.right * img_width)
                y2 = int(box.bottom * img_height)
                
                # Crop and save
                chunk_img = img.crop((x1, y1, x2, y2))
                filename = f"{chunk.type}_{chunk.id[:8]}.png"
                chunk_img.save(page_dir / filename)
        
        pdf.close()
    
    print(f"Chunks saved to: {output_dir}")
    return output_dir
```

## 7. Error Recovery & Validation

Handle extraction errors gracefully.

```python
class ExtractionValidator:
    @staticmethod
    def validate_extraction(data: dict, schema_class: BaseModel) -> List[str]:
        """Validate extracted data"""
        errors = []
        
        # Check required fields
        for field, field_info in schema_class.model_fields.items():
            if field_info.is_required and field not in data:
                errors.append(f"Missing required field: {field}")
        
        # Validate data types
        try:
            validated = schema_class(**data)
        except Exception as e:
            errors.append(f"Validation error: {str(e)}")
        
        return errors

def extract_with_retry(document: str, schema, max_retries: int = 3):
    """Extract with automatic retry on failure"""
    
    # Parse once
    parse_result = client.parse(
        document=Path(document),
        model="dpt-2-latest",
        save_to="./retry_output"
    )
    
    for attempt in range(max_retries):
        try:
            # Try extraction
            result = client.extract(
                schema=schema,
                markdown=parse_result.markdown,
                include_confidence=True
            )
            
            # Validate
            errors = ExtractionValidator.validate_extraction(
                result.extraction,
                schema
            )
            
            if not errors:
                return result
            
            print(f"Attempt {attempt + 1} validation errors: {errors}")
            
        except Exception as e:
            print(f"Attempt {attempt + 1} failed: {e}")
            if attempt == max_retries - 1:
                raise
    
    return None
```

## 8. Production Pipeline

Complete production pipeline with logging and monitoring.

```python
import logging
from datetime import datetime
import hashlib

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class DocumentPipeline:
    def __init__(self, client: LandingAIADE):
        self.client = client
        self.stats = {"processed": 0, "success": 0, "failed": 0}
    
    def process(self, file_path: Path) -> dict:
        """Full pipeline with logging"""
        start_time = datetime.now()
        doc_hash = self._get_file_hash(file_path)
        
        logger.info(f"Processing: {file_path.name} (hash: {doc_hash[:8]})")
        
        try:
            # Parse
            parse_result = self.client.parse(
                document=file_path,
                save_to=f"./output/{doc_hash}"
            )
            logger.info(f"Parsed: {len(parse_result.chunks)} chunks")
            
            # Classify
            doc_type = self._classify_document(parse_result.markdown)
            logger.info(f"Classified as: {doc_type}")
            
            # Extract
            schema = self._get_schema_for_type(doc_type)
            extraction = self.client.extract(
                schema=schema,
                markdown=parse_result.markdown,
                include_confidence=True
            )
            
            # Validate
            if self._validate_extraction(extraction):
                self.stats["success"] += 1
                result = {
                    "status": "success",
                    "doc_hash": doc_hash,
                    "doc_type": doc_type,
                    "data": extraction.extraction,
                    "confidence": extraction.confidence_scores,
                    "processing_time": (datetime.now() - start_time).total_seconds()
                }
            else:
                raise ValueError("Validation failed")
            
        except Exception as e:
            logger.error(f"Failed: {e}")
            self.stats["failed"] += 1
            result = {
                "status": "error",
                "doc_hash": doc_hash,
                "error": str(e),
                "processing_time": (datetime.now() - start_time).total_seconds()
            }
        
        self.stats["processed"] += 1
        logger.info(f"Stats: {self.stats}")
        
        return result
    
    def _get_file_hash(self, file_path: Path) -> str:
        """Generate file hash for deduplication"""
        with open(file_path, "rb") as f:
            return hashlib.md5(f.read()).hexdigest()
    
    def _classify_document(self, markdown: str) -> str:
        # Classification logic
        return "invoice"
    
    def _get_schema_for_type(self, doc_type: str):
        # Schema selection logic
        return InvoiceSchema.model_json_schema()
    
    def _validate_extraction(self, extraction) -> bool:
        # Validation logic
        return True
```

## Tips

- **Use confidence scores** to identify uncertain extractions
- **Save parse results** with `save_to` for debugging
- **Process in parallel** for better throughput
- **Validate extractions** before downstream processing
- **Log everything** in production environments

## Next Steps

- **[Concepts](concepts.md)** - Understand the internals
- **[Examples](examples.md)** - Basic code patterns
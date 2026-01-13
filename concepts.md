# Essential Concepts Guide

## Table of Contents
- [Understanding Parse Output](#understanding-parse-output)
- [Visual Grounding](#visual-grounding)
- [Understanding Splits](#understanding-splits---document-organizer-magic)
- [Understanding Extraction Output](#understanding-extraction-output---your-data-detectives-report-)
- [Extraction Schemas Explained](#extraction-schemas-explained)
  - [Pydantic vs JSON Schema](#pydantic-vs-json-schema-the-easy-way)
  - [JSON Schema for Dummies](#json-schema-for-dummies)

## Understanding Parse Output

ParseResponse is the master container returned when you run client.parse(). 

It transforms a chaotic pile of pixels into a structured system where every item is categorized (**chunks**), transcribed (**markdown**), and tracked to its exact original location (**grounding**).

### **The Anatomy of a Parse Response**

| Field | Description | Use Case | Example |
| :--- | :--- | :--- | :--- |
| **`markdown`** | Complete document text with chunk anchors | **Feed to LLMs** for Q&A about the document | `"# Invoice\n<a id='chunk_1'></a>Total: $500"` |
| **`chunks`** | List of document elements with their own markdown, type, ID, and location | **Process specific content types** - "Find all tables" or "Extract logos" | `[{id: "chunk_1", type: "table", markdown: "...", grounding: {...}}]` |
| **`splits`** | Pages or sections with their chunks organized | **Page-level processing** - "Get content from page 3" | `[{class: "page", identifier: "1", pages: [1], chunks: ["chunk_1"]}]` |
| **`metadata`** | Processing information and usage stats | **Billing & monitoring** - Track credits and performance | `{filename: "invoice.pdf", credit_usage: 3.0, duration_ms: 5979}` |

### Complete JSON Structure

```json
{
  "markdown": "<string>",  // Full document with chunk anchors: <a id='chunk_id'></a>
  "chunks": [
    {
      "markdown": "<string>",     // This chunk's content
      "type": "<string>",          // text, table, figure, etc.
      "id": "<string>",            // Unique ID matching anchors in markdown
      "grounding": {
        "box": {
          "left": 123,             // Pixel coordinates (not normalized)
          "top": 123,
          "right": 456,
          "bottom": 789
        },
        "page": 1                  // Page number (1-indexed)
      }
    }
  ],
  "splits": [
    {
      "class": "<string>",         // Usually "page"
      "identifier": "<string>",    // Page or section ID
      "pages": [1, 2],            // Which pages are included
      "markdown": "<string>",      // Combined markdown for this split
      "chunks": ["chunk_1", "chunk_2"]  // IDs of chunks in this split
    }
  ],
  "metadata": {
    "filename": "invoice.pdf",
    "org_id": "<string>",
    "page_count": 3,
    "duration_ms": 5979,          // Processing time
    "credit_usage": 3.0,           // Credits consumed
    "version": "1.0.0"
  }
}
```

### Quick Access Example

```python
# Parse the document
response = client.parse("invoice.pdf")

# Access the full markdown with chunk anchors
print(response.markdown)  # "# Invoice\n<a id='chunk_1'></a>Total: $500..."

# Work with individual chunks
for chunk in response.chunks:
    print(f"Type: {chunk.type}, ID: {chunk.id}")
    print(f"Content: {chunk.markdown}")
    print(f"Location: Page {chunk.grounding.page}, Box: {chunk.grounding.box}")

# Process by page
for split in response.splits:
    if split.class == "page" and "1" in split.identifier:
        print(f"Page 1 has {len(split.chunks)} chunks")
        print(f"Page 1 content: {split.markdown}")

# Check usage
print(f"Used {response.metadata.credit_usage} credits")
print(f"Processed {response.metadata.page_count} pages in {response.metadata.duration_ms}ms")
```


## Visual Grounding

```
┌─────────────────────────────┐
│         INVOICE             │
│  ┌──────────────────┐       │
│  │ Invoice #: 001   │←──────┼── Chunk: {type: "text", 
│  └──────────────────┘       │           grounding: {
│                             │             left: 0.1,
│  ┌──────────────────┐       │             top: 0.2,
│  │ Item  │ Price    │←──────┼──           right: 0.6,
│  │────────────────  │       │             bottom: 0.3
│  │ Widget│ $100     │       │           }}
│  └──────────────────┘       │
└─────────────────────────────┘
```

## Understanding Splits - Document Organizer Magic

Think of **splits** as your personal filing assistant who can take a messy stack of mixed documents and automatically sort them into organized folders. 

**The Magic:** You upload ONE file with bank statements and pay stubs all mixed together, and ADE automatically separates them by type and even identifies which pay stub is which!

### Real-World Example: Mixed Financial Documents

Imagine you scanned 3 pages:
- Page 1: Bank statement for January
- Page 2: Pay stub from January 15
- Page 3: Pay stub from January 30

### Visual: How Splits Work

```
Your Uploaded PDF:                 ADE Splits It Into:
┌─────────────────┐                ┌──────────────────┐
│ Page 1:         │                │ 📁 Bank Statement│
│ Bank Statement  │   ─────►       │    - January     │
│─────────────────│                └──────────────────┘
│ Page 2:         │                ┌──────────────────┐
│ Pay Stub Jan 15 │   ─────►       │ 📁 Pay Stub      │
│─────────────────│                │    - Jan 15      │
│ Page 3:         │                └──────────────────┘
│ Pay Stub Jan 30 │   ─────►       ┌──────────────────┐
└─────────────────┘                │ 📁 Pay Stub      │
                                   │    - Jan 30      │
                                   └──────────────────┘
```

Here's what ADE returns:

```json
{
  "splits": [
    {
      "classification": "Bank Statement",     // What type of doc
      "identifier": null,                      // No specific ID needed
      "pages": [0],                           // Found on page 1 (0-indexed)
      "markdowns": ["Bank Statement\nAccount Number: 1234567890..."]
    },
    {
      "classification": "Pay Stub",           // Document type
      "identifier": "2025-01-15",            // Specific date found!
      "pages": [1],                          // Page 2
      "markdowns": ["Pay Stub\nEmployee: John Smith\nPay Date: January 15..."]
    },
    {
      "classification": "Pay Stub",          // Another pay stub
      "identifier": "2025-01-30",           // Different date
      "pages": [2],                         // Page 3
      "markdowns": ["Pay Stub\nEmployee: John Smith\nPay Date: January 30..."]
    }
  ]
}
```

## Understanding Extraction Output - Your Data Detective's Report 🔍

Think of extraction as hiring a **data detective** who not only finds the information you asked for but also shows you exactly where they found it, like leaving breadcrumbs back to the evidence.

### The Three-Part Detective Report

When you extract data, you get THREE things back:

```
1. 🎯 extraction       = The treasure (clean data you wanted)
2. 🔍 extraction_metadata = The evidence trail (where each piece came from)  
3. 📋 metadata        = The receipt (processing details)
```

### Real Example: Extracting from a Pay Stub

Here's what your data detective brings back:

```json
{
  // PART 1: The Treasure - Clean, ready-to-use data
  "extraction": {
    "employee_name": "MICHAEL D BRYAN",
    "employee_ssn": "555-50-1234",
    "gross_pay": 6000.00
  },
  
  // PART 2: The Evidence Trail - Shows their work!
  "extraction_metadata": {
    "employee_name": {
      "value": "MICHAEL D BRYAN",
      "references": ["72ba3cca-01e5-407b-9fc4-81f54f9f0c51"]  // Found in this chunk!
    },
    "employee_ssn": {
      "value": "555-50-1234",
      "references": ["a3f5d8c9-2b4e-4a1c-8f7e-9d6c5b4a3e2f"]  // Different chunk
    },
    "gross_pay": {
      "value": "$6,000.00",
      "references": ["5b8865b9-1a81-46df-bcf7-0bdbed9130dc"]  // Yet another chunk
    }
  },
  
  // PART 3: The Receipt - Processing details
  "metadata": {
    "duration_ms": 1523,              // Took 1.5 seconds
    "credit_usage": 1.0,               // Cost 1 credit
    "schema_violation_error": null    // No errors! 🎉
  }
}
```

### Visual: How References Connect Everything

```
Your Document                    Extraction Output
┌──────────────────┐            ┌────────────────────┐
│ Chunk 72ba3cca:  │  ────►     │ employee_name:     │
│ "MICHAEL D BRYAN"│            │ "MICHAEL D BRYAN"  │
├──────────────────┤            │ ↓ references:      │
│ Chunk a3f5d8c9:  │  ────►     │ [72ba3cca...]     │
│ "SSN: 555-50..." │            ├────────────────────┤
├──────────────────┤            │ employee_ssn:      │
│ Chunk 5b8865b9:  │  ────►     │ "555-50-1234"     │
│ "Gross: $6,000"  │            │ ↓ references:      │
└──────────────────┘            │ [a3f5d8c9...]     │
                                └────────────────────┘
```

### Why This Is Awesome

1. **Verification**: You can trace every extracted value back to its source
2. **Debugging**: If something looks wrong, check the referenced chunk
3. **Compliance**: Perfect audit trail for regulated industries
4. **Confidence**: Multiple references mean the value appeared multiple times

### Working with Extraction Output

```python
# Extract data
result = client.extract(
    schema={"properties": {"employee_name": {"type": "string"}}},
    markdown=response.markdown
)

# Get the clean data
name = result.extraction["employee_name"]  # "MICHAEL D BRYAN"

# Check where it came from
metadata = result.extraction_metadata["employee_name"]
source_chunk_id = metadata["references"][0]  # "72ba3cca..."

# Find the original chunk
original_chunk = next(c for c in response.chunks if c.id == source_chunk_id)
print(f"Found '{name}' in chunk type: {original_chunk.type}")

# Check for errors
if result.metadata["schema_violation_error"]:
    print("Schema problem:", result.metadata["schema_violation_error"])
```

### Quick Reference

| Component | What You Get | Use It For |
|-----------|-------------|------------|
| **extraction** | Clean, structured data | Your application logic |
| **extraction_metadata** | Source references for each field | Verification & debugging |
| **metadata** | Processing stats | Monitoring & billing |
| **references** | Chunk IDs where data was found | Trace back to source |
| **schema_violation_error** | Error messages if schema failed | Error handling |

**Pro Tip:** Always check `schema_violation_error` first! If it's not null, something went wrong with your schema. The detective couldn't complete their mission! 🕵️

## Extraction Schemas Explained

### Pydantic vs JSON Schema: The Easy Way

Think of it this way:
- **Pydantic** = The friendly Python translator 🐍
- **JSON Schema** = The universal language everyone speaks 🌍

| Aspect | Pydantic (Python-Friendly) | JSON Schema (Universal) |
|--------|---------------------------|------------------------|
| **What it looks like** | Clean Python classes with type hints | Nested dictionaries with strings |
| **Writing experience** | `name: str` | `"name": {"type": "string"}` |
| **IDE Support** | Auto-complete, type checking ✨ | Just a dictionary 📝 |
| **Validation** | Automatic with helpful errors | Manual validation needed |
| **Best for** | Python developers who want comfort | Working across languages/tools |

**Quick Example - Same Thing, Two Ways:**

```python
# Pydantic Way (write this in Python)
class Invoice(BaseModel):
    invoice_number: str
    total: float
    
# JSON Schema Way (what actually gets sent to ADE)
{
    "properties": {
        "invoice_number": {"type": "string"},
        "total": {"type": "number"}
    }
}
```

**Pro Tip:** Use Pydantic in Python, then call `.model_json_schema()` to convert it. Best of both worlds! 🎉

### JSON Schema for Dummies

Think of a **Schema** as a "shopping list" you give to the AI.

If you send the AI a messy document without instructions, it doesn't know what you care about. A schema tells the AI: *"Ignore the noise. Just find these specific things, and give them to me in this specific format."*

#### 1. The Basic Skeleton

Every schema starts the same way. It's a container (an "object") that holds the list of things you want ("properties").

```json
{
  "type": "object",          // This says "I want a bundle of data"
  "properties": {            // "Here is the list of things to find"
                            // ... Your fields go here ...
  }
}
```

#### 2. Defining Your Fields (The "Shopping Items")

For every piece of data you want, you need to tell the AI three things:

1. **Name:** What you want to call it (e.g., `patient_name` instead of just `name`)
2. **Type:** What kind of data is it? (Text? A number? A list?)
3. **Description:** A hint to help the AI find it (e.g., "The total cost in USD, excluding tax")

**The Data Types (Flavors of Data):**
- **`string`**: Plain text (names, descriptions, codes)
- **`number`**: Money or math stuff (decimals allowed)
- **`integer`**: Whole numbers only (no decimals)
- **`boolean`**: True/False (e.g., "Is this checked?")
- **`array`**: A list of things (like line items on a bill)
- **`object`**: A group of related things (like an address)

**A Simple Example:**

```json
{
  "type": "object",
  "properties": {
    "patient_name": {                 // 1. The Name
      "type": "string",               // 2. The Type (Text)
      "description": "Patient name"   // 3. The Hint
    },
    "copay": {
      "type": "number",               // Money = Number
      "description": "Amount paid"
    }
  },
  "required": ["patient_name"]        // "You MUST find this"
}
```

#### 3. Cool Tricks (Advanced Features Simplified)

**Multiple Choice (`enum`)** - Force the AI to pick from your list:
```json
"account_type": {
  "type": "string",
  "enum": ["Checking", "Savings"] 
}
```

**Lists (`array`)** - Grab multiple similar items:
```json
"charges": {
  "type": "array",
  "items": {"type": "string"}  // "Get me a list of strings"
}
```

**Grouping (`object`)** - Keep related data together:
```json
"invoice": {
  "type": "object", 
  "properties": {
     "number": {"type": "string"},
     "total": {"type": "number"}
  }
}
```

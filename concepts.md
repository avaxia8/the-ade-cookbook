# Visual Concepts Guide

## Table of Contents
- [Understanding Parse Output](#understanding-parse-output)
- [Visual Grounding](#visual-grounding)
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

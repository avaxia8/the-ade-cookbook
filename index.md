# The ADE Cookbook
A Developers' guide to LandingAI's Agentic Document Extraction(ADE)

Extract structured data with [99.16% precision](https://landing.ai/blog/superhuman-on-docvqa-without-images-in-qa-agentic-document-extraction).

## Get Started
First, you need a LandingAI API key. [Acquire an API key here.](https://docs.landing.ai/ade/agentic-api-key)

Then choose your favourite mode.

<table>
  <tr>
    <td width="20%" align="center">
      <h3>🎮 Playground</h3>
      <p>Try ADE in your browser.</p>
      <a href="https://va.landing.ai"><strong>Open Playground →</strong></a>
    </td>
    <td width="20%" align="center">
      <h3>🐍 Python</h3>
      <code>pip install landingai-ade</code><br><br>
      <a href="https://github.com/landing-ai/ade-python"><strong>Python Library →</strong></a>
    </td>
    <td width="20%" align="center">
      <h3> Typescript</h3>
      <code>npm install landingai-ade</code><br><br>
      <a href="https://github.com/landing-ai/ade-typescript"><strong>Typescript Library →</strong></a>
    </td>
    <td width="20%" align="center">
      <h3>🔌 REST API</h3>
      <p>Direct API access.</p>
      <br>
      <a href="https://docs.landing.ai/api-reference/tools/ade-parse"><strong>API Docs →</strong></a>
    </td>
  </tr>
</table>

## Document Parsing Quick Start
Every workflow starts with document parsing.

```python
from pathlib import Path
from landingai_ade import LandingAIADE

client = LandingAIADE()  # Instantiate the class
response = client.parse(
    document=Path("invoice.pdf"),
    model="dpt-2-latest",
    save_to="./output_folder",
)

print(f"Found {len(response.chunks)} chunks")
print(response.markdown[:200])
```
## Workflow Decision Tree
The parsing output can then be split, extracted or classified.

![The ADE Decision Tree](/images/ade_decision_tree.png)
## Learn More

### [Code Examples](examples.md)
1. **[Parse](examples.md#1-basic-parse-and-extract)** - Parse documents and access output
2. **[Extract](examples.md#1-basic-parse-and-extract)** - Extract structured data with schemas
3. **[Split](examples.md#3-page-by-page-processing)** - Process documents page by page
4. **[Save Visual Grounding](examples.md#1-basic-parse-and-extract)** - Auto-save parse results
5. **[Parallel Processing](examples.md#5-async-processing-for-large-files)** - Async handling for large files



### [Concepts](concepts.md)
- [Parse Output Guide](concepts.md#understanding-parse-output) - Understand the parse response
- [Chunk Types](concepts.md#chunk-types) - Available content types
- [Visual Grounding](concepts.md#visual-grounding) - Location tracking
- [Extraction Schema](concepts.md#extraction-schemas-explained) - Schemas as shopping lists for AI


### [Recipes](recipes.md) 
- [Invoice Processing](recipes.md#1-invoice-processing) - Complete invoice extraction
- [Form Extraction](recipes.md#2-form-extraction-with-checkboxes) - Handle complex forms
- [Batch Processing](recipes.md#3-batch-document-processing) - Process multiple files
- [Document Classification](recipes.md#4-document-classification) - Auto-categorize documents
- [Production Pipeline](recipes.md#8-production-pipeline) - Enterprise-ready setup

### [Projects](https://github.com/landing-ai/ade-helper-scripts)

#### RAG & LLM Integration
- **[RAG Document Parser](https://github.com/landing-ai/ade-helper-scripts/tree/main/workflows/retrieval_augmented_generation/document_parser_for_rag_applications)** - Chunk documents for RAG
- **[Local RAG with ChromaDB](https://github.com/landing-ai/ade-helper-scripts/tree/main/workflows/retrieval_augmented_generation/ade_local_rag_with_openai_and_chromadb)** - Vector database integration

#### Industry Solutions
- **[Invoice Processing](https://github.com/landing-ai/ade-helper-scripts/tree/main/use_cases/automated_invoice_parsing)** - Automated invoice extraction
- **[Utility Bill Parser](https://github.com/landing-ai/ade-helper-scripts/tree/main/use_cases/automated_utility_bill_parsing)** - Financial document processing

#### UI & Deployment
- **[Streamlit Frontend](https://github.com/landing-ai/ade-helper-scripts/tree/main/workflows/special_use_case/streamlit_frontend_for_ade)** - Build interactive UIs
- **[AWS Lambda Deployment](https://github.com/landing-ai/ade-helper-scripts/tree/main/workflows/special_use_case/ade_lambda_s3)** - Serverless processing


## Key Features

✅ **Various Document Formats Supported** - PDF, images, docx, pptx, csv, XLSX and [so much more](https://docs.landing.ai/ade/ade-file-types)

✅ **Native Proprietary Parsing Model** - Document Pretrained Transformer [DPT2](https://docs.landing.ai/ade/ade-parse-models)

✅ **Expanded Chunk Types** - [Text, table, marginalia, figure, scan_code, logo, card and attestation](https://docs.landing.ai/ade/ade-chunk-types)

✅ **Parallel Processing** - Async and non-blocking processing for [max throughput](https://docs.landing.ai/ade/ade-parse-async)

## Resources

Visit LandingAI [Homepage](https://landing.ai/)

Our [Youtube Channel](https://www.youtube.com/channel/UCYQS3jkfB79Diyr9sQJAj5Q)

Hackathons and [Events](https://events.landing.ai/) 

For Detailed Reference, [read the docs](docs.landing.ai)


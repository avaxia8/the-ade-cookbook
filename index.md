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
5. **[Document Classification](recipes.md#4-document-classification)** - Auto-categorize documents
5. **[Parallel Processing](examples.md#5-async-processing-for-large-files)** - Async handling for large files

### [Essential Concepts](concepts.md)
- [Parse Output Guide](concepts.md#understanding-parse-output) - Understand the parse response
- [Extraction Output Guide](concepts.md#understanding-extraction-output---your-data-detectives-report-) - Data detective's report
- [Split Output Guide](concepts.md#understanding-splits---document-organizer-magic) - Document organizer magic
- [Visual Grounding](concepts.md#visual-grounding) - Location tracking
- [Extraction Schema](concepts.md#extraction-schemas-explained) - Schemas as shopping lists for AI

### [Projects](https://github.com/landing-ai/ade-helper-scripts)
This repo contains many end-to-end examples that make use of ADE.


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


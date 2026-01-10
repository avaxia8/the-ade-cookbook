# The ADE Cookbook

Extract structured data with [AI-powered precision](https://landing.ai/blog/superhuman-on-docvqa-without-images-in-qa-agentic-document-extraction).

## Get Started

<table>
  <tr>
    <td width="33%" align="center">
      <h3>🎮 Playground</h3>
      <p>Try ADE instantly in your browser.</p>
      <a href="https://va.landing.ai"><strong>Open Playground →</strong></a>
    </td>
    <td width="33%" align="center">
      <h3>🐍 Python Library</h3>
      <p>Install and start coding.</p>
      <code>pip install landingai-ade</code><br><br>
      <a href="https://github.com/landing-ai/ade-python"><strong>Python Library →</strong></a>
    </td>
    <td width="33%" align="center">
      <h3>🔌 REST API</h3>
      <p>Direct API access for any language.</p>
      <br>
      <a href="https://docs.landing.ai/api-reference/tools/ade-parse"><strong>API Docs →</strong></a>
    </td>
  </tr>
</table>

## Workflow Overview
![The ADE Decision Tree](/images/ade_decision_tree.png)

## Document Parsing Quick Start
All workflows start with document parsing.

```python
import os
from pathlib import Path
from landingai_ade import LandingAIADE

# Initialize
client = LandingAIADE(
    api_key=os.environ.get("VISION_AGENT_API_KEY"), 
    environment="production",  # default is "production", "eu" for EU region
)

# Parse document first
response = client.parse(
    document=Path("invoice.pdf"),  # use document_url= for remote files
    model="dpt-2-latest", # refer to https://docs.landing.ai/ade/ade-parse-models for detailed model versions
    save_to="./output_folder",  # optional: saves as invoice_parse_output.json
)

print(f"Found {len(response.chunks)} chunks")
print(response.markdown[:200])  # Preview markdown
```

## Learn More

- **[How It Works](how-it-works.md)** - Visual explanation of the Parse → Extract workflow
- **[Concepts](concepts.md)** - All you need to know about Chunks, Schemas, Output Structures and so much more
- **[Examples](examples.md)** - 5 focused code examples for common tasks: Split, Extract, Classify, Parallel Processing and Save Visual Grounding
- **[Recipes](recipes.md)** - Production-ready patterns and solutions

## Key Features

✅ **Various Document Formats Supported** - PDF, images, docx, pptx, csv, XLSX and [so much more](https://docs.landing.ai/ade/ade-file-types)

✅ **Native Proprietary Parsing Model** - Document Pretrained Transformer [DPT2](https://docs.landing.ai/ade/ade-parse-models)

✅ **Expanded Chunk Types** - [Text, table, marginalia, figure, scan_code, logo, card and attestation](https://docs.landing.ai/ade/ade-chunk-types)

✅ **Parallel Processing** - Async and non-blocking processing for [max throughput](https://docs.landing.ai/ade/ade-parse-async)

<style>
.nav-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
  margin: 30px 0;
}

.card {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 20px;
  text-align: center;
}

.card h3 {
  margin-top: 0;
  color: #333;
}

.card code {
  display: block;
  background: #f5f5f5;
  padding: 10px;
  margin: 15px 0;
  border-radius: 4px;
  font-size: 14px;
}

.button {
  display: inline-block;
  background: #007bff;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  text-decoration: none;
  margin-top: 10px;
}

.button:hover {
  background: #0056b3;
}
</style>

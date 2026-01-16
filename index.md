# The ADE Overview
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

### 💡 Essential Concepts(Link to the ADE essential concepts blog post)


### 👨🏻‍💻 Code Examples(link to the docs sample code section)


### 🚀 [Projects](https://github.com/landing-ai/ade-helper-scripts)
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


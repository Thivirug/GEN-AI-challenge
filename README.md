# GEN-AI Challenge: Diagnostic Code Mapping for Mechanic Notes

## Context & Objective

This repository contains my technical solution to the GenAI-powered tool challenge: mapping free-text mechanic descriptions to the correct automotive diagnostic codes (DTCs). The goal is to help mechanics quickly identify the most relevant codes based on their notes, using only provided resources (dictionary.json and sample_data.json) without any LLM training or finetuning.

---

## Approach Overview

### 1. Exploratory Data Analysis

- **Diagnostic Dictionary**: Loaded and reviewed the structure of codes and their descriptions. The codes follow standard automotive DTC formats (e.g., "P0102"), with descriptions that are concise but often technical.
- **Mechanic Notes Dataset**: Inspected the sample set of mechanic notes. These notes are brief, often use mechanic lingo, abbreviations, and contain spelling errors or incomplete grammar (e.g., "maf sig intermittnt, cuts out sumtimes").
- **Key Challenges Identified**:
  - Informal language with domain-specific abbreviations.
  - Frequent spelling errors and non-standard phrasing.

---

### 2. Baseline: Zero-Shot / Few-Shot Prompting with LLM

- **Prompt Design**: Crafted two styles:
  - **Zero-Shot**: Direct prompt instructing the LLM to output only the code for a given mechanic note.
  - **Few-Shot**: Prompt includes a couple of mechanic-note/code pairs as examples to encourage pattern recognition.
- **Implementation**: Used Hugging Face’s `flan-t5-base` model (or similar). The notebook contains the prompt templates and code for inference.
- **Inference Logic**: For each mechanic note, prompt the LLM and record its code prediction.
- **Documentation**: The prompt was deliberately constrained to request only the code output (to avoid extra text and ambiguity).
- **Why This Design?**: Mechanic notes are noisy; explicit prompts and few-shot examples help guide the model to focus on the DTC output and ignore extraneous details.

---

### 3. Improved: Retrieval-Augmented Generation (RAG)

- **Embedding Step**: Used sentence-transformers (`all-MiniLM-L6-v2`) to embed both the mechanic note and all dictionary descriptions.
- **Retrieval**: For each note, computed cosine similarity to retrieve the top-N most similar diagnostic codes from the dictionary.
- **Final Selection**:
  - **Option 1**: Use LLM with a new prompt, presenting the top candidate codes and asking it to select the best match.
- **Why RAG?**: Direct prompting struggles with noisy language. Retrieval narrows the search space and provides contextually relevant options for the LLM, improving accuracy.

---

## Testing & Results

- **Metrics Computed**:
  - **Accuracy**: Percentage of exact matches between predicted and true codes.
- **Results**:
  - **Zero-Shot Accuracy**: ~1.32%
  - **Few-Shot Accuracy**: ~1.32%
  - **RAG Accuracy**: ~34.21%
- **Interpretation**:
  * Prompt-only methods are insufficient for this noisy domain, often defaulting to frequent codes or failing to parse abbreviations. Retrieval-augmented generation offers a substantial improvement by leveraging semantic similarity.
  * The LLM model used (~250 mil params) highly affects the generated codes. Due to memory contraints this small model was chosen, but if a larger model was chosen, the results are expected to be better.
  * Another factor which affects the output of RAG method specifically is the embedding model used. Again, if a state-of-the-art model was used, the results would have been signifcantly better as the semantic variations can be captured more accurately by those models.

---

## Failure Analysis & Takeaways

- **Example Failures**:
  - Prompt-only approaches often output the same frequent code regardless of note specifics.
  - RAG sometimes retrieves similar but not exact matches (e.g., different sensor codes for similar issues).
- **Improvement Suggestions**:
  - More sophisticated embedding models or domain-specific retrievers could further boost accuracy.
  - Finetuning or post-processing on mechanic lingo would help.
  - Incorporating top-N accuracy or providing multiple suggested codes may be more practical for real-world use (something like a probability ranking).

---

## Design Decisions

- **No Data Cleaning**: All approaches work directly on raw, noisy mechanic notes as required.
- **Explicit Prompt Constraints**: To avoid LLM output ambiguity, prompts were structured to request only code outputs.
- **RAG Pipeline**: Embedding + retrieval + LLM selection combines strengths of semantic similarity and generative reasoning.
- **Reproducibility**: All code is provided in the notebook; results are saved to JSON for transparency.

---

## Final Thoughts

- **Prompting alone is fragile** for specialized, noisy domains like mechanic notes.
- **Retrieval augmentation** dramatically improves robustness and accuracy.
- **Future work**: Larger mechanic note datasets, domain-specific embeddings, and feedback-driven improvements would close the gap with expert human performance.

---

## Repository Structure

- `Challenge.ipynb`: Main notebook implementing all steps.
- `dictionary.json`: Diagnostic code dictionary.
- `sample_data.json`: Mechanic notes and ground truth codes.
- `zero_shot_results.json`, `few_shot_results.json`, `RAG_results.json`: Output files for each approach.

---

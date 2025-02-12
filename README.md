# Cadastral Takehome Assignment
**Author**: Rohan Prakash \
**Time Spent**: ~6 hours

### Introduction
This analysis evaluates the effectiveness and accuracy of OCR (Optical Character Recognition) providers for Cadastral's product design requirements. Our investigation centers on Tesseract, an open-source OCR engine developed by Google and implemented through the Python library pytesseract. While LLMs (Large Language Models) have advanced significantly in text extraction capabilities, OCR remains essential for Cadastral’s business needs due to its ability to provide bounding box information for text localization.

To establish a reliable ground truth for evaluation, we leveraged Claude's PDF processing capabilities, which demonstrated higher accuracy in text extraction compared to its Vision API. Our methodology involves comparing Tesseract's OCR output against this ground truth, enabling a systematic assessment of the OCR engine's performance in real-world scenarios.

### Data Curation
Our evaluation methodology focused on establishing reliable ground truth data using Claude Sonnet's PDF processing capabilities. The key challenge was crafting precise prompts that would extract clean, structured text while preserving the document's original formatting. 

Through some iterative prompt engineering, we developed instructions that specifically:
- Requested raw text extraction without any AI-generated commentary or reformatting
- Emphasized preservation of document structure including newlines, indentation, and spatial relationships

Initial experiments with Claude's Vision API, while accurate in content extraction, resulted in flattened outputs that lost critical structural elements. The PDF processing endpoint proved superior by maintaining document hierarchy and formatting crucial for real estate documents. 

The implementation utilized pytesseract for OCR processing, converting PDFs to images using pdf2image. Each page was processed individually to maintain granular control over the extraction process and enable detailed comparison with our structured ground truth data.

Throughout our extraction pipeline, we logged per-page processing times for both approaches to understand the operational trade-offs:

**Latency Analysis**
| API | Mean (s) | Max (s) |
| --- | --- | --- |
| Claude PDF | 23.19 | 38.54 |
| Tesseract OCR | 1.84 | 2.43 |

### Metrics
When evaluating OCR performance for real estate documents, several critical factors guide our choice of metrics. Document structure preservation is essential, as real estate documents often contain formatted sections like property descriptions, legal clauses, and tabulated data. Content accuracy is crucial for legal compliance and data extraction reliability. Additionally, the ability to handle varied formatting, including addresses, parcel numbers, and legal descriptions, requires metrics that can assess both exact matches and structural similarity.

Our evaluation framework employs four complementary metrics:

1. **Character Error Rate (CER)**
    - Measures character-level edit distance between OCR output and ground truth
    - Particularly valuable for real estate documents where precise transcription of parcel numbers, addresses, and legal descriptions is critical
    - However, its sensitivity to minor shifts can overstate errors in cases where content is correct but slightly reformatted
2. **Word Error Rate (WER)**
    - Assesses accuracy at the word level, providing a more natural measure for document comprehension
    - Especially relevant for legal clauses and property descriptions where word-level accuracy is more important than character-perfect matches
    - Better handles common OCR challenges like word spacing issues in formatted documents
3. **Fuzz Partial Ratio**
    - Evaluates similarity based on longest contiguous matching substring
    - Excels at handling common OCR issues in real estate documents such as:
        - Line breaks in addresses
        - Variations in formatting of legal descriptions
        - Slight misalignments in tabulated data
    - More forgiving of structural differences while maintaining focus on content accuracy
4. **Text Similarity (SequenceMatcher)**
    - Identifies longest contiguous matching subsequences
    - Particularly effective for evaluating structural preservation of:
        - Section headers and hierarchical document organization
        - Paragraph boundaries in legal descriptions
    - Provides a balanced measure of both content and structural similarity

This comprehensive set of metrics allows us to evaluate OCR performance across different aspects of real estate document processing, from precise character recognition to overall document structure preservation.

### Results
| Metric | Mean | Median (50%) | 75th percentile |
| --- | --- | --- | --- |
| Character Error Rate (CER) | 0.104 | 0.013 | 0.056 |
| Word Error Rate (WER) | 0.181 | 0.070 | 0.175 |
| Fuzz Partial Ratio | 93.74 | 99.00 | 100.00 |
| Text Similarity | 0.825 | 0.917 | 0.977 |

Before analyzing the performance metrics, it's important to note that several pages in our 46-page real estate document contained non-textual elements such as diagrams (ex. Page 29) and tables. Since Tesseract OCR is primarily designed for character recognition, these pages naturally resulted in lower similarity scores when compared against the ground truth. This accounts for some of the outlier measurements in our results. For detailed error measures across each page, refer to the accompanying notebook.

**Performance Analysis**

- Character-level performance indicates most transcription errors are isolated to specific challenging sections rather than distributed throughout the document. This suggests Tesseract reliably handles standard text content while struggling primarily with edge cases.
- Word-level accuracy shows expected degradation from character-level performance, but maintains sufficient accuracy for practical use in text extraction tasks. The median WER of 0.070 suggests most content remains semantically intact.
- Structure preservation stands out as Tesseract's strongest feature, with Fuzz Partial Ratio scores consistently above 95 for most pages. This is particularly valuable for real estate documents where layout and formatting carry semantic meaning.
- Text Similarity scores confirm Tesseract's overall reliability, with a median of 0.932 indicating strong preservation of both content and structure across most pages.

### Conclusion

Tesseract OCR demonstrates strong performance for real estate document processing, with high accuracy in both text extraction and structural preservation, while maintaining significantly faster processing times compared to LLM-based approaches. Its robust performance across multiple evaluation metrics, particularly in maintaining document structure and layout, makes it a viable solution for Cadastral's document processing needs.

The modular evaluation framework developed in this analysis can be readily extended to assess other OCR providers, requiring minimal modifications to accommodate new APIs. Future work could focus on:
- Enhancing OCR performance through image preprocessing techniques
- Evaluating other OCR providers using the existing evaluation framework
- Developing domain-specific accuracy metrics for real estate documents (e.g., address formatting, parcel number accuracy)


\
**Final OCR Score**: 85/100 


### References:
[Comprehensive Guide to OCR Implementation](https://www.cloudraft.io/blog/comprehensive-ocr-guide)  
[Claude Documentation](https://docs.anthropic.com/en/docs/welcome)  
[PyTesseract - Python-tesseract is an optical character recognition (OCR) tool for python](https://github.com/madmaze/pytesseract)\
[R. Smith, "An Overview of the Tesseract OCR Engine," Ninth International Conference on Document Analysis and Recognition (ICDAR 2007)](https://doi.org/10.1109/ICDAR.2007.4376991)
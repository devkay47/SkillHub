---
name: pdf-text-extractor
description: Extracts text, tables, and metadata from PDF files using pdfplumber, handling multi-column layouts, scanned documents, and mixed content. Use when the user needs to read a PDF, extract text from a PDF, parse PDF tables, or process a PDF file programmatically.
---

## Overview

This skill extracts content from PDF files reliably, handling the three main PDF types developers encounter: text-based PDFs (direct extraction), multi-column PDFs (layout-aware extraction), and scanned PDFs (OCR pipeline). It uses pdfplumber as the primary library because it provides superior table detection compared to PyPDF2 or pdfminer.six.

## Instructions

When this skill activates, first determine the PDF type by attempting text extraction on the first page. If the result is empty or contains only garbled characters, the PDF is scanned and requires OCR. If it contains readable text, proceed with direct extraction.

**For text-based PDFs:** Use pdfplumber to extract text page by page. Preserve page boundaries by inserting a page marker between pages. For tables, use `page.extract_tables()` which returns a list of lists — convert these to a structured format (list of dicts with the first row as headers) before returning them.

**For multi-column PDFs:** Use pdfplumber's `crop()` method to extract each column separately. Identify column boundaries by analysing the x-coordinates of text elements on the page. Column boundaries are usually evident from gaps in horizontal text distribution.

**For scanned PDFs:** Convert pages to images using pdf2image, then run each image through pytesseract. Always set `lang='eng'` explicitly. Apply image preprocessing (grayscale conversion, contrast enhancement) before OCR to improve accuracy on low-quality scans.

Always extract and return the PDF metadata (title, author, creation date, page count) alongside the content. Report any pages where extraction failed, with the reason, rather than silently skipping them.

## Example Output Structure

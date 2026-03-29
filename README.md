# Universal Doc Converter

A browser-based document conversion tool that runs entirely in the client. No server uploads—all processing happens locally.

## Features

| Mode | Input | Output |
|------|-------|--------|
| **PDF to Image** | PDF file | PNG, JPEG, or TIFF per page |
| **Image to PDF** | Multiple images | Single PDF |
| **Word to PDF** | .docx file | PDF |
| **Merge PDFs** | Multiple PDFs | Single combined PDF |
| **Unlock PDF** | Password-protected PDF | Unprotected PDF |
| **Encrypt PDF** | Unprotected PDF | Password-protected PDF |

## How It Works

### Architecture

```
User uploads file → Mode-specific processor → Output (download or gallery)
```

### 1. PDF to Image

- **Library**: PDF.js
- **Flow**:
  1. Load PDF using `pdfjsLib.getDocument()` (prompts for password if encrypted)
  2. For each page: render to a canvas with the selected scale (1x, 1.5x, 2x)
  3. Export canvas as PNG, JPEG, or TIFF (TIFF via UTIF.js)
  4. Show a gallery with per-page preview and download buttons

### 2. Image to PDF

- **Library**: jsPDF
- **Flow**:
  1. Read each image as a data URL
  2. Add images to a jsPDF document (one per page)
  3. Page size options:
     - **A4**: Fit image to A4 (width-based)
     - **Fit**: Scale image to fit A4 while keeping aspect ratio

### 3. Word to PDF

- **Libraries**: Mammoth.js → html2canvas → jsPDF
- **Flow**:
  1. Mammoth converts .docx to HTML
  2. HTML is rendered into a hidden off-screen div
  3. html2canvas captures the div as an image
  4. jsPDF creates a multi-page PDF from the image (split by A4 height)

### 4. Merge PDFs

- **Library**: pdf-lib
- **Flow**:
  1. User adds multiple PDFs to a staging list (add/remove/reorder)
  2. Each PDF is loaded with `PDFDocument.load()`
  3. Pages are copied losslessly into a new document via `copyPages()`
  4. Single merged PDF is saved for download

### 5. Unlock PDF

- **Libraries**: pdf-lib, PDF.js, jsPDF
- **Flow** (two strategies depending on encryption type):
  - **Owner-password-only** (restrictions like no-print/no-copy): pdf-lib loads with `ignoreEncryption: true` and copies pages losslessly
  - **User-password** (content encrypted): PDF.js prompts for the password, renders each page at 2x scale, and jsPDF rebuilds an unprotected PDF
  - Non-encrypted PDFs are detected and rejected with a message

### 6. Encrypt PDF

- **Libraries**: PDF.js, jsPDF
- **Flow**:
  1. User uploads a PDF and enters a password (with confirmation)
  2. PDF.js reads and renders each page at 2x scale
  3. jsPDF creates a new PDF with `encryption: { userPassword, ownerPassword }` enabled
  4. Pages are rendered as high-quality JPEG images in the encrypted output

## Key Functions

| Function | Purpose |
|----------|---------|
| `switchMode(mode)` | Switches between tools and updates UI/hints |
| `handleFiles(e)` | Validates files and routes to the correct processor |
| `processPdfToImg(file)` | PDF → images |
| `processImgToPdf(files)` | Images → single PDF |
| `processWordToPdf(file)` | Word → PDF |
| `processMergePdf(files)` | Multiple PDFs → single PDF (lossless) |
| `processUnlockPdf(file)` | Encrypted PDF → unprotected PDF |
| `processEncryptPdf(file, pw)` | PDF → password-protected PDF |
| `promptForPassword()` | Shows password modal, returns entered password or null |
| `loadPdfWithPassword()` | Retry loop for PDF.js password entry |
| `saveBlobWithPicker()` | Saves file via File System Access API or fallback download |

## Dependencies (CDN)

- **Tailwind CSS** – styling
- **PDF.js** – read and render PDFs
- **UTIF.js** – TIFF encoding
- **jsPDF** – create PDFs (with encryption support)
- **pdf-lib** – lossless PDF manipulation (merge, unlock)
- **Mammoth.js** – Word (.docx) to HTML
- **html2canvas** – HTML to canvas
- **Lucide** – icons

## Usage

1. Open `index.html` in a modern browser.
2. Choose a mode from the tab bar.
3. Drag and drop files or click the upload button.
4. For Merge PDFs: add files to the staging list, then click "Merge PDFs".
5. For Encrypt PDF: upload a PDF, set a password, then click "Encrypt PDF".
6. Download the converted files.

## Notes

- All processing is client-side; files never leave your machine.
- `showSaveFilePicker` (File System Access API) is used when available for better save dialogs.
- Word conversion preserves layout via HTML rendering; complex formatting may not match exactly.
- Encrypt and Unlock (user-password) modes render pages as images; vector text and fonts are rasterized at 2x resolution.
- Merge and Unlock (owner-password) modes are fully lossless—pages are copied directly without re-rendering.

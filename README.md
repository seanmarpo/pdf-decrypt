# @localonlytools/pdf-decrypt

> **Fork notice:** This is a fork of [`@pdfsmaller/pdf-decrypt`](https://github.com/smither777/pdf-decrypt) by [PDFSmaller.com](https://pdfsmaller.com). All credit for the original work goes to the original author.

Full-featured PDF decryption with **AES-256**, **AES-128**, and **RC4** support. Built for browsers, Node.js 18+, Cloudflare Workers, and Deno.

This package powers the PDF tools on [Local Only Tools](https://localonly.tools) — a privacy-focused collection of browser-based utilities where all processing happens entirely on your device. No data is ever transmitted to a server. Check out the full suite of tools at [localonly.tools](https://localonly.tools), including [PDF Merge](https://localonly.tools/tools/pdf-merge/), [PDF Split](https://localonly.tools/tools/pdf-split/), [PDF Compress](https://localonly.tools/tools/pdf-compress/), and [PDF Redact](https://localonly.tools/tools/pdf-redact/).

## Features

- **AES-256 decryption** (V=5, R=6) — PDF 2.0 standard
- **AES-128 decryption** (V=4, R=4) — PDF 1.6+ standard
- **RC4 40/128-bit decryption** (V=1-2, R=2-3) — legacy support
- **User + Owner passwords** — accepts either password to decrypt
- **Batched async decryption** — processes thousands of objects without browser freeze
- **Web Crypto API** — no native dependencies, works everywhere
- **Lightweight** — ~18KB total (crypto + decryption logic)
- **Zero dependencies** — only `pdf-lib` as a peer dependency
- **TypeScript types** included

## Installation

```bash
npm install @localonlytools/pdf-decrypt pdf-lib
```

## Quick Start

```javascript
import { decryptPDF } from '@localonlytools/pdf-decrypt';
import fs from 'fs';

const pdfBytes = fs.readFileSync('encrypted.pdf');
const decrypted = await decryptPDF(new Uint8Array(pdfBytes), 'my-password');
fs.writeFileSync('decrypted.pdf', decrypted);
```

## API

### `decryptPDF(pdfBytes, password)`

Decrypt a password-protected PDF. Supports AES-256, AES-128, and RC4 encryption — the algorithm is detected automatically.

| Parameter | Type | Description |
|-----------|------|-------------|
| `pdfBytes` | `Uint8Array` | The encrypted PDF file as bytes |
| `password` | `string` | The user or owner password |

**Returns:** `Promise<Uint8Array>` — The decrypted PDF bytes

**Throws:**
- `"This PDF is not encrypted"` — if the PDF has no encryption dictionary
- `"Incorrect password"` — if neither user nor owner password matches
- `"Unsupported encryption"` — if the encryption version is not supported

### `isEncrypted(pdfBytes)`

Check if a PDF is encrypted without attempting to decrypt it.

| Parameter | Type | Description |
|-----------|------|-------------|
| `pdfBytes` | `Uint8Array` | The PDF file as bytes |

**Returns:** `Promise<{ encrypted: boolean, algorithm?: 'AES-256' | 'AES-128' | 'RC4', version?: number, revision?: number, keyLength?: number }>`

## Examples

### Decrypt with Auto-Detection

```javascript
import { decryptPDF, isEncrypted } from '@localonlytools/pdf-decrypt';

// Check encryption type first
const info = await isEncrypted(pdfBytes);
if (info.encrypted) {
  console.log(`Encrypted with ${info.algorithm}`);
  const decrypted = await decryptPDF(pdfBytes, password);
}
```

### Roundtrip with @pdfsmaller/pdf-encrypt

```javascript
import { encryptPDF } from '@pdfsmaller/pdf-encrypt';
import { decryptPDF } from '@localonlytools/pdf-decrypt';

// Encrypt
const encrypted = await encryptPDF(pdfBytes, 'secret');

// Decrypt
const decrypted = await decryptPDF(encrypted, 'secret');
```

### Browser Usage

```html
<input type="file" id="pdf-input" accept=".pdf" />
<input type="password" id="password" placeholder="Enter password" />
<button id="decrypt-btn">Decrypt</button>

<script type="module">
  import { decryptPDF } from '@localonlytools/pdf-decrypt';

  document.getElementById('decrypt-btn').addEventListener('click', async () => {
    const file = document.getElementById('pdf-input').files[0];
    const password = document.getElementById('password').value;
    const pdfBytes = new Uint8Array(await file.arrayBuffer());

    try {
      const decrypted = await decryptPDF(pdfBytes, password);

      // Download
      const blob = new Blob([decrypted], { type: 'application/pdf' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'decrypted.pdf';
      a.click();
    } catch (e) {
      alert(e.message);
    }
  });
</script>
```

## Supported Encryption

| Algorithm | PDF Version | Key Length | Status |
|-----------|-------------|-----------|--------|
| AES-256 (V=5, R=6) | 2.0 (ISO 32000-2) | 256-bit | Supported |
| AES-128 (V=4, R=4) | 1.6+ (ISO 32000-1) | 128-bit | Supported |
| RC4 (V=2, R=3) | 1.4+ (ISO 32000-1) | 128-bit | Supported |
| RC4 (V=1, R=2) | 1.1+ | 40-bit | Supported |

## Comparison with pdf-decrypt-lite

| Feature | pdf-decrypt | pdf-decrypt-lite |
|---------|-------------|-----------------|
| AES-256 | Yes | No |
| AES-128 | Yes | No |
| RC4 128-bit | Yes | Yes |
| RC4 40-bit | Yes | Yes |
| Batched async | Yes | No (sync only) |
| Size | ~18KB | ~8KB |
| Use case | Full decryption | RC4-only, minimal |

Choose `pdf-decrypt-lite` if you only need RC4 and want the smallest possible bundle. Choose `pdf-decrypt` for full AES-256 + AES-128 + RC4 support.

## Related Packages

| Package | Description |
|---------|-------------|
| [@pdfsmaller/pdf-encrypt](https://www.npmjs.com/package/@pdfsmaller/pdf-encrypt) | Full encryption — AES-256 + RC4 (companion to this package) |
| [@pdfsmaller/pdf-encrypt-lite](https://www.npmjs.com/package/@pdfsmaller/pdf-encrypt-lite) | Lightweight RC4-only encryption (~7KB) |
| [@pdfsmaller/pdf-decrypt-lite](https://www.npmjs.com/package/@pdfsmaller/pdf-decrypt-lite) | Lightweight RC4-only decryption (~8KB) |

## License

MIT — Originally by [PDFSmaller.com](https://pdfsmaller.com). See the [original repository](https://github.com/smither777/pdf-decrypt) for more details. Maintained by [Local Only Tools](https://localonly.tools).

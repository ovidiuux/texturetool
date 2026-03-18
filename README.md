# texturetool

texturetool is a utility designed for mobile textures (VC): efficient extraction, deduplication, and rebuilding of texture archives with support for standard image and compression libraries.

---
## Features

### PNG Support (libpng + zlib)
Uses industry-standard libraries:
- **libpng**
- **zlib**

### Offset Table (.toc)

- Reads and validates `.toc` files
- Regenerates offsets if invalid or missing
- Ensures `.dat` consistency using file size checks


### Hash-Based Validation

Each texture entry is validated using a hash of its name.

- Prevents `.txt` / `.dat` desync
- Ensures correct mapping without storing names in `.dat`

### DXT Compression Support (libsquish)
Supports:
- DXT1
- DXT3
- DXT5

Uses **libsquish** for high-quality texture processing.

### Automatic Texture Deduplication
Detects identical textures even if they have different names.


## Planned Features

- **ETC (Ericsson Texture Compression)** support  
- **PVR / PVRTC (PowerVR Texture Compression)** support  

These formats are used heavily on mobile GPUs and will extend compatibility.

## Dependencies

- https://zlib.net/
- http://www.libpng.org/
- https://github.com/svn2github/libsquish

---

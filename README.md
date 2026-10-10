# texturetool

Converts Vice City PC RenderWare TXDs and PNGs into Vice City mobile texture databases (`.txt`, `.dat`, `.toc`, `.tmb`). Supports DXT1/3/5, ETC1, PVRTC 2/4-bpp RGB/RGBA, raw RGBA32, RGBA5551, RGBA4444, luminance and luminance-alpha textures. Batch import reads neighboring DFF material usage and GTA III/VC COLL collision models.

---

## Commands

```powershell
texturetool build "C:\textures\frankiesbase.txd" -pvr
texturetool build "C:\textures\png_folder" -pvr
texturetool "C:\textures\out\frankiesbase.txt" -pvr
texturetool import "C:\textures\gta3img.txt" "C:\textures\pc_txds" -pvr
texturetool convert gta3img -pvr
texturetool convert "C:\textures\gta3img.txt" "C:\converted\gta3img" -pvr
```

`build` writes to an `out` directory beside the input. Extraction uses an explicit `.txt` path and writes PNGs into a directory named after the database. `import` updates the destination database; `convert` accepts a separate output base. Without an output argument, conversion writes beside the source unless PVRTC requires resizing: it then writes all four files under `out/<base>` to preserve the source cache. Import's `--normal` option strips optional rendering hints from imported entries.

With `-pvr`, build/import/convert default to **PVRTC1 2-bpp RGB** (`0x8C01`) for opaque textures and **PVRTC1 4-bpp RGBA** (`0x8C02`) for textures containing alpha, including luminance-alpha images. No additional alpha option is required. Here “PVRTC2/PVRTC4” means 2/4 bits per pixel of PVRTC1, rather than the separate PVRTC-II format. PNG suffixes can explicitly request other supported PVRTC variants or raw RGBA. Build/import preserve NPOT dimensions using raw RGBA. `convert ... -pvr` resizes NPOT images to power-of-two dimensions before PVRTC compression, using alpha-weighted bilinear filtering.

PVRTextureLib compresses each texture's complete, alpha-weighted mip chain in one call, using its `PVRTCFastest` quality preset. Texture jobs run in parallel; the library uses one worker per job to avoid nested thread pools. PVRTC is lossy, including its alpha channel.

## Independent metadata, DFF and COL processing

Build/import use their source TXD/PNG pixels. They require no original mobile databases, extracted reference images, configuration file or environment variable. Existing destination properties and alias/TXD context are retained during import where appropriate. Output aliases point to textures actually present in the output database; reimport updates the existing qualified TXD variant. DFF geometry, material usage, source IDE definitions and spatial COL correspondence supply numerical context for the embedded property classifier. Compression still depends on the image, independently of model mask/opacity hints.

Import discovers TXDs recursively, excluding `out` and hidden directories. A source inside a `TXD`/`DFF` subdirectory uses its parent region for model context. All DFFs in that context are scanned once per batch for texture spelling, base/mask names, material alpha, geometry prelight alpha, transformed triangles, UVs and surface orientation. Optional source `.ide` files associate models with TXD dictionaries and provide numerical flags/draw distances; they are also discovered in a neighboring game DATA/MAPS tree. DFF names need not match the TXD name. Opaque, binary and gradual image alpha are classified separately. Endpoint noise may use one-bit alpha only if the error on partial-alpha pixels has RMS at most 8/255 and no individual error exceeds 16/255, with both transparent and opaque pixels present. Even one meaningful soft texel retains gradual alpha. A DFF mask cannot override this decision.

For binary/near-binary images, the actual DXT1-A and DXT5 encoders are compared on visible RGB and compositing against black/white backgrounds. DXT5 is selected if it materially reduces an error above the 3/255 RMS budget. This handles small colored textures and text without a texture-name or size rule. Otherwise DXT1-A remains sufficient. PNG builds, TXD imports and conversions share this selection. Explicit PNG codec suffixes remain overrides. Source PC RGBA storage no longer forces mobile RGBA when compression is appropriate. Binary `alphamode=1` does not downgrade a quality-selected DXT5 payload.

All neighboring `.col` files are parsed automatically as GTA III/VC `COLL` records: bounds, spheres, lines, boxes, vertices, triangle indices and physical surface bytes. Record boundaries, finite coordinates and vertex indices are validated. An invalid file contributes no partial model list. COL parsing is part of build/import and has no separate CLI command.


## Dependencies

- https://zlib.net/
- http://www.libpng.org/
- https://github.com/svn2github/libsquish

---

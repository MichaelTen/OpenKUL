# OpenKUL

Keeping Unified Layers

# OpenKUL Specification

**OpenKUL** (“Keeping Unified Layers”) will be an open container format and toolchain for classic tile-based multiplayer worlds. It takes inspiration from Ultima-like engines, but all design, art, and assets will be original. The goal is to provide a modern, permissively licensed alternative to legacy `.mul` files, while still allowing optional compatibility for communities that wish to connect with servers such as ServUO or clients such as ClassicUO.

---

## Philosophy

- **Original assets only**: OpenKUL will focus on brand new artwork, music, sounds, and gameplay data. No Ultima Online assets will be included or distributed.  
- **Separation from GPL code**: The Steam-distributable client and `.kul` assets will remain completely standalone. Players may perhaps connect to ServUO (GPLv3) servers over the public network protocol, but the client will not include ServUO code. This separation will avoid GPL licensing obligations on the client.  
- **Compatibility bridge**: A conversion/exporter tool may allow `.kul` packages to be written out as `.mul`/`.idx` files for legacy compatibility. Over time, engines may be patched to read `.kul` directly.  
- **Future-proofing**: Multiple resolutions (44px, 88px, etc.), clean manifests, and explicit versioning will help ensure OpenKUL remains usable long-term.  

---

## File Extensions

- `.kul` — the primary container format (binary-framed, streamable)  
- `.2kul` — optional high-resolution container (88px)  
- `.kulf` — optional plain-text manifest sidecar (human-readable)  

Note: resolution variants may be best handled via `@2x` asset naming in the manifest, but `.2kul` could still be useful shorthand if desired.

---

## Container Structure

### Option 1: ZIP Container
- Root may contain `manifest.json`  
- Assets may be placed under `/tiles/`, `/maps/`, `/art/`  
- This seems the most straightforward to author and edit

### Option 2: Binary Container
- 8-byte header: magic string `KUL0` + version  
- Directory of resource tables (offset/length/type)  
- Payload sections for maps, statics, art, etc.  
- Designed for fast seeks and streamable reads  

Both approaches may share the same manifest schema.

---

## Manifest (JSON Sketch)

```json
{
  "format": "openkul",
  "version": 1,
  "canonical_tile_size": 44,
  "tile_variants": [1, 2],
  "atlases": {
    "terrain": {
      "1x": "tiles/terrain@1x.png",
      "2x": "tiles/terrain@2x.png",
      "meta": "tiles/terrain.meta.json"
    }
  },
  "maps": [
    {
      "id": "world_a",
      "size_tiles": [7168, 4096],
      "layers": ["height", "terrain", "statics"],
      "sources": {
        "height": "maps/world_a_height.kub",
        "terrain": "maps/world_a_terrain.kut",
        "statics": "maps/world_a_statics.kus"
      }
    }
  ]
}
````

---

## Map Data Options

In legacy UO `.mul` files (e.g. `map0.mul`), each cell combines a **terrain ID** and a small **altitude value** into the same structure. This was efficient for its time, but it may feel tightly coupled and harder to extend.

OpenKUL may allow either of two approaches:

1. **Combined style (UO-like)**

   * Each cell record may hold both terrain ID and height together.
   * This seems simplest for compatibility and for exporting back to `.mul` when needed.

2. **Separated layers (modern style)**

   * Terrain IDs may be stored in `.kut` (tile/art indices).
   * Heights may be stored in `.kub` (int16 heightmap values).
   * Statics may be stored in `.kus` (objects placed on top).
   * This separation may make it easier for engines like Godot to stream and render data flexibly.

Both options could be valid. The manifest may specify which method is used. Over time, the separated layer model may prove cleaner, while the combined model may help with legacy compatibility.

---

## Engine Integration

* **Godot 4.4.1**: A loader will read the manifest, select the appropriate atlas variant (`1x`, `2x`), map tile ids to rects, and build terrain meshes from `.kub` height data.
* **ClassicUO / ServUO**: An exporter will convert `.kul` into `.mul` for legacy compatibility.
* **Scalability**: `.kul` seems format-agnostic and may be paired with any server backend.

---

## Related Projects

Other open-source MMO server frameworks (non-GPL, permissive licenses) may be relevant:

* **Nakama** (Apache-2.0) — Modern, cloud-native game server with Godot SDK.
* **Orleans** (MIT) — Actor-model framework used in Xbox Live and Halo; highly scalable.
* **FinEx Server** (MIT) — Java-based modular MMO backend.
* **Multiverse Platform** (MIT) — Virtual world engine for sandbox MMOs.

OpenKUL is intended to work with these as well, not tied to any single backend.

---

## Naming and Community Optics

* **OpenKUL** will be distinct from Ultima Online’s `.mul` files, avoiding legacy baggage.
* By design, it will be a clean, modern container format that can perhaps echo classic mechanics without replicating copyrighted expressions.
* Skills, stats, and professions may be renamed or re-themed while maintaining gameplay depth (e.g., “Swordsmanship” → “Sword Work”).

---

## Roadmap

1. Finalize the container spec (`.kul`/`.2kul`, manifest schema)
2. Publish a reference loader (Godot 4.4.1)
3. Implement an exporter for legacy `.mul`/`.idx`
4. Encourage adoption in non-GPL servers (Nakama, Orleans, FinEx)
5. Grow community-authored `.kul` asset packs

```

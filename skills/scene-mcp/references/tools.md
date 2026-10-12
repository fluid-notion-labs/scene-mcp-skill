# Tool surface

The server registers every tool below over stdio (rmcp). Names are exactly
as listed. All coordinates are metres, Y-up, rotations are quaternions
`[x, y, z, w]`.

## Scene lifecycle

| Tool | Parameters | Notes |
|---|---|---|
| `scene_new` | `path` | Fresh scene; sets the save path |
| `scene_open` | `path` | Open an existing GLB |
| `scene_save` | `path?` | Write the current scene (defaults to its path) |
| `export_glb` | `path?` | Export GLB (single file, JSON + BIN chunks) |
| `import_glb` | `path` | Foreign GLB in as an opaque mesh node; loses textures, keeps geometry |

## Structure

| Tool | Parameters | Notes |
|---|---|---|
| `create_group` | `name`, `parent?` | Container node |
| `add_primitive` | `name`, `kind` (box/cylinder/plane), `dimensions`, `position?`, `parent?` | box: full extents `[sx,sy,sz]`; cylinder: `[radius,height]`; plane: `[width,depth]` (XZ, normal +Y) |
| `rename` | `id`, `name` | |
| `reparent` | `id`, `parent?` | Cycle-checked |
| `delete` | `id` | Deletes subtree |
| `set_transform` | `id`, `translation?`, `rotation?`, `scale?` | Scale is **rejected** on parametric primitives (dimensions are baked in) — allowed on groups and imported nodes |

## Query

| Tool | Parameters | Notes |
|---|---|---|
| `get_tree` | `depth?`, `root?` | Text tree + JSON; shows `name#id [part_id]` |
| `get_node` | `id` | Node detail; mesh is a *summary* (kind, vertex/triangle counts) |
| `list_objects` | `root?` | Flat listing |
| `bill_of_materials` | `root?` | Aggregates `part_id`, quantity, material |

## Part metadata

`set_part_info` — all fields optional except `id` and `part_id`:

| Field | Meaning |
|---|---|
| `part_id` | Stable identity, e.g. `stud-90x45-mgp10`; BOM aggregates by it |
| `category` | Free text, e.g. `stud walls`, `diagram-node` |
| `label` | Short text rendered **on the part** in the viewer (diagram labels, signage) |
| `sku`, `supplier` | Catalogue fields |
| `nominal_dimensions` | `[l, w, h]` metres, as-sold size |
| `unit` | `each`, `lm`, `m2`, `sheet`… |
| `quantity` | Per-node count for the BOM |
| `material` | `{kind, spec, treatment?}` — see materials-bom.md for the kind vocabulary |
| `notes` | Hover-only detail in the viewer |

The whole part record rides the GLB in node `extras.part`; the stable node
id rides in `extras.scene_id`, so identities survive export/import.

## Image prep (photos → generative massing input)

Needs `CLOUDFLARE_ACCOUNT_ID` / `CLOUDFLARE_API_TOKEN` (from `env/.env`, nearest
ancestor directory); clutter removal and masking also need Docker (IOPaint/LaMa,
CPU). These tools don't touch the scene — they prepare photos for `scene-gen`.

| Tool | Parameters | Notes |
|---|---|---|
| `detect_objects` | `image`, `targets` | Text-phrase detection (Workers AI moondream); returns normalized bboxes per target: `{target: [{x_min,y_min,x_max,y_max}]}` |
| `prep_image` | `image`, `target?`, `margin?`, `remove?`, `output?` | One-stop prep: optionally inpaint away `remove` clutter phrases (e.g. `["bicycle"]`), detect `target` (default `"the building"`), crop with margin (default 0.06). Writes `{stem}-prep.jpg`, mask/cleaned intermediates, and a `-prep.json` report to `output` (default `tmp/prep`) |
| `inpaint_image` | `image`, `mask`, `output?` | LaMa inpainting (IOPaint in Docker); mask: white = remove, same dimensions as image. Returns cleaned path |

Feed the cropped `-prep.jpg` into `scene-gen` (references/generation.md).

## Viewer interaction (needs scene-api running)

| Tool | Parameters | Notes |
|---|---|---|
| `get_viewer_logs` | — | The browser console, relayed through scene-api |
| `get_viewer_photos` | — | List photos the user uploaded in the viewer |
| `get_viewer_photo` | `id` | One photo as image content — you can *see* it |
| `set_photo_camera` | `id`, `position`, `target`, `fov?` | Guess where the photo was taken; renders as a frustum |
| `clear_photo_camera` | `id` | Remove the pose |

## Ids

Node ids are arena ids for the *current session*; they change across
export/import. For durable identity use `part_id` (your naming) or
`scene_id` (stable across round trips). `get_node` returns `scene_id` when
present.

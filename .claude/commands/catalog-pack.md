# Catalog Pack

Add a new sprite pack to the library: inspect its contents, determine correct metadata, move it to the right folder, and add it to CATALOG.json.

**Arguments:** $ARGUMENTS — path to the new pack folder (relative to sprites-new/ or absolute)

## Steps

1. **Resolve the pack path.** If $ARGUMENTS is relative, treat it as relative to `/home/kaliuzhnyi/asocity/sprites-new/`. If no argument given, list untracked folders in sprites-new/ (i.e. paths not in CATALOG.json) and ask which one to process.

2. **Inspect the pack contents:**
   - Run `find <pack_path> -name "*.png" | head -40` to sample PNG filenames
   - Run `find <pack_path> -name "*.png" | wc -l` to get total PNG count
   - Check for `README*`, `LICENSE*`, or `*.txt` files and read them briefly
   - Note the folder name (this becomes the pack ID suffix and stays as-is)

3. **Determine metadata** based on filenames, folder name, and README:
   - **source**: `craftpix` / `kenney` / `itch` / `other`
   - **type[]**: one or more of: `tileset`, `character`, `enemy`, `vehicle`, `background`, `object`, `effect`, `ui`
   - **view[]**: `side`, `top-down`, `isometric`, `any`
   - **theme[]**: use values from this list — cave, city, cyberpunk, dark, desert, fantasy, forest, generic, jungle, ocean, platformer, racing, sci-fi, sky, snow, space, tropical, military, nature, any, varied
   - **style[]**: `pixel`, `cartoon`, `flat`
   - **png_root**: if PNGs are in a subdirectory (not spread across the whole pack), set to the relative path of that subdir from sprites-new/

4. **Compute the correct destination path** using these rules:
   - Primary type priority (first match): `tileset > character > enemy > vehicle > background > object > effect > ui`
   - Theme folder: first non-generic theme from the list; if none, use `generic`
   - For `tileset` or `vehicle`: path = `{type}s/{view}/{theme}/{pack_folder_name}`
     - view = `top-down` if top-down, `isometric` if isometric, else `side`
   - For all others: path = `{type}s/{theme}/{pack_folder_name}`
   - Note: `ui` type uses folder `ui/` (not `uis/`), `enemy` uses `enemies/`

5. **Confirm with user** — print a summary:
   ```
   Pack:    <folder_name>
   ID:      <proposed_id>
   Source:  <source>
   Type:    <type[]>
   View:    <view[]>
   Theme:   <theme[]>
   Style:   <style[]>
   PNGs:    <png_count>
   Dest:    sprites-new/<new_path>
   ```
   Ask: "Looks correct? (y to proceed, or describe corrections)"

6. **Execute** (after confirmation):
   - Create destination directory if needed: `mkdir -p /home/kaliuzhnyi/asocity/sprites-new/<new_path_parent>`
   - Move the folder: `mv <current_path> /home/kaliuzhnyi/asocity/sprites-new/<new_path>`
   - Build the new CATALOG.json entry:
     ```json
     {
       "id": "<id>",
       "source": "<source>",
       "category": "<primary_type>",
       "path": "<new_path>",
       "png_root": "<new_path>/<subdir>",   // only if needed
       "png_count": <n>,
       "type": [...],
       "view": [...],
       "theme": [...],
       "style": [...]
     }
     ```
   - Append the entry to CATALOG.json (maintain alphabetical order by `id`):
     ```python
     python3 -c "
     import json
     with open('/home/kaliuzhnyi/asocity/sprites-new/CATALOG.json') as f:
         cat = json.load(f)
     new_entry = <entry_dict>
     cat.append(new_entry)
     cat.sort(key=lambda e: e['id'])
     with open('/home/kaliuzhnyi/asocity/sprites-new/CATALOG.json', 'w') as f:
         json.dump(cat, f, indent=2, ensure_ascii=False)
         f.write('\n')
     "
     ```
   - Print the final entry and confirm it was added.

## Reference: Folder structure

```
sprites-new/
  backgrounds/{theme}/
  characters/{theme}/
  enemies/{theme}/
  tilesets/{top-down|side|isometric}/{theme}/
  objects/{theme}/
  effects/{theme}/
  vehicles/{top-down|side}/{theme}/
  ui/{theme}/
```

Theme folders: cave, city, dark, desert, fantasy, forest, generic, jungle, ocean, platformer, racing, sci-fi, sky, snow, space, tropical, military

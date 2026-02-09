1. If FBX embed their textures everything is good
2. If not then try file/External Data/Find Missing Files
3. If didn't work try it manually with this script

```python
import bpy
import os

TEXTURE_DIR = "/Users/iggysleepy/blender-art/assets/All/Models/Textures/"

# Build a lookup of actual files on disk (lowercase name -> full path)
texture_files = {}
for f in os.listdir(TEXTURE_DIR):
    if f.lower().endswith(('.jpg', '.jpeg', '.png', '.tga', '.bmp', '.tiff')):
        texture_files[f.lower()] = os.path.join(TEXTURE_DIR, f)

fixed = 0
failed = []

for mat in bpy.data.materials:
    if not mat.use_nodes:
        continue
    for node in mat.node_tree.nodes:
        if node.type != 'TEX_IMAGE':
            continue
        if node.image and node.image.has_data:
            continue  # already loaded fine, skip
        
        # Get the name the node expects
        if node.image:
            img_name = node.image.name
        else:
            continue
        
        # Remove .001 .002 suffixes Blender adds
        clean_name = img_name
        while clean_name and clean_name[-1].isdigit() and '.' in clean_name:
            parts = clean_name.rsplit('.', 1)
            if parts[-1].isdigit():
                clean_name = parts[0]
            else:
                break
        
        # Try to find matching file
        lookup = clean_name.lower()
        if lookup in texture_files:
            filepath = texture_files[lookup]
            new_img = bpy.data.images.load(filepath)
            node.image = new_img
            fixed += 1
            print(f"FIXED: {mat.name} -> {clean_name}")
        else:
            failed.append((mat.name, img_name))
            print(f"NOT FOUND: {mat.name} wants '{img_name}' (cleaned: '{clean_name}')")

print(f"\n--- Done ---")
print(f"Fixed: {fixed}")
print(f"Not found: {len(failed)}")
if failed:
    print("Missing textures:")
    for mat_name, img_name in failed:
        print(f"  {mat_name} -> {img_name}")
```



After you are done:
- Select the object
- File → Export → FBX
- In export settings: check **"Selected Objects"**
- **Path Mode → Copy** and click the **small box icon** next to it (embed textures)
- Export

With textures embedded, Unity will just work — drag the FBX into your project and materials/textures come with it.

One gotcha: if multiple models share the same material/texture (which is likely with this pack since they probably use a shared atlas), Unity will create duplicate texture copies per FBX. Not a big deal for a small game, but if it bothers you later, you can extract materials in Unity and point them all to one shared texture.
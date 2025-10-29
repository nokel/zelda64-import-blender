# zelda64-import-blender

This is a Blender addon allowing to import models and animations from files extracted from N64 Zelda roms.

**Required Blender version: 3 (tested with 3.4.1)**

If an import doesn't work the messages in the system console may help understand why an import is failing.

# Limitations

Not sure why but the animations don't work exactly as expected, the textures are also importing a bit weird, and some of the models will appear far away from the widget, which can easily be fixed by resetting the model pose.

# Data from other segments

Data from other segments will be loaded from files named `segment_XX.zdata` located in the same directory as the imported file, if any, where `XX` is the segment number in hex.

For segment 2 (scene segment) data will load from `XXX_scene.zscene` assuming the imported file is named like `XXX_room.*`, or from `segment_02.zdata`, or from any `.zscene` file, trying in that order.

# History

## SoulofDeity

Originally written by SoulofDeity, they did most of the work. Their code can be found in this repository's history or 
[on google code](https://code.google.com/archive/p/sods-blender-plugins/)
([http url](http://code.google.com/archive/p/sods-blender-plugins/)).

## Nokaubure

Then, Nokaubure distributed a version with following added features:
- Textures tags `#MirrorX/Y` and `#ClampX/Y` for exporting again using SharpOcarina
- Importing vertex alpha
- A `displaylists.txt` file alongside the imported file is read for display lists to import (made by Skawo)
- If the imported file is named like `XXX_room.*`, data for segment 2 will first tried to be read from `XXX_scene.zscene`
- Scale map models by 48 (it turned out this was just cancelling out the 1/48 scale SoulofDeity implemented)
- Set Blender's 3D view grid size parameters for maps
- Made pre-rendered scenes able to import

## Dragorn421

Then, Dragorn421 made changes to import all animations at once instead of only one every import, and other smaller changes, changelog for these versions was:

**Import Options**
- Added option to use shadeless materials (don't use environment colors in-game) (default False)
- Added original object scale option, blender objects will be at inverted scale (default 100, use 48 for maps?)
- Made Texture Clamp and Texture Mirror options True by default
- Removed option for animation to import

**Features**
- Made all animations import at once, each action named animXX_FF where XX is some number and FF the number of frames
- Removed the (useless) actions every mesh part had
- Made armature modifier show in edit mode by default
- Made end frame (in Timeline) be the maximum duration of animations
- Improved console log a tiny bit (open console before importing to see progress)

**Remarks**
- Importing normals only lacks ability to apply them reliably (Blender limitations), currently data is lost when merging doubles, so in this version normals are not used at all

Later, these additional changes were made:
- Support 0xE1 commands for importing display lists, used by level-of-detail stuff (thanks Nokaubure for the explanations)
- Introduced logging
- Handle importing an object with several hierarchies (skeletons) better (thanks Nokaubure for pointing that out)

At this point, version number was introduced to be 2.0

## Nokel

Nokel has updated the code to work with the Blender 3 API

### Property Annotations
**Before (2.79)**:
```python
class IMPORT_OT_zelda64(bpy.types.Operator):
    my_prop = StringProperty(name="Property")
```

**After (3.4.1)**:
```python
class IMPORT_OT_zelda64(bpy.types.Operator):
    my_prop: StringProperty(name="Property")
```

### Class Naming & bl_idname
**Before**: `class ImportZ64`, `bl_idname = "file.zobj2020"`  
**After**: `class IMPORT_OT_zelda64`, `bl_idname = "import_scene.zelda64"`

- Class renamed to follow UPPERCASE_PREFIX naming convention
- bl_idname updated to follow proper naming pattern

### Registration System
**Before (2.79)**:
```python
def register():
    registerLogging()
    bpy.utils.register_module(__name__)
    bpy.types.INFO_MT_file_import.append(menu_func_import)
```

**After (3.4.1)**:
```python
classes = (
    IMPORT_OT_zelda64,
)

def register():
    registerLogging()
    for cls in classes:
        bpy.utils.register_class(cls)
    bpy.types.TOPBAR_MT_file_import.append(menu_func_import)
```

- Removed `bpy.utils.register_module()` (deprecated)
- Implemented explicit class registration
- Updated menu type: `INFO_MT_file_import` → `TOPBAR_MT_file_import`

### Scene & Object API Updates
**Before**: `bpy.context.scene.objects.active = obj`  
**After**: `bpy.context.view_layer.objects.active = obj`

- Updated 3 instances of scene.objects.active

### Bone Selection API
**Before**: `bone.select = True`  
**After**: `bone.select_set(True)`

- Updated 6 instances of bone selection

### Viewport Display
**Before**: `area.spaces.active.viewport_shade = "TEXTURED"`  
**After**: `area.spaces.active.shading.type = 'SOLID'`

- Updated viewport shading API

### Material Properties
The following material properties still work in 3.4.1 but are legacy:

**Commented/Noted**:
- `mtl.use_shadeless` - Removed in 2.80+, needs shader nodes
- `mtl.use_transparency` - Deprecated but functional
- `mtl.game_settings.alpha_blend` - Removed (game engine removed in 3.0)
- `mtl.texture_slots` - Deprecated but still functional

### 10. UI Updates
- Updated `layout.label()` calls to use `text=` parameter

---

## Backward Compatibility

**This updated addon is NOT backward compatible with Blender 2.79.**

---

## Installation in Blender 3.4.1

1. Open Blender 3.4.1
2. Go to `Edit` → `Preferences` → `Add-ons`
3. Click `Install...`
4. Select `io_import_z64.py`
5. Enable the addon by checking the box next to "Import-Export: Zelda64 Importer"
6. The import option will appear under `File` → `Import` → `Zelda64 (.zobj;.zroom;.zmap)`

---

## Files Modified

- **io_import_z64.py**: Main addon file (migrated to 3.4.1)
- **README.md**: Added Blender 3.4.1 development section

---

### Installing the Addon in Blender (Local Use)

1. Download `io_import_z64.py` from this repository
2. Open Blender 3
3. Navigate to `File` → `User Preferences` → `Add-ons`
4. Click `Install Add-on from File...`
5. Select `io_import_z64.py`
6. Enable the addon by checking its checkbox
7. Save user settings

### Using the Addon in Blender

1. Open Blender's system console: `Window` → `Toggle System Console`
2. Go to `File` → `Import` → `Zelda64 (.zobj, .zmap, etc.)`
3. Browse to your extracted Zelda 64 ROM files
4. Configure import options and click Import
5. Monitor import progress in the system console

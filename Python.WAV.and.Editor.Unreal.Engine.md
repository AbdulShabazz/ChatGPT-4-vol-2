## OpenAI ChatGPT v1.2025.330 Python.WAV.Unreal.Engine.md

**Q: Can I use Python to wire WAV audio file to a blueprint class and Metasound in Unreal Engine content folder and or in Editor?**

**A:** Yes. In Unreal Engine, Python can (1) import a `.wav` into `/Game/...` as a `SoundWave`, (2) create/modify a `MetaSound Source` graph via the MetaSound Builder API, and (3) assign the resulting `SoundWave`/`MetaSound Source` into a Blueprint’s default properties. This is primarily an **Editor** workflow; Python is not available in a packaged runtime build by default.

Editor scope constraints

* Python asset import/creation targets the **Content Browser** (assets under `/Game/...`), i.e., project content folders. ([Epic Games Developers][1])
* MetaSound authoring can be procedural via the Builder API (Blueprint/C++ and exposed to Python). ([Epic Games Developers][2])
* Serializing a builder to an actual MetaSound asset uses the MetaSound Editor Subsystem (Editor-only). ([Epic Games Developers][3])

Minimal Python pattern (Editor)

1. Import WAV → SoundWave asset

```python
import unreal

def import_wav_to_game(wav_path: str, dest_path="/Game/Audio", dest_name=None) -> unreal.SoundWave:
    task = unreal.AssetImportTask()
    task.filename = wav_path
    task.destination_path = dest_path
    task.destination_name = dest_name or ""
    task.automated = True
    task.replace_existing = True
    task.save = True

    unreal.AssetToolsHelpers.get_asset_tools().import_asset_tasks([task])
    if not task.imported_object_paths:
        raise RuntimeError(f"Import failed: {wav_path}")

    asset_path = task.imported_object_paths[0]
    sound = unreal.load_asset(asset_path)
    if not isinstance(sound, unreal.SoundWave):
        raise TypeError(f"Imported asset is not SoundWave: {asset_path}")
    return sound
```

2. Build a MetaSound Source that plays that SoundWave

Core APIs:

* Create a source builder: `unreal.MetaSoundBuilderSubsystem.create_source_builder(...)` ([Epic Games Developers][4])
* Add/connect nodes: `unreal.MetaSoundBuilderBase.add_node_by_class_name(...)`, `connect_nodes(...)`, etc. ([Epic Games Developers][5])
* Save to `/Game/...`: `unreal.MetaSoundEditorSubsystem.build_to_asset(...)` ([Epic Games Developers][3])
* Node class naming: discover the “full class name” by holding **Shift** while hovering the node name in the MetaSound Editor. ([Epic Games Developers][2])
* Class-name struct: `unreal.MetasoundFrontendClassName(namespace, name, variant)` ([Epic Games Developers][6])

Skeleton (you fill in the node class names you actually use, e.g., the Wave Player node’s class name from Shift-hover):

```python
import unreal

def build_metasound_source_from_soundwave(
    sound: unreal.SoundWave,
    asset_name="MS_FromWav",
    package_path="/Game/Audio/MetaSounds",
    player_node_namespace="",
    player_node_name="",
    player_major_version=1,
):
    builder_subsys = unreal.get_engine_subsystem(unreal.MetaSoundBuilderSubsystem)
    editor_subsys  = unreal.get_editor_subsystem(unreal.MetaSoundEditorSubsystem)

    builder, on_play_out, on_finished_in, audio_out_ins, result = builder_subsys.create_source_builder(
        unreal.Name("TmpBuilder"),
        output_format=unreal.MetaSoundOutputAudioFormat.STEREO,
        is_one_shot=True,
    )
    # Add a “player” node (Wave Player / similar)
    cls = unreal.MetasoundFrontendClassName(
        namespace=unreal.Name(player_node_namespace),
        name=unreal.Name(player_node_name),
        variant=unreal.Name(""),
    )
    player_node, result = builder.add_node_by_class_name(cls, player_major_version)

    # Find the player node input that takes the SoundWave (name depends on node; use find_node_input())
    # node_in, result = builder.find_node_input(player_node, unreal.Name("WaveAsset"))

    # Set the default literal to the SoundWave object
    # lit = builder_subsys.create_object_meta_sound_literal(sound)  # :contentReference[oaicite:8]{index=8}
    # builder.set_node_input_default(node_in, lit)                  # :contentReference[oaicite:9]{index=9}

    # Connect player audio output to graph audio output(s) (names depend on node/output format)
    # player_out, result = builder.find_node_output(player_node, unreal.Name("Audio"))
    # builder.connect_nodes(player_out, audio_out_ins[0])

    metasound_asset, out_result = editor_subsys.build_to_asset(
        builder=builder,
        author="Python",
        asset_name=asset_name,
        package_path=package_path,
        template_sound_wave=sound,  # copies settings like attenuation/modulation/sound class :contentReference[oaicite:10]{index=10}
    )
    return metasound_asset
```

3. “Wire” the SoundWave / MetaSound into a Blueprint class

Typical Editor approach: set a Blueprint variable’s default on the generated class’s Class Default Object (CDO). Asset references can be direct or soft (for load control). ([Epic Games Developers][7])

```python
import unreal

def assign_asset_to_blueprint_default(bp_path: str, property_name: str, asset: unreal.Object):
    bp = unreal.load_asset(bp_path)
    if not isinstance(bp, unreal.Blueprint):
        raise TypeError(f"Not a Blueprint: {bp_path}")

    cdo = unreal.get_default_object(bp.generated_class)
    cdo.set_editor_property(property_name, asset)

    unreal.BlueprintEditorLibrary.compile_blueprint(bp)
    unreal.EditorAssetLibrary.save_loaded_asset(bp)
```

Notes that avoid common failures

* Blueprint property must already exist and be type-compatible (e.g., `SoundBase` can accept a `MetaSoundSource`; `SoundWave` is narrower).
* MetaSound graph automation is feasible, but you must use the correct MetaSound node class names and pin names; Epic’s Builder API explicitly expects native node classes to be addressed by class name and recommends Shift-hover to discover them. ([Epic Games Developers][2])
* Editing an existing serialized MetaSound asset is done via the editor subsystem “find/begin building” style flow (Editor-only per documentation). ([Epic Games Developers][2])

If you paste (a) your Unreal Engine version, (b) the MetaSound node you want (Wave Player vs another), and (c) the Blueprint variable type/name you want to set, a concrete end-to-end script can be provided without placeholders.
```
[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/importing-audio-files?utm_source=chatgpt.com "Importing Audio Files | Unreal Engine 5.7 Documentation"
[2]: https://dev.epicgames.com/documentation/en-us/unreal-engine/metasound-builder-api-in-unreal-engine "MetaSound Builder API in Unreal Engine | Unreal Engine 5.7 Documentation | Epic Developer Community"
[3]: https://dev.epicgames.com/documentation/en-us/unreal-engine/python-api/class/MetaSoundEditorSubsystem?application_version=5.3 "unreal.MetaSoundEditorSubsystem — Unreal Python 5.3 (Experimental) documentation"
[4]: https://dev.epicgames.com/documentation/en-us/unreal-engine/python-api/class/MetaSoundBuilderSubsystem?application_version=5.3 "unreal.MetaSoundBuilderSubsystem — Unreal Python 5.3 (Experimental) documentation"
[5]: https://dev.epicgames.com/documentation/en-us/unreal-engine/python-api/class/MetaSoundBuilderBase?application_version=5.3 "unreal.MetaSoundBuilderBase — Unreal Python 5.3 (Experimental) documentation"
[6]: https://dev.epicgames.com/documentation/en-us/unreal-engine/python-api/class/MetasoundFrontendClassName?application_version=5.6&utm_source=chatgpt.com "Unreal Python 5.6 (Experimental) documentation"
[7]: https://dev.epicgames.com/documentation/en-us/unreal-engine/referencing-assets-in-unreal-engine?utm_source=chatgpt.com "Referencing Assets in Unreal Engine"
```

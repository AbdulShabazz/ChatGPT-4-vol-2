## OpenAI ChatGPT v1.2025.336 Common.Unreal.Engine.Packaging.Errors.md

**Q: What are the primary packaging error when trying to ship an unreal engine game?**

**A:** **Primary Unreal Engine packaging errors (Win64 focus).**

1. **Missing or invalid target/platform**

* Symptom: “No target found”, “Unknown platform”.
* Cause: Platform SDK not installed; wrong Target.cs; Shipping target not generated.
* Fix: Install platform support; regenerate project files; verify `<ProjectName>Target.cs` and `<ProjectName>EditorTarget.cs`.

2. **C++ compile failures during packaging**

* Symptom: Build fails only in Shipping/Development.
* Cause: Code relying on editor-only modules, macros, or includes; warnings-as-errors in Shipping.
* Fix: Guard editor code with `#if WITH_EDITOR`; move editor modules to Editor-only targets; fix warnings.

3. **Editor-only assets referenced at runtime**

* Symptom: Cook failure referencing `/Editor/` paths.
* Cause: Runtime Blueprints or materials referencing editor utilities, widgets, or plugins.
* Fix: Remove editor references; split assets into Runtime vs Editor folders; disable editor plugins for shipping.

4. **Cook failures (missing assets / redirectors)**

* Symptom: “Unable to find package”, “Failed to load”.
* Cause: Deleted or moved assets; unresolved redirectors.
* Fix: Fix up redirectors; resave assets; run full cook; verify soft references.

5. **Plugin incompatibility**

* Symptom: Plugin fails to load in Shipping.
* Cause: Plugin marked Editor-only; unsupported platform; missing binaries.
* Fix: Disable for target platform; rebuild plugin; update `.uplugin` platform filters.

6. **Invalid packaging settings**

* Symptom: Immediate failure or empty build.
* Cause: Wrong map list; “Cook only maps” misconfigured; missing Game Default Map.
* Fix: Set Game/Editor Default Maps; define Maps to Cook; verify packaging profile.

7. **Path length / illegal characters**

* Symptom: Windows file copy or cook errors.
* Cause: Deep project paths; non-ASCII or special characters.
* Fix: Shorten project path; enable long paths in Windows; sanitize names.

8. **Out-of-date Derived Data Cache (DDC)**

* Symptom: Random shader/material cook errors.
* Cause: Corrupt or stale cache.
* Fix: Clear local/shared DDC; rebuild shaders.

9. **Shader compile failures**

* Symptom: Material compilation errors during cook.
* Cause: Platform-incompatible shader code; feature level mismatch.
* Fix: Check material quality switches; validate feature level; recompile shaders.

10. **Config (.ini) errors**

* Symptom: Runtime crash on launch.
* Cause: Invalid config values; missing input mappings; bad subsystem settings.
* Fix: Validate Default*.ini; test clean config; diff against working build.

11. **Packaging permissions / antivirus interference**

* Symptom: Access denied; file lock errors.
* Cause: Antivirus quarantining build steps; insufficient permissions.
* Fix: Exclude project/build directories; run with proper permissions.

12. **Incorrect build configuration**

* Symptom: Works in Development, fails in Shipping.
* Cause: Assertions/logging stripped; reliance on debug-only behavior.
* Fix: Test Shipping early; remove debug dependencies; add runtime checks.

If you want, specify platform (Win64, Linux, consoles, Android, iOS), engine version, and whether the project is Blueprint-only or C++; I can narrow this to the highest-probability failures and exact fixes.

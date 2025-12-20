## OpenAI ChatGPT v1.2025.343 UMG.Font.Integration.md

**Q: In Unreal Engine for Blueprints, I've assigned a font to my custom data asset, `DA_Typography_Console`. Now how do I use it in the UMG workflow?**

**A:** Use `DA_Typography_Console` as the single source of truth for `FSlateFontInfo` (or `FTextBlockStyle`) and apply it to UMG widgets by (a) setting the font on each widget at construct-time, or (b) binding a style provider that returns the correct font/style. The cleanest Blueprint-only pattern is a small “Typography” Blueprint Function Library that converts your data asset entries into `Slate Font Info` and pushes them into Text widgets (and any components that accept `SlateFontInfo`).

UMG can only consume fonts via:

* A widget property that is an `FSlateFontInfo` (TextBlock/Button text/etc.).
* A `Slate Widget Style` (TextBlockStyle/ButtonStyle) inside a `Slate Style Set` (mostly C++), or exposed style structs.

Blueprint workflow (recommended)

1. Ensure your data asset exposes the right types
   In `DA_Typography_Console`, store either:

* `FSlateFontInfo` directly (best for UMG), or
* `Font` (Font Object) + `Size` + `TypefaceName` + `Outline` data, then build an `FSlateFontInfo` in Blueprint.

Practical struct for the asset:

* `Font` (Font Object / Font Face)
* `Size` (int)
* `Typeface` (Name) (optional)
* `LetterSpacing` (int) (optional, UE version-dependent)
* `OutlineSize` (int) + `OutlineColor` (optional)

2. Create a Blueprint Function Library: `BFL_Typography`
   Add functions like:

* `GetConsoleFontInfo(DA_Typography_Console, StyleTag) -> SlateFontInfo`
* `ApplyConsoleTypography(TextWidget, DA_Typography_Console, StyleTag)` (takes `TextBlock` reference)

Implementation notes (Blueprint):

* Inputs: `DA_Typography_Console` (your asset), `StyleTag` (Name or Enum like Regular/Medium/Bold/Title/etc.).
* Retrieve the entry from the data asset (Map lookup or switch).
* If you stored `FSlateFontInfo`, return it.
* If you stored pieces, use “Make SlateFontInfo” (or “Set Members in SlateFontInfo”) to assemble.

3. Apply at runtime in UMG
   In your widget blueprint (e.g., `WBP_HUD`, `WBP_Console`, etc.):

* Add a variable: `Typography` (type: `DA_Typography_Console`), set to your asset in defaults.
* On `Event PreConstruct` (so designer preview works) and/or `Event Construct`:

  * Call `ApplyConsoleTypography` for each relevant TextBlock.
  * Example sequence:

    * `ApplyConsoleTypography(TB_Title, Typography, Title)`
    * `ApplyConsoleTypography(TB_Body, Typography, Body)`
    * `ApplyConsoleTypography(TB_Caption, Typography, Caption)`

If you need to apply to many TextBlocks:

* Put TextBlocks in a common container (VerticalBox/Overlay).
* Use `Get All Widgets of Class` is global and not ideal.
* Prefer storing references in an array variable `TypographyTargets` and loop.

4. Buttons and other widgets

* `Button` text is usually a nested `TextBlock` inside the button content. Apply to that `TextBlock`.
* `RichTextBlock` uses a `Data Table` of `Rich Text Styles` (different system). If you need rich text, create a style table where each row uses your same font asset and keep it aligned with `DA_Typography_Console`.

5. Make it scalable: Enum-based style keys
   Define an enum `EConsoleTypographyStyle` (e.g., Title, H1, H2, Body, Mono, Caption).

* Use it as the lookup key in the data asset (Map `EConsoleTypographyStyle -> FSlateFontInfo`).
* Avoid string/name keys; enums keep refactors safe.

Common gotchas

* Use a Font Asset / Font Face that is packaged with the project (not an OS-installed font reference).
* If you see fallback fonts in shipping builds, the font asset is not being cooked or referenced strongly. Keeping it referenced in the data asset (and the widget defaults) usually fixes cooking.
* `PreConstruct` runs in editor preview; guard logic if you have runtime-only dependencies, but setting fonts is safe.

Minimal “do this now” steps

* Store `FSlateFontInfo` entries in `DA_Typography_Console` keyed by an enum.
* Add `BFL_Typography::GetFontInfo(DA, Style)` and `Apply(TextBlock, DA, Style)`.
* In each widget’s `PreConstruct`, call `Apply` for each TextBlock.

If you paste (1) how `DA_Typography_Console` is structured (fields/types) and (2) which UE version (4.27 vs 5.x), I can give the exact Blueprint node sequence (“Make SlateFontInfo” vs “Set Members in SlateFontInfo”, outline support, letter spacing availability).

**Q: `DA_Typography_Console` is structured as described in earlier discussion and the Unreal Editor version is 5.7. Proceed!**

**A:** Blueprint-only UMG hookup (UE 5.7): build an `FSlateFontInfo` from `DA_Typography_Console` and push it into each Text widget via `Set Font` (run in `PreConstruct` for designer preview and `Construct` for runtime). UMG consumes fonts through `FSlateFontInfo` on widgets like `TextBlock`. ([Epic Games Developers][1])

1. Data Asset shape (assumed from your prior description)
   `DA_Typography_Console`

* `Map<EConsoleTypographyStyle, FTypographyEntry>`
  `FTypographyEntry`
* `Font` (Font Object / Font asset)
* `Size` (int)
* `TypefaceName` (Name)
* `OutlineSize` (int)
* `OutlineColor` (Linear Color)
* `LetterSpacing` (int, optional; only if exposed/used in your project)

2. Blueprint Function Library
   Create `BFL_Typography`.

Function A: `GetConsoleFontInfo`

* Inputs: `DA_Typography_Console` (object ref), `Style` (EConsoleTypographyStyle)
* Output: `Slate Font Info` (FSlateFontInfo)

Node sequence:

* `Map Find` on `DA->Styles` with `Style` → `Entry` (FTypographyEntry)
* `Make Font Outline Settings` (FFontOutlineSettings)

  * `Outline Size` = `Entry.OutlineSize`
  * `Outline Color` = `Entry.OutlineColor`
  * Leave other pins default unless you explicitly need them (fill/alpha/shadow coupling varies per use case)
* `Make SlateFontInfo` (FSlateFontInfo)

  * `Font` = `Entry.Font`
  * `Size` = `Entry.Size`
  * `Typeface Font Name` = `Entry.TypefaceName` (if you use composite faces)
  * `Outline Settings` = output of `Make Font Outline Settings`
  * If `Letter Spacing` is available on your node/build, set it from `Entry.LetterSpacing`; otherwise omit.

Function B: `ApplyConsoleTypography_TextBlock`

* Inputs: `TextBlock` (ref), `DA_Typography_Console` (ref), `Style` (enum)
* Body:

  * Call `GetConsoleFontInfo` → `FontInfo`
  * `TextBlock -> Set Font` with `In Font Info = FontInfo`

(You can add siblings for `RichTextBlock` later; it uses a style table rather than `Set Font`.)

3. Use inside a Widget Blueprint (UMG)
   In `WBP_*`:

* Add variable `Typography` (type `DA_Typography_Console`), set default to your `DA_Typography_Console` asset.
* Ensure each TextBlock you want to drive is checked `Is Variable` (Designer panel).
* Event graph:

  * `Event PreConstruct`

    * Call `ApplyConsoleTypography_TextBlock` for each target TextBlock (Title/Body/etc.).
  * Optionally repeat in `Event Construct` if you swap typography at runtime (language/theme).

4. Packaging/cooking guardrails

* Ensure the font is a project asset (Font / Font Face / Composite Font), not an OS-installed-only reference.
* Keeping the font referenced by the data asset and by widgets that reference the data asset generally ensures it cooks correctly; runtime-cached fonts are the typical path for Slate/UMG. ([Epic Games Developers][2])

5. Known edge case
   If you observe font size not updating when using `MakeSlateFontInfo` + `Set Font` in some engine versions, the workaround pattern is still “make the TextBlock a variable and call `Set Font` in the graph,” as shown in community repro steps. ([Epic Developer Community Forums][3])

If you want, paste your exact `FTypographyEntry` member names/types and I’ll mirror them 1:1 into a concrete node recipe (including default/fallback behavior when `Map Find` misses).
```
[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/using-fonts-with-umg-in-unreal-engine?utm_source=chatgpt.com "Using Fonts with UMG in Unreal Engine"
[2]: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/UFont?utm_source=chatgpt.com "UFont | Unreal Engine 5.7 Documentation"
[3]: https://forums.unrealengine.com/t/bug-report-ue5-4-1-umg-makeslatefontinfo-font-size/1864136?utm_source=chatgpt.com "[Bug Report, UE5.4.1] UMG, MakeSlateFontInfo Font Size - UI"
```
**Q: Explain the `TextBlock` input parameter further and its expected reference type. I'm having difficulty recognizing its exact data type.**

**A:** `TextBlock` in that helper function is a reference to a live UMG widget instance of class `UTextBlock` (Blueprint type: `Text Block` → **Object Reference**).

What it is

* Runtime instance: a `UTextBlock*` created when your `UserWidget` is constructed.
* Not a “font” type and not a `Slate Font Info` type.
* Not a class/type descriptor: do not use `Text Block Class Reference`.

How to recognize the exact type in Blueprints

* In a Widget Blueprint (Designer), select the TextBlock and enable `Is Variable`.
* In the Graph, it appears in `My Blueprint` as a variable named like `TB_Title`.
* Hover it: it will show `Text Block` (Object Reference).
* Drag it into the graph: the pin type is `Text Block Object Reference` (category: UMG).

How to declare the function input correctly (Blueprint Function Library)

* Input parameter type should be:

  * `Text Block` (Object Reference)
* Not:

  * `Text Block Class Reference`
  * `Widget` (Object Reference) unless you plan to cast
  * `Slate Font Info` (that is what you *set*, not what you *set it on*)

How you pass it

* From within the same Widget Blueprint:

  * Drag `TB_Title` (the TextBlock variable) into the graph and wire it into the function input.
* From outside the widget:

  * You cannot directly access child widgets unless you expose them (e.g., via a public getter function or by marking variables public) or apply typography inside the widget itself (preferred).

If you want the helper to accept “any widget”

* Use input type `Widget` (Object Reference), then `Cast To TextBlock`, and only call `Set Font` on success.
* Alternatively, create overloads for `TextBlock` and `RichTextBlock` (they do not share the same font-setting API).

Common confusion

* `Text Block` (Object Reference) = instance you can call `Set Font` on.
* `Text Block Class Reference` = a class pointer used for spawning/typing, cannot call instance methods like `Set Font`.

**Q: Perhaps you mean `TextBox` as its reference type? Such as `TextBox`, `RichTextBlock`, TextBox (Multilline)` -- these are valid object references in UE 5.7!**

**A:** No. `TextBlock` and `TextBox` are different UMG (Unreal Motion Graphics) widget classes with different intended use and (sometimes) different styling APIs.

What I meant by `TextBlock`

* UMG Designer widget: **Text** (Palette → Common → Text).
* Blueprint reference type: **Text Block (Object Reference)**.
* C++ class: `UTextBlock`.
* Typical node you call: `Set Font` (input: `Slate Font Info`).

What a `TextBox` is

* UMG Designer widget: **Editable Text Box** (single-line input) or **Multi-Line Editable Text Box** (multi-line input).
* Blueprint reference types:

  * `Editable Text Box (Object Reference)` (`UEditableTextBox`)
  * `Multi-Line Editable Text Box (Object Reference)` (`UMultiLineEditableTextBox`)
* These are input controls; they have additional style/state (focused/hovered) beyond just a font.

Rich text is separate again

* Widget: **RichTextBlock** (`URichTextBlock`).
* Uses a **text style set** / style rows, not just “set a font on the widget” in the same way as `TextBlock`.

Practical implication for your typography helper

* If your target widgets are display-only labels, use **TextBlock** and pass `Text Block (Object Reference)` into the function.
* If you need editable console input, you likely want **Editable Text** / **Editable Text Box**; then create a second helper that takes that widget type and sets its font (and optionally its full style).

Recommended Blueprint Function Library signatures (separate overloads)

* `ApplyTypography_TextBlock(Text Block Object Reference, DA_Typography_Console, Style)`
* `ApplyTypography_EditableTextBox(Editable Text Box Object Reference, DA_Typography_Console, Style)`
* `ApplyTypography_MultiLineEditableTextBox(Multi-Line Editable Text Box Object Reference, DA_Typography_Console, Style)`
* `ApplyTypography_RichTextBlock(Rich Text Block Object Reference, …)` (usually assigns a Style Set / row-based style, not just `Slate Font Info`)

How to confirm the type you have in your widget

1. In the Widget Blueprint Designer, select the widget.
2. In Details, check the class name at the top (Text, Editable Text Box, RichTextBlock, etc.).
3. Enable `Is Variable`.
4. In the Graph, hover the variable: it will show `Text Block`, `Editable Text Box`, etc.

If you tell me which specific widgets you’re using for the console (display lines vs input line), I can give the exact node sequence for each (TextBlock vs EditableTextBox vs RichTextBlock) using your `DA_Typography_Console` entries.

**Q: I will default to `RichTextBlock`, instead. There is no `TextBlock`; 'Text` is derived from STextBlock which is a simple type.**

**A:** `Text` in the UMG (Unreal Motion Graphics) palette is the UMG wrapper class `UTextBlock` (it wraps Slate’s `STextBlock`). The Blueprint reference type is **Text Block (Object Reference)**, even if the palette label is just “Text.” ([Epic Games Developers][1])

If you are standardizing on `RichTextBlock`, the correct UMG workflow is “style set + markup,” not “per-widget Set Font.”

RichTextBlock typography pipeline (UE 5.7)

1. Create a style set DataTable

* Create a **Data Table** asset.
* Row Struct: **Rich Text Style Row** (`FRichTextStyleRow`).
* Add rows named after your styles (example: `Body`, `Title`, `Prompt`, `Warn`, `Error`).
* For each row, set `TextStyle` (an `FTextBlockStyle`), including `Font` (`FSlateFontInfo`), size, outline, color, etc. (This is the canonical RichText style source.) ([Epic Games Developers][2])

2. Assign the style set to each RichTextBlock

* In the widget designer: select the `RichTextBlock` → set **Text Style Set** to that DataTable. ([Unreal Engine][3])
* Or at runtime (Blueprint): call `SetTextStyleSet(NewTextStyleSet)` on the `RichTextBlock`. ([Epic Games Developers][4])

3. Use markup in the RichTextBlock text
   Example `SetText` string:

* `<Title>Console</>\n<Prompt>> </><Body>help</>\n<Warn>Unknown command</>`
  Row names map to tags. RichTextBlock parses and applies the row’s `TextStyle`. ([Epic Games Developers][2])

How `DA_Typography_Console` fits

* If you want the Data Asset (DA) to remain the “source of truth,” store a `UDataTable*` reference inside `DA_Typography_Console` (e.g., `ConsoleRichTextStyleSet`) and point all widgets to it (designer assignment) or call `SetTextStyleSet(DA->ConsoleRichTextStyleSet)` in `PreConstruct/Construct`. `URichTextBlock` exposes `SetTextStyleSet` in UE 5.7. ([Epic Games Developers][4])
* If you also need runtime theme swaps: call `SetTextStyleSet`, then `RefreshTextLayout()` (optional), and re-`SetText`. ([Epic Games Developers][4])

When DA-only (no DataTable) is sufficient

* If you only need *one* style for the entire RichTextBlock, you can skip tags and set the default style via:

  * `SetDefaultFont(FSlateFontInfo)` or
  * `SetDefaultTextStyle(FTextBlockStyle)` ([Epic Games Developers][4])
    This does not provide per-span styling.

If you paste your enum/style keys (e.g., `Regular/Medium/Bold/Warn/Error`) I can give an exact mapping recipe: DataTable row names + RichText tags + which fields from `DA_Typography_Console` should populate `FSlateFontInfo` vs `FTextBlockStyle`.
```
[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/UMG/UTextBlock "UTextBlock | Unreal Engine 5.7 Documentation | Epic Developer Community"
[2]: https://dev.epicgames.com/documentation/en-us/unreal-engine/umg-rich-text-blocks-in-unreal-engine?utm_source=chatgpt.com "UMG Rich Text Block"
[3]: https://www.unrealengine.com/en-US/tech-blog/advanced-text-styling-with-rich-text-block?utm_source=chatgpt.com "Advanced Text Styling with Rich Text Block"
[4]: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/UMG/URichTextBlock "URichTextBlock | Unreal Engine 5.7 Documentation | Epic Developer Community"
```
**Q: Can I specify `TextBlock` as an `Object (Reference)` type?**

**A:** Yes. In Blueprints, `TextBlock` is an object type (`UTextBlock`), so the correct parameter type is:

* `Text Block` → `Object Reference`

How to set it:

* In your function (Blueprint Function Library or Widget BP function), add an input.
* In the input’s type dropdown, search for `Text Block`.
* Choose `Text Block` (it will show as an object reference type, not a value type).

Notes:

* This only works if the widget you pass is actually a `Text` widget (UMG “Text”), i.e., a `UTextBlock` instance.
* Do not use `Object` (generic) unless you plan to `Cast To TextBlock` inside the function.
* `Text Block Class Reference` is not correct for calling `Set Font`; it is for class typing/spawning, not instance mutation.

**Q: Notice I can set `TextBlock` type on the Output but not on the input!**

**A:** You are selecting the wrong “Text” type on the input.

In UE Blueprints there are two different types that display as “Text”:

* `Text` (pink pin) = `FText` value type (localized string)
* `Text` / `Text Block` (blue pin) = `UTextBlock` object reference (UMG widget)

Your screenshot shows:

* Input `TextBlock` is currently **pink** → `FText`
* Output `AsText` is **blue** → `UTextBlock` (UMG “Text” widget)

Fix

1. In the function signature panel, click the type dropdown for the `TextBlock` input (currently pink `Text`).
2. In the picker, search `Text`.
3. Choose the entry that is an **Object Reference** (blue icon). It may be labeled:

   * `Text Block` (Object Reference), or
   * `Text` (Object Reference) (UMG widget wrapper)

If you still cannot find it in that picker

* Set the input type to `Widget` (Object Reference), then keep your existing `Cast To TextBlock` and call `Set Font` on the cast result.
* This is also the more general pattern if you later want to support `RichTextBlock`, `EditableTextBox`, etc.

Fastest workaround (forces correct type)

* In a Widget Blueprint, drag a `Text` widget variable (UMG “Text”) into the graph to get a blue `Text Block` pin.
* Copy that pin’s type by making a new function input and using the same type selection (it will appear as the blue object type in the picker).

**Q: FText object types are the only choice in the blueprint function class editor.**

**A:** Your input pin is currently in the **Structure/Text (FText)** category, so the picker is only showing **struct types** (hence “Text”, “Anim Pose Context”, “Text Box Style”, etc.). You are not in the “Object Reference” category yet.

Fix (Blueprint Function Library function input)

1. Select the function in the left panel (so the right “Inputs/Outputs” details show up).
2. Under **Inputs**, find your `TextBlock` parameter.
3. Click the **colored type icon** (the small pink pin/box next to the type). Do not click the dropdown list you showed.
4. In the category menu, change the pin category from:

   * `Text` (pink, `FText`)
     to:
   * `Object Reference` (blue)
5. After switching to `Object Reference`, a class picker appears. Search and select:

   * `Text Block` (UMG widget, `UTextBlock`) if you want the “Text” widget, or
   * `Rich Text Block` (`URichTextBlock`) if that is your target.

Why output “works”

* Your output pin is already an **Object Reference** (blue), so it can pick `Text Block`/`RichTextBlock`.
* Your input pin is still a **struct** (`FText`), so it only offers structs.

Fallback if you want one function to accept multiple widget types

* Make the input type `Widget` (Object Reference), then `Cast To TextBlock` / `Cast To RichTextBlock` internally and apply the appropriate API per type.

**Q: These are my options.**

**A:** `UTextBlock` is exposed in the picker as **`Text` (Object Reference)**, not as “Text Block”.

In your screenshot you are searching `text block`, so the picker only returns types whose display name contains that substring (styles + RichTextBlock types). The plain UMG Text widget is named “Text”, so it will not match.

Do this:

1. Keep the parameter type category as **Object Reference** (blue).
2. In the class picker, search **`Text`** (not `text block`).
3. Select **`Text`** under Object Types (this is the UMG “Text” widget; class `UTextBlock`).

If you want one function that works for both `Text` and `RichTextBlock`:

* Make the input type **`Widget` (Object Reference)**.
* Inside the function:

  * `Cast To TextBlock` → apply `Set Font`
  * `Cast To RichTextBlock` → apply `SetTextStyleSet` / `SetDefaultTextStyle` (RichText uses style sets, not per-span Set Font).

**Q: How can you assert that based upon the past two images. I provided the populated list box result for when `Text` was entered! In the Blueprint function class editor, it doesn't exist! Please provide a workaround, is what I really desire.**

**A:** Workaround: do not type the input as `UTextBlock` in the Blueprint Function Library signature. Accept a generic `Widget` (or `Object`) input, then `Cast` inside the function and apply the correct API per widget type.

This avoids the type-picker limitation you are hitting.

Option A (recommended): `Widget` input + internal casts

1. Function signature (Blueprint Function Library)

* Inputs:

  * `DA_Typography_Console` (your data asset)
  * `TargetWidget` : `Widget` (Object Reference)  ← pick `Widget`, not `Text`
  * `Style` : your enum
* Outputs: none (this is a mutator)

2. Function graph

* `IsValid(TargetWidget)` → Branch
* `Cast To TextBlock` (Object = TargetWidget)

  * On Success:

    * Build `Slate Font Info` from `DA_Typography_Console` + `Style`
    * Call `Set Font` on the cast result
    * Return
* Cast failed:

  * `Cast To RichTextBlock` (Object = TargetWidget)

    * On Success:

      * If you want multi-style tags: call `SetTextStyleSet(DataTable)` (store the `DataTable` in the DA)
      * If you want one style only: call `SetDefaultTextStyle(TextBlockStyle)` or `SetDefaultFont(SlateFontInfo)` (whichever is available on your node set)
    * Return

Notes:

* `Widget` (Object Reference) is the UMG base class, so it will accept `Text`, `RichTextBlock`, `EditableTextBox`, etc.
* Your earlier `Cast To TextBlock` node already proves the engine can resolve `UTextBlock` at graph time; the limitation is specifically the function input type picker.

Option B: keep your current `Object` input (also works)

If you can’t (or don’t want to) use `Widget`:

* Input: `TargetObject : Object (Reference)`
* Same internal logic: `Cast To TextBlock` → `Set Font`, else `Cast To RichTextBlock` → apply style set/default style.

Option C: stop passing child widgets; apply inside the widget

If your goal is “apply typography to a widget tree”:

* Put the apply function inside a base `UserWidget` (e.g., `WBP_TypographyBase`) and have HUD widgets inherit.
* Inside that widget, you can reference `Text` / `RichTextBlock` variables directly and call `Set Font` / `SetTextStyleSet` without ever needing a `UTextBlock`-typed function parameter.

If you paste the fields in `DA_Typography_Console` (map key + value struct), I’ll provide the exact node chain to construct `FSlateFontInfo` and (for `RichTextBlock`) the minimal `DataTable`-backed style set hookup.

**Q: Is this the correct wiring layout for `ApplyConsoleTypography`?**

**A:** Control-flow is broadly correct (cast → apply), but the data wiring into `Make Slate Font Info` is not.

What is correct in your graph

* `TargetWidget (Object)` → `Cast To TextBlock` (success path) → apply font.
* `Cast Failed` → `Cast To RichTextBlock` → apply font (you are using `Set Default Font`, which is valid if you are not using per-span Rich Text tags).

What is incorrect (critical)

* The `Font Object` pin on `Make Slate Font Info` must be a **Font/FontFace/CompositeFont asset reference**, not:

  * the widget reference (`As Text Obj Ref`), and not
  * the data asset reference (`DA Typography Console Obj Ref`).

Right now, your `Make Slate Font Info` nodes appear to be fed from variables named like widget/DA refs. Even if the graph compiles, this is not selecting a font entry from your data asset; it will not produce the intended typography.

Workable wiring layout (minimal changes)

1. Keep the function input `TargetWidget` as `Widget` (Object Reference) or `Object` (Object Reference).

2. Immediately do:

   * `Cast To TextBlock` (Object = TargetWidget)

     * Success: `ApplyFontToTextBlock(CastResult)`
     * Cast Failed: `Cast To RichTextBlock` (Object = TargetWidget)

       * Success: `ApplyFontToRichTextBlock(CastResult)`
       * Failed: return

3. Replace the `Switch on E_FontStyle` + multiple `Make Slate Font Info` duplicates with a single “GetFontInfo” step:

   * `Map Find` on `DA_Typography_Console->Styles` using `Style` (enum)
   * From the found `Entry`, wire:

     * `Entry.Font` → `Make Slate Font Info.Font Object`
     * `Entry.Size` → `Make Slate Font Info.Size`
     * `Entry.TypefaceName` → `Make Slate Font Info.Typeface Font Name` (if used)
     * `Entry.Outline` → `Outline Settings` (optional)

4. Feed the returned `Slate Font Info` into:

   * `TextBlock -> Set Font (In Font Info)`
   * `RichTextBlock -> Set Default Font (In Font Info)` (your current approach)

If you actually intend to use RichText tags (`<Body>...</>` etc.)

* Do not rely on `Set Default Font` alone.
* Instead: store a `DataTable` of `RichTextStyleRow` in the DA and call `RichTextBlock -> Set Text Style Set(DataTable)`; tags then select rows. `Set Default Font` only affects the default style, not tagged spans.

Debug checks

* Before casting: `TargetWidget -> Get Class -> Get Display Name` (Print String) to confirm what you are receiving.
* Put a breakpoint on `Cast To TextBlock` and inspect the Object.

If you paste the member names inside your `DA_Typography_Console` style entry struct (exact field names/types), I can mirror the exact node chain for the `Map Find → Make Slate Font Info` portion so the pins match 1:1.

**Q: Okay well why did I create the data asset, `DA Typography Console`, input pin if I have to destructure it in this context? Why can I apply the asset?**

**A:** You cannot “apply a Data Asset” to a UMG widget because UMG styling APIs accept **value structs** (primarily `FSlateFontInfo`, `FTextBlockStyle`, or a RichText style DataTable), not arbitrary objects. A `UDataAsset` is only a container; it is not a style payload type that UMG knows how to consume directly.

What the `DA_Typography_Console` pin is for

* It is a **handle to your style database** (fonts, sizes, outline, etc.).
* At the moment of application you must convert (“destructure”) the selected entry into the struct that the widget expects.

This is analogous to: you do not plug a “database” into a renderer; you query it to produce render parameters.

How to avoid repeated destructuring
Centralize the conversion once, then reuse.

Pattern 1 (TextBlock / EditableText): “GetFontInfo” function

* `GetConsoleFontInfo(DA, StyleEnum) -> FSlateFontInfo`
* `ApplyConsoleTypography(DA, TargetWidget, StyleEnum)` just calls `GetConsoleFontInfo` then `Set Font` / `Set Default Font`.

Your DA pin stays meaningful: it selects the data source, but the conversion is done in one place.

Pattern 2 (RichTextBlock): store a Style Set DataTable in the DA
If you want RichText (tagged spans) as the default:

* In `DA_Typography_Console`, add `ConsoleRichTextStyleSet : DataTable` (RowStruct = `RichTextStyleRow`).
* Then “applying” becomes a single call:

  * `RichTextBlock -> SetTextStyleSet(DA.ConsoleRichTextStyleSet)`
    No per-style destructuring per widget; the DataTable rows carry `FTextBlockStyle` which carries `FSlateFontInfo`.

This is the closest thing to “apply the asset” in UMG using Blueprints.

Why UE doesn’t allow direct DA application

* `UTextBlock::SetFont` expects `FSlateFontInfo` (a struct).
* `URichTextBlock` expects either a `DataTable` style set or default `FTextBlockStyle`/`FSlateFontInfo`.
* There is no generic interface like `ApplyStyle(UObject*)` on these widgets, so a `UDataAsset` cannot be consumed without conversion.

Recommended decision for your console

* If you are standardizing on `RichTextBlock` with multiple styles: put the **RichText style DataTable** on the DA and apply via `SetTextStyleSet`.
* If you only need a single style per widget: keep the DA map and use `GetConsoleFontInfo` once, then `Set Default Font` / `Set Font`.

If you want, describe whether your console needs (a) multi-style spans within one line, or (b) one style per widget, and I’ll give the minimal graph layout for the chosen pattern.

**Q: How do I destructure the custom data asset?**

**A:** In Blueprints, “destructuring” a `UDataAsset` means: get the asset instance, read one of its UPROPERTY fields, then (optionally) read a sub-struct or map entry and feed those values into `Make Slate Font Info` / `Make TextBlockStyle`.

Procedure (generic, works for any custom DA)

1. Drag off the `DA_Typography_Console` pin

* You will see “Get” accessors for the DA’s exposed variables (the UPROPERTY fields you defined).
* Example: `Get Styles`, `Get FontMap`, `Get Regular`, etc. (names match your DA variables).

2. If you stored a Map keyed by an enum
   Typical DA layout:

* `Styles` : `Map<E_FontStyle, FTypographyEntry>`

Node chain:

* `DA_Typography_Console` → `Styles` (get)
* `Find` (Map) with Key = `Style` (your enum input)

  * Outputs:

    * `Found` (bool)
    * `Value` (FTypographyEntry)

3. Break the returned struct

* From `Value` pin:

  * `Break FTypographyEntry` (or “Split Struct Pin”)
  * You now have `Font`, `Size`, `TypefaceName`, `OutlineSize`, etc.

4. Re-pack into the struct UMG wants
   TextBlock path:

* `Make Slate Font Info`

  * `Font Object` = `Font` from entry (font asset)
  * `Size` = `Size`
  * `Typeface Font Name` = `TypefaceName` (if used)
  * `Outline Settings` = `Make Font Outline Settings` from outline fields (optional)
* `TextBlock -> Set Font` (In Font Info = result)

RichTextBlock (single default style) path:

* Same `Make Slate Font Info`
* `RichTextBlock -> Set Default Font` (In Font Info = result)

RichTextBlock (tagged multi-style) path (preferred)

* Store `StyleSetDataTable : DataTable` in the DA
* Apply with one node:

  * `DA_Typography_Console -> StyleSetDataTable`
  * `RichTextBlock -> Set Text Style Set (StyleSetDataTable)`

5. Handle missing keys

* Use the `Found` bool from `Map Find`.
* If not found:

  * Use a default entry (e.g., `Body`) or return early.

Notes specific to your earlier wiring issue

* The `Font Object` pin on `Make Slate Font Info` must be the font asset from the DA entry (e.g., `Entry.Font`), not the widget reference and not the DA reference.

If you paste the exact variable names in `DA_Typography_Console` (and the struct name for the map value), I can spell the node names exactly as they will appear (e.g., `Find` vs `Find (Map)`; `Break <StructName>` fields).

**Q: What do I specify for font info and or font name? `DA_Typography_Console` is a meta data asset.**

**A:** You must supply an actual **font asset reference** in `FSlateFontInfo` (UMG cannot render from “metadata-only”). The “font name” is not the font file name; it is the **typeface entry name inside the font asset** (only relevant if you use a composite font with multiple faces).

What to set in `Make Slate Font Info`

1. `Font Object` (required)
   Set this to a project font asset, typically one of:

* **Composite Font / Font asset** created in the Content Browser (preferred for shipping + fallback).
* **Font Face** asset (single face), if you are using per-face assets.

Practical rule:

* If your DA is “metadata,” it must still **reference** the real font asset(s). Store a `Font Object` (or `Soft Object Reference` to a Font/FontFace) per style entry.

2. `Typeface Font Name` (optional)
   Only set this if your `Font Object` is a **composite font** that contains multiple faces (e.g., Regular/Medium/Bold) under named entries.

* If `Font Object` = Composite Font with faces named `Regular`, `Medium`, `Bold`:

  * `Typeface Font Name` = `Regular` / `Medium` / `Bold` (must exactly match the face key in the font asset).
* If `Font Object` = a single **Font Face** asset:

  * Leave `Typeface Font Name` as `None` / default.

3. `Size`
   Set to the point size (int) you want.

4. Outline (optional)
   Build `Font Outline Settings` and plug it into `Outline Settings`.

How to make your “metadata” DA usable
Define your per-style struct like:

* `FontObject` : **Font / FontFace Object Reference** (or Soft Object Reference)
* `TypefaceName` : `Name` (only if using composite font)
* `Size` : int
* `OutlineSize`, `OutlineColor`, etc.

Then your apply function does:

* `Entry = MapFind(DA.Styles, StyleEnum)`
* `Make Slate Font Info(FontObject=Entry.FontObject, TypefaceFontName=Entry.TypefaceName, Size=Entry.Size, …)`
* `TextBlock:SetFont(FontInfo)` or `RichTextBlock:SetDefaultFont(FontInfo)`

If you do not currently have any font assets to reference

* Import `.ttf/.otf` into the Content Browser.
* Create **Font Face** assets (one per file/weight), then (optionally) a **Composite Font** that groups them with face keys (`Regular`, `Bold`, etc.).
* Reference that asset from the DA.

If you paste what you currently stored in `DA_Typography_Console` for “font” (its type), I can tell you whether it is already a valid `Font Object` for `FSlateFontInfo` or needs to be changed to a Font/FontFace asset reference.

**Q: I don't like this workflow. The entire point of `DA_Typography_Console` is to have to specify `HUD`, `HUD BODY`, `NUMERIC`, or `ALERT`!**

**A:** You can keep the workflow “pick HUD / HUD_BODY / NUMERIC / ALERT” and never touch font assets at call sites. The font assets must exist somewhere (UMG ultimately needs a real font), but you can encapsulate them entirely inside `DA_Typography_Console` and expose only a semantic key.

Two workable patterns.

A) RichTextBlock-first (best if you chose RichTextBlock)

Goal: you set one property once, then your text uses tags like `<HUD>…</>` / `<ALERT>…</>`.

1. Create a DataTable `DT_ConsoleRichTextStyles` with row struct `RichTextStyleRow`.

   * Row names: `HUD`, `HUD_BODY`, `NUMERIC`, `ALERT`.
   * Each row’s `TextStyle` contains the complete `FTextBlockStyle` including `Font` (`FSlateFontInfo`), size, outline, color, etc.
2. In `DA_Typography_Console`, add:

   * `StyleSet` : `DataTable` (reference to `DT_ConsoleRichTextStyles`)
3. In each `RichTextBlock`, set **Text Style Set** = `DA_Typography_Console.StyleSet` (designer), or in `PreConstruct` call:

   * `RichTextBlock -> SetTextStyleSet(DA.StyleSet)`
4. Author text with tags:

   * `<HUD>MISSION</>\n<HUD_BODY>Eliminate target</>\n<ALERT>LOW HEALTH</>`

Result: the only “selection” is the tag name (HUD, ALERT). No per-widget font wiring.

B) “Drop-in styled text widget” (works for TextBlock or RichTextBlock, no tags required)

Goal: you place `WBP_StyledText` instead of a raw Text widget; you pick a style enum in Details.

1. Create enum `EConsoleTypographyStyle`: `HUD`, `HUD_BODY`, `NUMERIC`, `ALERT`.
2. In `DA_Typography_Console`, store:

   * `Map<EConsoleTypographyStyle, FTextBlockStyle>` (or your own struct that can build `FTextBlockStyle`)
3. Create `WBP_StyledText` with:

   * `Style` (EConsoleTypographyStyle, ExposeOnSpawn, InstanceEditable)
   * `Typography` (DA_Typography_Console, default asset)
   * Internal widget: `TextBlock` or `RichTextBlock`
4. In `PreConstruct`:

   * `StyleValue = Typography.Styles[Style]`
   * If internal is TextBlock: set `Font`, `ColorAndOpacity`, `Shadow`, etc from `StyleValue`
   * If internal is RichTextBlock: call `SetDefaultTextStyle(StyleValue)`

Result: in UMG you only set `Style = HUD_BODY` per instance; the DA contains all font assets/sizes/colors.

Key point

* You do not “apply the data asset” directly because UMG consumes `FSlateFontInfo` / `FTextBlockStyle` / RichText style tables, not arbitrary `UDataAsset`. The asset is your semantic registry; the widget still needs a resolved style struct, but you can hide that behind a single dropdown.

If you tell me whether you want tags (Pattern A) or “no tags, just a style dropdown per widget” (Pattern B), I’ll give the exact node list for UE 5.7 for the apply step you should standardize on (`SetTextStyleSet` vs `SetDefaultTextStyle` vs `SetFont + SetColorAndOpacity`).

**Q: These attributes already exist as font .uasset files linked to `DA_Typography_Console`.**

**A:** If `DA_Typography_Console` already holds the font `.uasset` references, the only “destructuring” you should be doing is a single lookup by semantic key, then passing the resulting struct to the widget. You can make this effectively “apply the asset” by storing the *UMG-consumable structs* directly in the data asset.

Best workflow (no per-field wiring): store `FSlateFontInfo` or `FTextBlockStyle` in the DA

1. In `DA_Typography_Console`, change (or add) one of these maps:

A. Text-only (smallest)

* `Map<EConsoleTypographyStyle, SlateFontInfo>` (value type `FSlateFontInfo`)

B. Full text styling (recommended)

* `Map<EConsoleTypographyStyle, TextBlockStyle>` (value type `FTextBlockStyle`)

2. Blueprint Function Library

`GetTypography(DA, Style) -> SlateFontInfo` (or `TextBlockStyle`)

* `DA -> Styles (Map)`
* `Find` with `Style`
* Return the value directly

3. Apply

Text (UMG “Text”, class `UTextBlock`)

* If you store `FSlateFontInfo`: `TextBlock -> SetFont(FontInfo)`
* If you store `FTextBlockStyle`: set `Font`, `ColorAndOpacity`, `Shadow`, etc. from the style (UMG does not always expose a single “SetStyle” for `UTextBlock`, so you may set 2–4 properties, but still not per-field from the DA entry—just “Break TextBlockStyle” once).

RichTextBlock

* If you store `FSlateFontInfo`: `RichTextBlock -> SetDefaultFont(FontInfo)`
* If you store `FTextBlockStyle`: `RichTextBlock -> SetDefaultTextStyle(TextStyle)`

This yields the UX you want: pick `HUD`, `HUD_BODY`, `NUMERIC`, `ALERT` (enum), and everything else is automatic.

If you must keep the DA as “metadata fields” (Font asset + Size + Outline…)
You still do not apply the asset directly; you do a single map lookup and a single `Make Slate Font Info`:

* `Find(DA.Map, Style)` → `Entry`
* `Make Slate Font Info`

  * `Font Object` = `Entry.FontAsset` (your `.uasset`)
  * `Size` = `Entry.Size`
  * `Typeface Font Name` = `Entry.TypefaceName` (only if composite font)
  * `Outline Settings` = built from `Entry.Outline*`
* Apply to widget (`SetFont` / `SetDefaultFont`)

But if your `.uasset` references are already in the DA, the cleanest fix is to store `FSlateFontInfo` (or `FTextBlockStyle`) directly so you do not rebuild it at all.

If you paste the current value struct type inside `DA_Typography_Console` (field names/types), I will tell you exactly which of the two refactors above is minimal (often it is just changing the map value type to `SlateFontInfo` and migrating existing fields once).

**Q: Yes, indeed, perhaps this is what `DA_Typography_Console`, `DA_Typography_PC`, and `DA_Typography_Mobile` blueprint instances were trying to accomplish.**

**A:** Yes. The intended pattern is:

* One **base Data Asset class** (e.g., `DA_Typography`), containing a semantic map: `HUD`, `HUD_BODY`, `NUMERIC`, `ALERT`, etc.
* Multiple **platform instances** (`DA_Typography_Console`, `_PC`, `_Mobile`) that provide different resolved styles (different font assets, sizes, outlines) while preserving the same semantic keys.

Make it “one dropdown + apply” by storing UMG-ready structs in the DA and selecting the platform DA once.

Recommended structure

1. Enum
   `EConsoleTypographyStyle`:

* `HUD`
* `HUD_BODY`
* `NUMERIC`
* `ALERT`
* … (your set)

2. DA base class fields
   Option A (minimal, works everywhere):

* `Styles_FontOnly : Map<EConsoleTypographyStyle, FSlateFontInfo>`

Option B (better, includes color/shadow/etc.):

* `Styles_Text : Map<EConsoleTypographyStyle, FTextBlockStyle>`

3. Application API (Blueprint Function Library)

`ApplyTypography(TargetWidget, TypographyDA, StyleKey)`

Implementation:

* `Map Find` → returns `FSlateFontInfo` or `FTextBlockStyle`
* `Cast To TextBlock`:

  * If `FSlateFontInfo`: `Set Font`
  * If `FTextBlockStyle`: `Set Font` + `Set Color and Opacity` (+ shadow if you store it)
* Else `Cast To RichTextBlock`:

  * If `FSlateFontInfo`: `Set Default Font`
  * If `FTextBlockStyle`: `Set Default Text Style`

No other pins at call sites; you only pass:

* `TypographyDA` (Console/PC/Mobile)
* `StyleKey` (HUD/HUD_BODY/NUMERIC/ALERT)
* `TargetWidget`

How to select the correct platform DA once

A) In a top-level UI manager widget (or HUD widget) set:

* `ActiveTypography : DA_Typography` variable

On construct:

* Console build → assign `DA_Typography_Console`
* PC build → assign `DA_Typography_PC`
* Mobile build → assign `DA_Typography_Mobile`

Then every widget receives `ActiveTypography` (or reads it from a `GameInstance`/UI Subsystem variable) and applies styles by key.

If you are standardizing on RichTextBlock for the console
Use this instead for multi-style spans:

* In the DA store `RichTextStyleSet : DataTable` (row struct `RichTextStyleRow`)
* Apply once: `RichTextBlock.SetTextStyleSet(DA.RichTextStyleSet)`
* Use tags `<HUD>...</>` / `<ALERT>...</>` etc.

This is the most “semantic-only” workflow: the style key lives in the markup, and the platform DA swaps the entire style set.

Minimal next step

* Change your map value type inside the DA to `FSlateFontInfo` (or add a parallel map) and populate `HUD/HUD_BODY/NUMERIC/ALERT` with the font `.uasset` references + size.
* Update `ApplyConsoleTypography` to do exactly one `Map Find` and apply.

If you want, paste a screenshot of your `DA_Typography_Console` variables panel (field names and types). I will specify the exact map type and the exact node chain to replace your current switch/multiple “Make Slate Font Info” blocks.

**Q: I'm shipping a game (intended) with Roboto typeface. Should I activate/enable true the monospace flag?**

**A:** No. Do not enable the monospace flag for Roboto unless you deliberately want monospaced layout (console/code look) and have a monospaced Roboto variant.

Details

* Roboto is proportional by default; enabling a monospace/“fixed width” behavior will force equal advance widths and typically degrades readability and spacing for UI body text and headings.
* If you need monospaced output (numbers aligning, console logs, timers), use an actual monospaced face:

  * Roboto Mono (preferred if you want “Roboto-family” consistency)
  * Otherwise another monospaced UI font.

Recommended policy for your typography assets

* `HUD`, `HUD_BODY`, `ALERT`: monospace flag = false.
* `NUMERIC`: use proportional by default; switch to a monospaced font only if you require column alignment (scores, tables) or stable-width digits.
* If your goal is stable-width digits without full monospace, prefer fonts with **tabular numerals** (if exposed via your font asset/features). In Unreal, the simplest reliable approach is still a dedicated monospaced numeric style (e.g., Roboto Mono) for those fields.

Practical test

* If you see jitter in values like `99 → 100` or timers, switch just that style key (`NUMERIC`) to Roboto Mono; keep everything else proportional.

**Q: What is font material? Is it a required field or does it pertain only to custom fonts?**

**A:** `Font Material` is an optional `MaterialInterface` used to shade the text glyph quads when Slate/UMG renders them. It is not required, and it is not limited to “custom fonts.” ([Epic Games Developers][1])

What it does

* Overrides the default text draw material so you can implement UI text effects (gradient, glow, scanline, dissolve, animated UVs, etc.).
* It is referenced by `FSlateFontInfo.FontMaterial` (per-font/style) and can also be overridden on `RichTextBlock` via `SetDefaultMaterial`. ([Epic Games Developers][1])

When you should set it

* Only when you need shader-driven effects beyond color/shadow/outline.
* Otherwise leave it unset (`None`) and use normal text rendering.

RichTextBlock specifics

* `URichTextBlock` exposes `SetDefaultMaterial(UMaterialInterface*)` and `GetDefaultDynamicMaterial()` for runtime parameter animation (Material Instance Dynamic (MID)). ([Epic Games Developers][2])

Important constraint

* If you enable “material is stencil” behavior (`bMaterialIsStencil`) you are responsible for stenciling the text inside the material (typically using the font signed-distance field node/path); otherwise leave it off. ([Epic Games Developers][1])
```
[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/SlateCore/FSlateFontInfo "FSlateFontInfo | Unreal Engine 5.7 Documentation | Epic Developer Community"
[2]: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/UMG/URichTextBlock "URichTextBlock | Unreal Engine 5.7 Documentation | Epic Developer Community"
```
**Q: Ahh, but I can supply or use the FontMaterial from my custom material uasset to create a shimmer effect!**

**A:** Yes. A `FontMaterial` can be used to implement a shimmer on UMG text; it is applied during text rendering via `FSlateFontInfo.FontMaterial` (per style/font info) or via `URichTextBlock::SetDefaultMaterial`, and you can animate it with a `MaterialInstanceDynamic` (MID). ([Epic Games Developers][1])

Blueprint workflow (RichTextBlock)

1. Create a UI material for text

* Material Domain: **User Interface**
* Blend Mode: typically **Translucent**
* Preserve glyph coverage (alpha) and only modulate color/brightness inside the glyph.
* Add parameters (examples): `ShimmerSpeed`, `ShimmerWidth`, `ShimmerStrength`, `ShimmerPhase`.

Core idea inside the material:

* BaseColor = incoming text color (commonly `VertexColor`)
* GlyphMask = font coverage (alpha)
* ShimmerMask = moving band computed from screen/UV coordinates and time
* OutColor = BaseColor * lerp(1.0, 1.0 + ShimmerStrength, ShimmerMask)
* OutOpacity = GlyphMask

2. Attach the material to the RichTextBlock
   Pick one:

A) Single material for the entire block

* `RichTextBlock -> SetDefaultMaterial(M_FontShimmer_UI)` ([Epic Games Developers][2])

B) Per-style material (recommended if you already use `HUD/HUD_BODY/ALERT`)

* In your RichText style DataTable, each row’s `TextStyle.Font` is an `FSlateFontInfo` which contains `FontMaterial`.
* Set `FontMaterial` per row (e.g., only `ALERT` shimmers). ([Epic Games Developers][1])

3. Animate it (two options)

A) Blueprint-driven MID (precise control; pause-safe)

* On `Construct`:

  * `MID = RichTextBlock -> GetDefaultDynamicMaterial()` ([Epic Games Developers][3])
  * Store `MID`
* On `Tick` (or a timer):

  * `MID -> SetScalarParameterValue("ShimmerPhase", TimeSeconds * ShimmerSpeed)`

B) Material `Time` node (zero Blueprint)

* Use `Time` in the material to drive shimmer phase.
* Less control over pause/state; simplest wiring.

Notes / constraints

* This is not “custom fonts only.” It works with Roboto and any runtime-cached UMG font; the material is separate from the font asset. ([Epic Games Developers][1])
* Avoid `bMaterialIsStencil` unless you specifically know you need stencil behavior; it changes expectations on how the material masks glyphs. ([Epic Games Developers][1])
* If you need the shimmer to sweep across the whole block uniformly (not per-glyph), compute the band in **screen space** (ScreenPosition) rather than glyph UV space.

If you paste your material graph (or list the parameters you created), I can give the exact node math for a band-mask shimmer that brightens glyphs by +50% inside the moving stripe.
```
[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/SlateCore/FSlateFontInfo?utm_source=chatgpt.com "FSlateFontInfo - API/Runtime"
[2]: https://dev.epicgames.com/documentation/en-us/unreal-engine/python-api/class/RichTextBlock?application_version=5.4&utm_source=chatgpt.com "unreal.RichTextBlock — Unreal Python 5.4 (Experimental) ..."
[3]: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/UMG/URichTextBlock?utm_source=chatgpt.com "URichTextBlock | Unreal Engine 5.7 Documentation"
```
**Q: Better yet, font uassets in lieu of specifying a direct font type in `DA_Typogrraphy` supports typeface and a fallback font. This provides more platform control.**

**A:** Correct. A Font `.uasset` (typically a Composite Font) is the right abstraction boundary for platform control because it encapsulates:

* Typeface families (Regular/Bold/Italic or named faces)
* Fallback chain (secondary fonts for missing glyphs)
* (Optionally) per-platform substitution if you swap the font asset via `DA_Typography_Console/PC/Mobile`

Recommended `DA_Typography` design

1. Keep semantic keys

* `EConsoleTypographyStyle`: `HUD`, `HUD_BODY`, `NUMERIC`, `ALERT`, etc.

2. Store *resolved font asset + face key*, not raw TTF metadata
   Per-style entry struct (example):

* `Font` : Font Object reference (your Font `.uasset`, ideally Composite)
* `TypefaceName` : Name (e.g., `Regular`, `Medium`, `Bold`) matching the face key inside the Composite Font
* `Size` : int
* Optional: `OutlineSettings`, `FontMaterial`, `LetterSpacing` (if exposed)

3. Build `FSlateFontInfo` at apply-time (single function)

* `Entry = Styles[StyleKey]`
* `Make Slate Font Info`

  * `Font Object = Entry.Font`
  * `Typeface Font Name = Entry.TypefaceName` (or None if not using multiple faces)
  * `Size = Entry.Size`
  * (Optional) `Font Material = Entry.FontMaterial`
  * (Optional) outline

UMG then consumes the `FSlateFontInfo` normally (`Text`/`UTextBlock`: `SetFont`, `RichTextBlock`: `SetDefaultFont`).

Why this is better than “direct font face” references

* Centralizes fallback behavior per platform (your Console/PC/Mobile DAs can point at different Composite Fonts with different fallbacks).
* Allows language expansion later (CJK, Cyrillic, symbols) by updating the font asset fallback chain without touching widgets.
* Keeps style selection semantic (HUD/HUD_BODY/etc.) while the asset decides the actual typefaces.

Caveats

* `TypefaceName` must exactly match the face key inside the Composite Font; otherwise it will fall back to the default face.
* Ensure fallback fonts are actually referenced by the Composite Font asset so they cook/package.
* If you want RichText per-span style (`<ALERT>...</>`), you still need either:

  * a RichText style DataTable (rows contain `FTextBlockStyle` → `Font` is `FSlateFontInfo` referencing your Composite Font), or
  * accept “single default style per RichTextBlock” via `SetDefaultFont/SetDefaultTextStyle`.

Net: keep `DA_Typography_*` as the semantic-to-CompositeFont resolver; only ever pass `StyleKey` at call sites.

**Q: This is what I intended.**

**A:** Implement it as: `DA_Typography_*` maps semantic keys to a `Composite Font` `.uasset` + `TypefaceName` + size (+ optional outline/material), and your apply function converts that entry to `FSlateFontInfo` and sets it on the target widget.

Minimal, stable setup

1. Data types

* Enum: `ETwitterTypographyKey` = `HUD`, `HUD_BODY`, `NUMERIC`, `ALERT`, …

* Struct: `FTypographySpec`

  * `Font` : Font Object reference (Composite Font `.uasset`)
  * `TypefaceName` : Name (e.g., `Regular`, `Medium`, `Bold`)
  * `Size` : int
  * Optional: `OutlineSettings` (or outline fields), `FontMaterial`

* Data Asset: `DA_Typography`

  * `Specs : Map<ETwitterTypographyKey, FTypographySpec>`

Platform instances:

* `DA_Typography_Console`, `DA_Typography_PC`, `DA_Typography_Mobile` (same keys, different `Font`/sizes/fallbacks)

2. One Blueprint function
   `ApplyTypography(TargetWidget (Widget/Object Ref), TypographyDA, Key)`

Graph (core):

* `Find` in `TypographyDA.Specs` by `Key` → `Spec`
* `Make Slate Font Info`:

  * `Font Object = Spec.Font`
  * `Typeface Font Name = Spec.TypefaceName`
  * `Size = Spec.Size`
  * (Optional) `Font Material = Spec.FontMaterial`
  * (Optional) `Outline Settings = …`
* Cast target:

  * `Cast To TextBlock` → `SetFont(FontInfo)`
  * else `Cast To RichTextBlock` → `SetDefaultFont(FontInfo)` (or `SetDefaultTextStyle` if you store `FTextBlockStyle`)

3. Authoring rules

* Keep `TypefaceName` identical across platform font assets (so keys remain stable).
* Put fallback fonts into the Composite Font asset (not in Blueprints).
* For shimmer: use `FontMaterial` in `FTypographySpec` and apply only for keys like `ALERT`.

This preserves your intended UX: designers pick only `HUD/HUD_BODY/NUMERIC/ALERT`; platform control is achieved by swapping the `DA_Typography_*` instance, not changing widgets.

**Q: Inside, `BFL_Typography`, Is this correct for the HUD enum case?**

**A:** Mostly yes for the `HUD` enum branch: you are building an `FSlateFontInfo` from the `DA_Typography_Console` “HUD-*” fields and feeding it into `Set Font (Target is Text)`. That is the correct shape for applying a font to the UMG “Text” widget (`UTextBlock`).

Two correctness checks (the usual failure modes)

1. `Font Object` pin must be the actual font `.uasset` (Composite Font / Font Face / Font asset), not a “family name” string/metadata.

* Your DA output labeled `FSlate Font Info HUD Font Family` must be a Font asset reference type compatible with `Font Object`.

2. `Typeface Font Name` must match an actual face key inside the font asset (case-sensitive in practice).

* If the face key is missing/mismatched, Unreal will silently fall back to the default face.

Recommended improvement (eliminates the big Switch and the rebuild)

* Make `GetConsoleFontInfo(DA, Style)` return a single `FSlateFontInfo` (or store a `Map<Style, FSlateFontInfo>` in the DA).

* Then your apply function is:

* `FontInfo = GetConsoleFontInfo(DA, Style)`

* If `TargetWidget` casts to `Text` → `SetFont(FontInfo)`

* Else if it casts to `RichTextBlock` → `SetDefaultFont(FontInfo)` (or `SetDefaultTextStyle` if you store `FTextBlockStyle`)

This removes 4 duplicated `Make Slate Font Info` blocks and avoids wiring errors.

One caution

* `Force Monospaced` and `Material Is Stencil` should usually stay false for HUD text unless you explicitly need them; enabling either can produce spacing or masking surprises.

**Q: `GetConsoleFontInfo` requires a world context object reference. Why?**

**A:** Because a Blueprint Function Library (BFL) function is effectively “static.” If any node inside the function needs a `UWorld` (World) to operate, Unreal forces a `WorldContextObject` input so it can resolve the world at call time.

For `GetConsoleFontInfo` (a pure “lookup + build `FSlateFontInfo`” function), you should not need world context. If it is required, one of these is true:

1. You (or a template) added an input parameter flagged as World Context

* In the function’s Details panel, an input can be marked as “World Context.”
* Remove that parameter, or unmark it as World Context.

2. You used at least one world-dependent node inside `GetConsoleFontInfo`
   Common culprits (even if they seem unrelated):

* `Get Player Controller`, `Get Owning Player`, `Create Widget`
* `Get All Widgets Of Class`
* Any latent/timer node, subsystem getters that require world
* `Get Game Instance`, `Get World Delta Seconds` (via time-based effects)

Workaround (recommended)

* Make `GetConsoleFontInfo` a Pure function that only does:

  * `Map Find` (DA → Spec by enum)
  * `Make Slate Font Info`
  * `Return`
* Move any world-dependent logic (platform selection, time, shimmer phase, etc.) out of `GetConsoleFontInfo` into the caller (UserWidget / Actor) or into a separate function that intentionally takes World Context.

If you must keep the World Context input

* Pass `Self` from any object that lives in a world (UserWidget, Actor, PlayerController). That satisfies `WorldContextObject` without changing behavior.

If you paste a screenshot of the nodes inside `GetConsoleFontInfo` (full graph), I can point to the exact node that is forcing the world context requirement.

**Q: Yes, an internal cast is performed `TextBlock = cast_to_text_block(widget_obj)`!**

**A:** That cast is not what requires world context.

A `Cast To TextBlock` node is world-independent; it operates on an object pointer and RTTI (run-time type information). A `WorldContextObject` requirement is triggered by either:

1. The function is declared with a world context meta flag, or
2. Some other node in the function requires a `UWorld`.

Fix (keep your cast, remove world context)

1. Split responsibilities

* `GetConsoleFontInfo(DA, Style) -> FSlateFontInfo`
  Contents: Map Find + Make Slate Font Info + Return. No casts. No widgets. No world.
* `ApplyConsoleTypography(TargetWidget, DA, Style)`
  Contents: Cast TargetWidget to `Text` / `RichTextBlock` and apply the returned FontInfo.

2. Remove the world context flag if it exists

* Open `GetConsoleFontInfo` in the BFL.
* Click the function entry (not a node).
* In Details, check for anything like `World Context` / “Callable Without World Context” / metadata.
* Remove any `WorldContextObject` input or unset it as World Context.

If you cannot remove it (engine/editor quirk)

* Create the function somewhere else instead of a BFL:

  * In a `UserWidget` base class (or a `Blueprint Interface` implemented by widgets).
  * Widget member functions do not require explicit world context because `Self` resolves the world.

Minimal workaround (if you leave it as-is)

* Just pass `Self` (from the calling widget) into `WorldContextObject`. It is safe and correct.

Diagnostic
If `GetConsoleFontInfo` contains only `Cast` + `Make Slate Font Info`, then the world context flag was added (explicitly or via duplication from a template). In that case, the correct action is to remove the flagged parameter, not to change your cast.

**Q: The issue is with one of the input parameters: I created an empty function with matching inputs.**

**A:** If an empty Blueprint Function Library (BFL) function with the same signature still demands `WorldContextObject`, then the requirement is coming from the **function/parameter metadata**, not from nodes in the body.

This happens when one input parameter is marked as (or inferred as) a **World Context** parameter.

Causes

1. The parameter is explicitly flagged “World Context” in its Details.
2. The parameter is named `WorldContextObject` (or similar) and the editor auto-treats it as the world context parameter.
3. The function was duplicated from another function that had a world context flag; the flag persisted.

Workarounds

A) Remove the world-context flag on the parameter

1. Open the BFL.
2. Click the function (not a node).
3. In the right panel, click the suspect input parameter row.
4. In the parameter Details, look for any of:

   * “World Context”
   * “Is World Context”
   * “World Context Object”
5. Disable it, compile.

B) Rename the parameter
If the parameter name includes `WorldContextObject`, rename it to something neutral:

* `TargetWidget`, `TargetObj`, `WidgetObj`, etc.
  Compile again.

C) Stop using BFL for this function
If the editor still forces it:

* Move `GetConsoleFontInfo` into a `UserWidget` base class (or a widget utility object).
* Member functions get world via `Self` and do not require an explicit world context input.

D) Ignore it and pass `Self`
If you want to keep it as-is:

* Wire `Self` (from the calling widget/actor) into `WorldContextObject`. It is safe.

Diagnostic
Tell me the **exact input parameter list** (names + types). The name is usually the trigger; if any input is literally named `WorldContextObject`, that is almost certainly the reason.

**Q: Why not use a blueprint class that derives direct from Object or AActor?**

**A:** You can. Use a `UObject`-derived “TypographyService” (or a `GameInstanceSubsystem`) if you want an instance-based API and to avoid Blueprint Function Library (BFL) world-context quirks. Avoid `AActor` unless you specifically need world placement/ticking/replication.

Why `UObject` is the right choice

* Lowest overhead; no transform, no components, no replication, no level placement.
* Clean lifecycle if owned by `GameInstance`, `PlayerController`, or a subsystem.
* Can hold your active `DA_Typography_*` selection and expose `Apply(Widget, Key)` methods.
* World context becomes implicit via the outer (e.g., `GameInstance`) or irrelevant if your methods are pure “map lookup + set font”.

Why not `AActor` for this

* Forces world/level concerns (spawning, persistence across map loads, network replication settings).
* More overhead and complexity for a problem that is UI-only.
* Risk of ordering issues (Actor not spawned yet when widgets construct).

Recommended architecture

Option 1: `UGameInstanceSubsystem` (best)

* Create `BP_TypographySubsystem : GameInstanceSubsystem`.
* Vars:

  * `ActiveTypography : DA_Typography` (set based on platform)
* Functions:

  * `GetFontInfo(Key) -> SlateFontInfo`
  * `ApplyToWidget(TargetWidget, Key)`:

    * cast to `Text` (`UTextBlock`) → `SetFont`
    * cast to `RichTextBlock` → `SetDefaultFont` or `SetTextStyleSet`
* In widgets:

  * `Get Game Instance Subsystem (TypographySubsystem)` → `ApplyToWidget(SelfChildText, HUD_BODY)`

Option 2: Plain `UObject` service

* `BP_TypographyService : Object`
* Construct once (store in `GameInstance` or `PlayerController`), then call methods.
* Same API as above.

When a widget base class is still useful

* If you want automatic application in `PreConstruct` across many widgets, a `UserWidget` base class that calls the subsystem/service is convenient. This avoids having to wire calls everywhere.

Net

* `UObject`/Subsystem is the correct workaround to keep “semantic key only” (`HUD`, `ALERT`, etc.) and avoid BFL signature/world-context friction.
* `AActor` is unnecessary unless you need tick-driven behavior or spatial presence.

**Q: Okay, SUCCESS! How do I invoke `B_Typography` in the UMG graph?**

**A:** 1. Ensure `B_Typography` is accessible as an object reference

* If `B_Typography` is a `UObject`-derived Blueprint:

  * Create and store one instance somewhere persistent:

    * `GameInstance` (recommended), or
    * `PlayerController`, or
    * a `UserWidget` variable (if lifetime is only that widget)

2. Recommended: store it on `GameInstance`

* Create `BP_MyGameInstance` (if you do not already have one) and set it in Project Settings → Maps & Modes → Game Instance.
* In `BP_MyGameInstance`:

  * Variable: `TypographySvc` (type: `B_Typography` Object Reference)
  * On `Event Init`:

    * `TypographySvc = Construct Object from Class`

      * Class = `B_Typography`
      * Outer = `Self`
    * (Optional) set `TypographySvc.ActiveDA = DA_Typography_Console/PC/Mobile`

3. Invoke from a UMG Widget Blueprint
   In your `WBP_*` graph:

* `Event PreConstruct` (for designer preview) and/or `Event Construct` (runtime)

  * `Get Game Instance` → `Cast To BP_MyGameInstance`
  * `Get TypographySvc`
  * Call your service function, e.g.:

    * `ApplyTypography(TargetWidget, StyleKey)`

      * `TargetWidget` = your Text / RichTextBlock variable (must be `Is Variable`)
      * `StyleKey` = `HUD`, `HUD_BODY`, `NUMERIC`, `ALERT`

If you do not want to cast every time

* In the widget:

  * Variable `TypographySvc` (type `B_Typography` Object Reference)
  * On `PreConstruct`:

    * If not valid, fetch once from GameInstance and cache.
  * Then call `TypographySvc.ApplyTypography(...)`.

Key requirement in UMG

* The target text widgets must be variables:

  * Designer → select Text/RichTextBlock → check `Is Variable`
  * Then you can wire them into `ApplyTypography`.

If your `B_Typography` method expects `Widget` (Object Reference), pass the Text/RichTextBlock variable directly; it is a widget instance.

**Q: How do I add a color attribute and set it in the blueprint**

**A:** Add color to the same semantic style spec you already apply (HUD/HUD_BODY/NUMERIC/ALERT), then set the widget’s color after you set the font.

1. Add color to your typography spec
   In the struct you look up from `DA_Typography_*` (e.g., `FTypographySpec`), add:

* `Color` : `Linear Color` (`FLinearColor`)
* (Optional) `bOverrideColor` : `bool` (so styles can opt out)

Populate `DA_Typography_Console` entries: `HUD.Color`, `ALERT.Color`, etc.

2. Set color for UMG “Text” (UTextBlock)
   In `B_Typography.ApplyTypography` after the `Cast To Text` / `Cast To TextBlock` succeeds:

* `Spec = MapFind(DA.Specs, Key)`
* `TextWidget -> SetFont(FontInfo)`
* `TextWidget -> SetColorAndOpacity( InColorAndOpacity )`

Pin type note:

* `SetColorAndOpacity` expects a `Slate Color` (`FSlateColor`) in most UMG widgets.
* Convert your `Linear Color` with:

  * `Make Slate Color` (input: `Specified Color` = `Spec.Color`)
* Then wire that into `SetColorAndOpacity`.

3. Set color for RichTextBlock
   If you are using RichText without tags (single default style for the whole block):

* After `SetDefaultFont(FontInfo)`, set default color either by:

  * `RichTextBlock -> SetDefaultColorAndOpacity` (if available), using `Make Slate Color(Spec.Color)`, or
  * Prefer setting the default text style:

    * Build an `FTextBlockStyle` once (or store it in the DA), then call `SetDefaultTextStyle(TextStyle)`.

If you are using RichText tags (`<HUD>...</>`):

* Do not set color in Blueprint per-widget.
* Put color in the RichText style set DataTable rows (`HUD`, `ALERT`, etc.) so the tag controls color.

4. Recommended consolidation
   If you want “one apply call sets everything”:

* Change your DA map value from “font fields” to `FTextBlockStyle`:

  * `Map<Key, TextBlockStyle>`
* Then:

  * `TextBlock`: set `Font` and `ColorAndOpacity` from `Break TextBlockStyle`
  * `RichTextBlock`: `SetDefaultTextStyle(TextBlockStyle)`

Minimal patch (keeps your existing design)

* Add `Color` to `FTypographySpec`.
* In `ApplyTypography`:

  * After font application, call `SetColorAndOpacity(MakeSlateColor(Spec.Color))` for `Text`.
  * For `RichTextBlock`, either set default color (if node exists) or move to style-table/tag workflow.

If you paste the current `ApplyTypography` node list (Text + RichText branches), I can tell you exactly where to place the `Make Slate Color` and which RichText color node exists in your build (the node set can differ by project settings/modules).

**Q: Is color and opacity a parameter in the fontinfo object?**

**A:** No. `FSlateFontInfo` does not include color or opacity.

* `FSlateFontInfo` = font asset + size + typeface name + outline + (optional) font material.
* Color/opacity is stored separately:

  * For `Text` (`UTextBlock`): `Color and Opacity` (`FSlateColor`) property; set via `SetColorAndOpacity`.
  * For `RichTextBlock`: color is part of the default `FTextBlockStyle` or per-tag style rows (and there is also a default color property/node depending on build).

If you want a single struct that carries both font and color, use `FTextBlockStyle` (it contains `Font` and color-related fields).

**Q: Why then is the UMG graph is not accepting my in-editor color settings? I don't have color configured.**

**A:** UMG editor-set color is usually “ignored” for one of these reasons:

1. You are using `RichTextBlock` with a style set or markup

* If `Text Style Set` is assigned (DataTable) or your text contains tags like `<HUD>...</>`, the **row’s `FTextBlockStyle`** controls color. The widget’s Designer “Color and Opacity” is not authoritative for tagged spans.
  Fix: set color in the DataTable row (`HUD`, `ALERT`, etc.) under `TextStyle.ColorAndOpacity`, or stop using tags.

2. Your Blueprint is overriding color at runtime (directly or indirectly)
   Common nodes that override Designer settings:

* `SetColorAndOpacity` (Text)
* `SetDefaultTextStyle` / `SetTextStyleSet` (RichTextBlock; these styles carry color)
* Any binding on `Color and Opacity`
  Fix: if you call `SetDefaultTextStyle`, ensure the `FTextBlockStyle` you pass includes your desired color; otherwise you reset it to the style default (often white).

3. The text is set to “Use Foreground” / inherited color
   For `Text` (`UTextBlock`), `Color and Opacity` can be a `SlateColor` configured as “Use Foreground,” which then pulls color from the parent/widget style, not the specified value.
   Fix: set `Color and Opacity` to a **Specified Color** (not Foreground/inherited) on the widget, or set the parent foreground.

4. Your `FontMaterial` ignores vertex color
   If you assign `FontMaterial` (shimmer material), and the material does not multiply by `VertexColor` (or the UI text color input), the output color becomes material-defined and Designer color has no effect.
   Fix: in the font UI material, multiply your emissive/base color by `VertexColor` (and preserve glyph alpha).

Fast way to make it deterministic (recommended)

* Add `Color : LinearColor` to your `FTypographySpec` in `DA_Typography_*`.
* In `B_Typography.ApplyTypography`:

  * `Text` (`UTextBlock`): `SetColorAndOpacity(MakeSlateColor(Spec.Color))`
  * `RichTextBlock`: prefer `SetDefaultTextStyle` with a style that includes color (or set color per-tag in the style table).

If you confirm whether the failing widget is `Text` or `RichTextBlock`, and whether you set `FontMaterial`, the single most likely root cause can be pinpointed immediately (for RichText + tags, it is almost always #1 or #2).

## OpenAI ChatGPT-1-Oh-Preview (Strawberry version 1.2024.268)

**Q:** How do I create an unreal engine c++ plugin which contains a treeview and a tabview ? The plugin, DocumentationPlugin, populates the treeview from a local JSON file, TreeViewPanel.json, which contains chapter headings or subchapters as keys and textual pages as values which contain markdown -- which are to be displayed within the tabview!

**A:** To create an Unreal Engine C++ plugin named DocumentationPlugin that features a tree view populated from a JSON file and displays markdown content in a tab view, follow these comprehensive steps.

Prerequisites:

	•	Unreal Engine installed (version 4.26 or later recommended)
	•	Basic knowledge of C++ and Unreal Engine plugin development
	•	Familiarity with Unreal’s Slate UI framework for editor extensions

Step 1: Create a New Plugin

	1.	Open your Unreal Engine project.
	2.	Navigate to Edit > Plugins.
	3.	Click the New Plugin button.
	4.	Choose the Editor Standalone Window template (since we need a custom editor window).
	5.	Name the plugin DocumentationPlugin.
	6.	Click Create Plugin.

This action generates the basic plugin structure within your project’s Plugins directory.

Step 2: Set Up the Plugin Module

Locate the plugin source files, typically found at Plugins/DocumentationPlugin/Source/DocumentationPlugin/. Open DocumentationPlugin.h and DocumentationPlugin.cpp.

DocumentationPlugin.h

```cpp
#pragma once

#include "Modules/ModuleManager.h"

class FDocumentationPluginModule : public IModuleInterface
{
public:
    virtual void StartupModule() override;
    virtual void ShutdownModule() override;
};
```

DocumentationPlugin.cpp

```cpp
#include "DocumentationPlugin.h"
#include "LevelEditor.h"
#include "Widgets/Docking/SDockTab.h"
#include "ToolMenus.h"

static const FName DocumentationPluginTabName("DocumentationPlugin");

#define LOCTEXT_NAMESPACE "FDocumentationPluginModule"

void FDocumentationPluginModule::StartupModule()
{
    // Register the tab spawner
    FGlobalTabmanager::Get()->RegisterNomadTabSpawner(
        DocumentationPluginTabName,
        FOnSpawnTab::CreateRaw(this, &FDocumentationPluginModule::OnSpawnPluginTab)
    )
    .SetDisplayName(LOCTEXT("DocumentationTabTitle", "Documentation"))
    .SetMenuType(ETabSpawnerMenuType::Hidden);

    // Optionally register menu items here
}

void FDocumentationPluginModule::ShutdownModule()
{
    FGlobalTabmanager::Get()->UnregisterNomadTabSpawner(DocumentationPluginTabName);
}

TSharedRef<SDockTab> FDocumentationPluginModule::OnSpawnPluginTab(const FSpawnTabArgs& SpawnTabArgs)
{
    // Placeholder for the content; we'll fill this in later
    return SNew(SDockTab)
        .TabRole(ETabRole::NomadTab)
        [
            SNew(STextBlock).Text(LOCTEXT("WelcomeText", "Documentation Plugin"))
        ];
}

#undef LOCTEXT_NAMESPACE

IMPLEMENT_MODULE(FDocumentationPluginModule, DocumentationPlugin)
```

Step 3: Create the User Interface

We’ll create a custom Slate widget to house the tree view and tab view.

Create SDocumentationWidget.h

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Widgets/SCompoundWidget.h"
#include "Widgets/Views/STreeView.h"

struct FDocTreeItem : public TSharedFromThis<FDocTreeItem>
{
    FString Name;
    FString MarkdownContent;
    TArray<TSharedPtr<FDocTreeItem>> Children;

    FDocTreeItem(const FString& InName)
        : Name(InName)
    {}
};

class SDocumentationWidget : public SCompoundWidget
{
public:
    SLATE_BEGIN_ARGS(SDocumentationWidget) {}
    SLATE_END_ARGS()

    void Construct(const FArguments& InArgs);

private:
    void LoadDocumentationTree();
    TSharedRef<ITableRow> GenerateTreeRow(TSharedPtr<FDocTreeItem> InItem, const TSharedRef<STableViewBase>& OwnerTable);
    void GetChildrenForTree(TSharedPtr<FDocTreeItem> InParent, TArray<TSharedPtr<FDocTreeItem>>& OutChildren);
    void OnTreeSelectionChanged(TSharedPtr<FDocTreeItem> SelectedItem, ESelectInfo::Type SelectInfo);
    void DisplayMarkdownContent(const FString& MarkdownContent);

    TArray<TSharedPtr<FDocTreeItem>> TreeRootItems;
    TSharedPtr<STreeView<TSharedPtr<FDocTreeItem>>> TreeViewWidget;
    TSharedPtr<class STabManager> TabManager;
};
```
Create SDocumentationWidget.cpp
```cpp
#include "SDocumentationWidget.h"
#include "Misc/FileHelper.h"
#include "Misc/Paths.h"
#include "Serialization/JsonReader.h"
#include "Serialization/JsonSerializer.h"
#include "Widgets/Layout/SSplitter.h"
#include "Widgets/Text/STextBlock.h"
#include "Widgets/Docking/SDockTab.h"
#include "Widgets/Input/SMultiLineEditableText.h"
#include "Widgets/Layout/SScrollBox.h"

void SDocumentationWidget::Construct(const FArguments& InArgs)
{
    LoadDocumentationTree();

    ChildSlot
    [
        SNew(SSplitter)
        + SSplitter::Slot()
        .Value(0.3f)
        [
            SAssignNew(TreeViewWidget, STreeView<TSharedPtr<FDocTreeItem>>)
            .TreeItemsSource(&TreeRootItems)
            .OnGenerateRow(this, &SDocumentationWidget::GenerateTreeRow)
            .OnGetChildren(this, &SDocumentationWidget::GetChildrenForTree)
            .OnSelectionChanged(this, &SDocumentationWidget::OnTreeSelectionChanged)
        ]
        + SSplitter::Slot()
        .Value(0.7f)
        [
            SAssignNew(TabManager, STabManager)
            // Tab content will be added dynamically
        ]
    ];
}

void SDocumentationWidget::LoadDocumentationTree()
{
    FString JsonFilePath = FPaths::ProjectPluginsDir() / TEXT("DocumentationPlugin/Resources/TreeViewPanel.json");
    FString JsonContent;

    if (FFileHelper::LoadFileToString(JsonContent, *JsonFilePath))
    {
        TSharedPtr<FJsonObject> JsonObject;
        TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(JsonContent);

        if (FJsonSerializer::Deserialize(Reader, JsonObject) && JsonObject.IsValid())
        {
            for (auto& Elem : JsonObject->Values)
            {
                TSharedPtr<FDocTreeItem> RootItem = MakeShared<FDocTreeItem>(Elem.Key);

                if (Elem.Value->Type == EJson::Object)
                {
                    TSharedPtr<FJsonObject> ChildObject = Elem.Value->AsObject();
                    for (auto& ChildElem : ChildObject->Values)
                    {
                        TSharedPtr<FDocTreeItem> ChildItem = MakeShared<FDocTreeItem>(ChildElem.Key);
                        ChildItem->MarkdownContent = ChildElem.Value->AsString();
                        RootItem->Children.Add(ChildItem);
                    }
                }
                else if (Elem.Value->Type == EJson::String)
                {
                    RootItem->MarkdownContent = Elem.Value->AsString();
                }

                TreeRootItems.Add(RootItem);
            }
        }
    }
}

TSharedRef<ITableRow> SDocumentationWidget::GenerateTreeRow(TSharedPtr<FDocTreeItem> InItem, const TSharedRef<STableViewBase>& OwnerTable)
{
    return SNew(STableRow<TSharedPtr<FDocTreeItem>>, OwnerTable)
        [
            SNew(STextBlock).Text(FText::FromString(InItem->Name))
        ];
}

void SDocumentationWidget::GetChildrenForTree(TSharedPtr<FDocTreeItem> InParent, TArray<TSharedPtr<FDocTreeItem>>& OutChildren)
{
    OutChildren = InParent->Children;
}

void SDocumentationWidget::OnTreeSelectionChanged(TSharedPtr<FDocTreeItem> SelectedItem, ESelectInfo::Type SelectInfo)
{
    if (SelectedItem.IsValid() && !SelectedItem->MarkdownContent.IsEmpty())
    {
        DisplayMarkdownContent(SelectedItem->MarkdownContent);
    }
}

void SDocumentationWidget::DisplayMarkdownContent(const FString& MarkdownContent)
{
    // For demonstration, display the markdown as plain text
    TSharedRef<SDockTab> NewTab = SNew(SDockTab)
        .Label(FText::FromString("Documentation"))
        [
            SNew(SScrollBox)
            + SScrollBox::Slot()
            [
                SNew(SMultiLineEditableText)
                .Text(FText::FromString(MarkdownContent))
                .IsReadOnly(true)
            ]
        ];

    // Add the new tab to the tab manager
    if (TabManager.IsValid())
    {
        TabManager->InsertNewDocumentTab(
            FTabId("DocumentationTab"),
            FTabManager::ESearchPreference::PreferLiveTab,
            NewTab
        );
    }
}
```
Step 4: Modify the Plugin Module to Use the Custom Widget

Update the OnSpawnPluginTab function in DocumentationPlugin.cpp to use SDocumentationWidget.

DocumentationPlugin.cpp
```cpp
#include "DocumentationPlugin.h"
#include "SDocumentationWidget.h"

TSharedRef<SDockTab> FDocumentationPluginModule::OnSpawnPluginTab(const FSpawnTabArgs& SpawnTabArgs)
{
    return SNew(SDockTab)
        .TabRole(ETabRole::NomadTab)
        [
            SNew(SDocumentationWidget)
        ];
}
```
Step 5: Prepare the JSON File

Create the TreeViewPanel.json file in your plugin’s Resources directory (e.g., Plugins/DocumentationPlugin/Resources/TreeViewPanel.json).
```cpp
Example TreeViewPanel.json

{
    "Getting Started": {
        "Introduction": "# Introduction\nThis is the introduction section.",
        "Setup": "# Setup\nInstructions on setting up the environment."
    },
    "Advanced Topics": {
        "Optimization": "# Optimization\nDetails about optimization techniques.",
        "Best Practices": "# Best Practices\nGuidelines and best practices."
    }
}
```
Step 6: Handle Markdown Content

Unreal Engine does not have built-in markdown support, so you’ll need to parse the markdown content.

Option 1: Display as Plain Text

In the DisplayMarkdownContent function, we’re currently displaying markdown as plain text.

Option 2: Integrate a Markdown Parser

To render markdown properly, integrate a markdown parsing library like cmark. This requires:

	•	Adding the markdown parser as a third-party library.
	•	Converting the parsed markdown into Slate widgets or rich text.

Option 3: Basic Markdown Parsing

Implement simple parsing for common markdown elements (headers, bold text, etc.) using SRichTextBlock.

Example with SRichTextBlock

Update DisplayMarkdownContent:
```cpp
void SDocumentationWidget::DisplayMarkdownContent(const FString& MarkdownContent)
{
    FString ParsedContent = ParseMarkdownToRichText(MarkdownContent);

    TSharedRef<SDockTab> NewTab = SNew(SDockTab)
        .Label(FText::FromString("Documentation"))
        [
            SNew(SScrollBox)
            + SScrollBox::Slot()
            [
                SNew(SRichTextBlock)
                .Text(FText::FromString(ParsedContent))
                .DecoratorStyleSet(&FCoreStyle::Get())
                .AutoWrapText(true)
            ]
        ];

    if (TabManager.IsValid())
    {
        TabManager->InsertNewDocumentTab(
            FTabId("DocumentationTab"),
            FTabManager::ESearchPreference::PreferLiveTab,
            NewTab
        );
    }
}
```
Implement ParseMarkdownToRichText to handle basic markdown.

Step 7: Build and Test the Plugin

	1.	Build the Plugin:
	    •	Open your project solution in Visual Studio or your preferred IDE.
	    •	Build the solution to compile the plugin.
	2.	Enable the Plugin:
	    •	Restart Unreal Editor.
	    •	Go to Edit > Plugins.
	    •	Enable DocumentationPlugin if it’s not already enabled.
	3.	Access the Plugin:
	    •	Navigate to Window > Documentation (or wherever you registered the menu item).
	    •	The Documentation tab should appear.
	4.	Test Functionality:
	    •	Click on items in the tree view.
	    •	Verify that the corresponding markdown content is displayed in the tab view.

Additional Tips

	•	Error Handling: Add checks to handle file loading failures and JSON parsing errors.
	•	UI Enhancements: Customize the look and feel using Slate styling and custom widgets.
	•	Dynamic Updates: Implement file watchers to reload the JSON file when changes are detected.
	•	Markdown Support: For full markdown support, integrating a third-party library is recommended.

Conclusion

By following these steps, you have created a functional Unreal Engine C++ plugin that reads from a JSON file, displays a navigable tree view, and shows markdown-formatted content within tabs. This plugin can serve as a foundation for creating in-editor documentation or help systems.

Note: Remember to consult the Unreal Engine Documentation for detailed information on plugin development, Slate UI, and handling JSON data.

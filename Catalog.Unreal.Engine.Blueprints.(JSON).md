## OpenAI ChatGPT-1-Oh-mini (Strawberry-mini version 1.2024.261)

**Q:** If i have the unreal engine python plugin enabled, i'd like to write a script to catalog all unreal engine blueprints by category to include all methods, attributes, etc., et al., as a JSON library.

**A:** Creating a comprehensive JSON catalog of all Blueprints in your Unreal Engine project is a great way to manage and document your assets. Below is a step-by-step guide to help you achieve this using the Unreal Engine Python API. The script will:

1. **Access all Blueprints in the project.**
2. **Categorize them based on their specified categories.**
3. **Extract methods (functions), attributes (variables), and other relevant details.**
4. **Compile the data into a structured JSON format.**

### **Prerequisites**

1. **Unreal Engine Python Plugin Enabled:** Ensure that the Python Editor Script Plugin is enabled in your Unreal Engine project. You can enable it via **Edit > Plugins > Scripting > Python Editor Script Plugin**.

2. **Python Environment:** Ensure that your Python environment is set up correctly and that you have access to necessary modules like `json`.

### **Step-by-Step Guide**

#### **1. Import Necessary Modules**

First, import the required Unreal Engine Python modules and standard Python libraries.

```python
import unreal
import json
import os
```

#### **2. Define Helper Functions**

To keep the code organized, define helper functions to extract information from Blueprints.

```python
def get_all_blueprints():
    # Use the Asset Registry to find all Blueprint assets
    asset_registry = unreal.AssetRegistryHelpers.get_asset_registry()
    asset_data = asset_registry.get_assets_by_class("Blueprint")
    return asset_data

def get_blueprint_details(blueprint_asset):
    blueprint = unreal.EditorAssetLibrary.load_asset(blueprint_asset.object_path)
    if not blueprint:
        return None
    
    # Basic Blueprint Information
    blueprint_info = {
        'name': blueprint.get_name(),
        'path': blueprint.get_path_name(),
        'parent_class': blueprint.get_parent_class().get_name() if blueprint.get_parent_class() else 'None',
        'category': blueprint.get_category() if blueprint.get_category() else 'Uncategorized',
        'variables': [],
        'functions': []
    }
    
    # Extract Variables
    variables = blueprint.get_editor_property('new_variables')
    for var in variables:
        var_info = {
            'name': var.get_name(),
            'type': var.get_editor_property('variable_type').get_display_name(),
            'default_value': var.get_editor_property('default_value')
        }
        blueprint_info['variables'].append(var_info)
    
    # Extract Functions
    functions = blueprint.get_editor_property('uber_graph_pages')
    for func in functions:
        func_info = {
            'name': func.get_name(),
            'flags': func.get_function_flags(),
            # Add more function details as needed
        }
        blueprint_info['functions'].append(func_info)
    
    return blueprint_info
```

#### **3. Catalog Blueprints**

Iterate through all Blueprints, categorize them, and extract their details.

```python
def catalog_blueprints():
    blueprints = get_all_blueprints()
    catalog = {}

    for bp_asset in blueprints:
        bp_details = get_blueprint_details(bp_asset)
        if not bp_details:
            continue
        
        category = bp_details['category']
        if category not in catalog:
            catalog[category] = []
        
        catalog[category].append(bp_details)
    
    return catalog
```

#### **4. Export to JSON**

Define a function to export the catalog to a JSON file.

```python
def export_to_json(data, output_path):
    # Ensure the directory exists
    os.makedirs(os.path.dirname(output_path), exist_ok=True)
    
    with open(output_path, 'w') as json_file:
        json.dump(data, json_file, indent=4)
    
    unreal.log(f"Blueprint catalog exported to {output_path}")
```

#### **5. Main Execution Function**

Combine all steps into a main function.

```python
def main():
    catalog = catalog_blueprints()
    output_path = os.path.join(unreal.Paths.project_content_dir(), "BlueprintCatalog.json")
    export_to_json(catalog, output_path)

# Execute the script
if __name__ == "__main__":
    main()
```

#### **6. Running the Script**

1. **Save the Script:** Save the above script as `catalog_blueprints.py` within your project's `Scripts` directory or any accessible location.

2. **Run the Script:**
   - Open the **Output Log** in Unreal Engine (**Window > Developer Tools > Output Log**).
   - Execute the script using the Python command:
     ```
     py "path/to/catalog_blueprints.py"
     ```
   - Replace `"path/to/catalog_blueprints.py"` with the actual path to your script.

3. **Check the Output:**
   - After execution, check your project's `Content` directory for the `BlueprintCatalog.json` file.
   - Open the JSON file to view the categorized Blueprint catalog.

### **Enhancements and Considerations**

- **Error Handling:** Add error handling to manage assets that may not load correctly or lack certain properties.

- **Detailed Function Information:** The current script extracts basic function information. To get more details like parameters, return types, etc., you can expand the `get_blueprint_details` function accordingly.

- **Performance Optimization:** For projects with a large number of Blueprints, consider optimizing the script to handle large datasets efficiently.

- **Additional Properties:** You might want to include more Blueprint properties such as interfaces implemented, event graphs, components used, etc.

- **GUI Integration:** Integrate the script with Unreal's Editor UI for easier access and execution.

### **Sample JSON Output**

Here's a simplified example of what the `BlueprintCatalog.json` might look like:

```json
{
    "Gameplay": [
        {
            "name": "BP_PlayerCharacter",
            "path": "/Game/Blueprints/BP_PlayerCharacter.BP_PlayerCharacter",
            "parent_class": "Character",
            "category": "Gameplay",
            "variables": [
                {
                    "name": "Health",
                    "type": "Float",
                    "default_value": 100.0
                },
                {
                    "name": "Speed",
                    "type": "Float",
                    "default_value": 600.0
                }
            ],
            "functions": [
                {
                    "name": "MoveForward",
                    "flags": 0
                },
                {
                    "name": "TakeDamage",
                    "flags": 0
                }
            ]
        }
    ],
    "UI": [
        {
            "name": "BP_MainMenu",
            "path": "/Game/UI/BP_MainMenu.BP_MainMenu",
            "parent_class": "UserWidget",
            "category": "UI",
            "variables": [],
            "functions": [
                {
                    "name": "InitializeMenu",
                    "flags": 0
                }
            ]
        }
    ]
}
```

### **Final Notes**

- **Unreal Engine Python API Documentation:** Refer to the [Unreal Engine Python API documentation](https://docs.unrealengine.com/en-US/PythonAPI/index.html) for more detailed information on available classes and methods.

- **Testing:** Before running the script on a large project, test it on a smaller project to ensure it works as expected.

- **Extensibility:** This script serves as a foundational framework. Depending on your project's needs, you can extend it to include more detailed information or integrate it with other tools and workflows.

Feel free to customize and enhance the script to better fit your project's specific requirements!

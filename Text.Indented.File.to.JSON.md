```python
import json

def count_leading_tabs(line):
    """Count the number of leading tabs in a line."""
    return len(line) - len(line.lstrip('\t'))

def convert_to_json(input_file, output_file):
    with open(input_file, 'r') as f:
        lines = f.read().splitlines()

    json_lines = []
    stack = []  # To keep track of indentation levels
    json_lines.append('{')  # Start of JSON

    for i, line in enumerate(lines):
        current_indent = count_leading_tabs(line)
        key = line.strip()
        
        # Determine the indentation of the next line
        if i + 1 < len(lines):
            next_indent = count_leading_tabs(lines[i + 1])
        else:
            next_indent = 0  # No next line

        # Adjust the stack and close braces if current_indent < stack's last
        while stack and current_indent < stack[-1]:
            stack.pop()
            json_lines.append('  ' * stack.__len__() + '},')

        # Prepare the key with proper indentation
        indent = '  ' * (current_indent + 1)
        if next_indent > current_indent:
            # Current key has children
            json_lines.append(f'{indent}"{key}": {{')
            stack.append(current_indent)
        elif next_indent < current_indent:
            # Current key is a leaf and we need to close braces
            json_lines.append(f'{indent}"{key}": "",')
            # Close the braces outside the loop
        else:
            # Current key is a leaf
            json_lines.append(f'{indent}"{key}": "",')

    # Close any remaining open braces
    while stack:
        stack.pop()
        json_lines.append('  ' * (stack.__len__() +1) + '},')

    # Remove the last comma if exists and close the root
    if json_lines[-1].endswith(','):
        json_lines[-1] = json_lines[-1][:-1]
    json_lines.append('}')

    # Join all lines into a single JSON string
    json_str = '\n'.join(json_lines)

    # Optional: Validate JSON by loading it
    try:
        parsed_json = json.loads(json_str)
    except json.JSONDecodeError as e:
        print("Error in generated JSON:", e)
        print("Generated JSON:")
        print(json_str)
        return

    # Write the JSON to the output file with indentation
    with open(output_file, 'w') as f:
        json.dump(parsed_json, f, indent=2)

    print(f"JSON has been successfully written to {output_file}")

if __name__ == "__main__":
    input_file = 'input.txt'   # Replace with your input file path
    output_file = 'output.json'  # Replace with your desired output file path
    convert_to_json(input_file, output_file)
```

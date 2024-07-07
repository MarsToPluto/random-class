# Obfuscation of Class Names, IDs, and Selectors

## Purpose
The purpose of this document is to outline the process of obfuscating class names, IDs, and selectors in HTML, CSS, and JS files. This is done to prevent conflicts with existing or third-party code and to make the code less readable to others.

## Obfuscation Methodology

### 1. Generating Random Names
   - **Function:** A random string generator is used to create unique replacements for class names and IDs.
   - **Implementation:** Python's `random` and `string` modules are employed to generate alphanumeric strings of a specified length.

### 2. Finding Class Names and IDs
   - **Function:** Regular expressions (`re` module) are utilized to identify existing class names and IDs in the codebase.
   - **Implementation:** Separate functions handle HTML/JS and CSS files due to different syntax and contexts.

### 3. Replacing Class Names and IDs
   - **Function:** Another set of regular expressions replace identified class names, IDs, and selectors with the generated random names.
   - **Implementation:** Different replacement patterns are applied based on whether the content is HTML/JS or CSS.

### 4. Processing Files
   - **Function:** The main function orchestrates the entire process by scanning all specified files, identifying all class names and IDs, generating replacements, and finally replacing them.
   - **Implementation:** Python's file I/O functions (`open`, `read`, `write`) handle reading from and writing to files.

## Example Code

```python
import os
import re
import random
import string
import logging

# Setup logging
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(levelname)s - %(message)s')

# Function to generate a random string
def generate_random_string(length=8):
    return ''.join(random.choices(string.ascii_letters + string.digits, k=length))

# Function to replace class names and IDs in HTML and JS content
def replace_classnames_ids(content, replacements):
    for old_name, new_name in replacements.items():
        # Replace class names
        content = re.sub(rf'class=["\']{re.escape(old_name)}["\']', f'class="{new_name}"', content)
        content = re.sub(rf'className=["\']{re.escape(old_name)}["\']', f'className="{new_name}"', content)
        content = re.sub(rf'\bquerySelector\(["\'].{re.escape(old_name)}["\']\)', f'querySelector(".{new_name}")', content)
        content = re.sub(rf'\bquerySelectorAll\(["\'].{re.escape(old_name)}["\']\)', f'querySelectorAll(".{new_name}")', content)
        content = re.sub(rf'\bgetElementsByClassName\(["\']{re.escape(old_name)}["\']\)', f'getElementsByClassName("{new_name}")', content)
        
        # Replace IDs
        content = re.sub(rf'id=["\']{re.escape(old_name)}["\']', f'id="{new_name}"', content)
        content = re.sub(rf'\bgetElementById\(["\']{re.escape(old_name)}["\']\)', f'getElementById("{new_name}")', content)
        
    return content

# Function to replace class names and IDs in CSS content
def replace_classnames_ids_css(content, replacements):
    for old_name, new_name in replacements.items():
        content = re.sub(rf'\.({re.escape(old_name)})(\s|[{{:,])', rf'.{new_name}\2', content)
        content = re.sub(rf'#({re.escape(old_name)})(\s|[{{:,])', rf'#{new_name}\2', content)
    return content

# Function to find all class names and IDs in HTML and JS content
def find_classnames_ids(content):
    classnames = set(re.findall(r'class=["\']([^"\']+)["\']', content))
    ids = set(re.findall(r'id=["\']([^"\']+)["\']', content))
    
    classnames_js = set(re.findall(r'querySelector\(["\'].([^"\']+)["\']\)', content))
    classnames_js.update(re.findall(r'querySelectorAll\(["\'].([^"\']+)["\']\)', content))
    classnames_js.update(re.findall(r'getElementsByClassName\(["\']([^"\']+)["\']\)', content))
    
    ids_js = set(re.findall(r'getElementById\(["\']([^"\']+)["\']\)', content))
    
    classnames.update(classnames_js)
    ids.update(ids_js)
    
    return classnames, ids

# Function to find class names and IDs in CSS content
def find_classnames_ids_css(content):
    classnames = set(re.findall(r'\.([a-zA-Z0-9_-]+)(\s|[{{:,])', content))
    ids = set(re.findall(r'#([a-zA-Z0-9_-]+)(\s|[{{:,])', content))
    return {classname for classname, _ in classnames}, {id_name for id_name, _ in ids}

# Function to process files
def process_files(files):
    replacements = {}
    all_classnames = set()
    all_ids = set()

    # First pass: find all class names and IDs in all files
    for file_path in files:
        logging.debug(f"Scanning file: {file_path}")
        try:
            with open(file_path, 'r') as file:
                content = file.read()
                if file_path.endswith('.css'):
                    classnames, ids = find_classnames_ids_css(content)
                else:
                    classnames, ids = find_classnames_ids(content)
                all_classnames.update(classnames)
                all_ids.update(ids)
        except FileNotFoundError as fnf_error:
            logging.error(f"File not found: {fnf_error.filename}")
        except PermissionError as p_error:
            logging.error(f"Permission error: {p_error.filename}")

    # Create replacements dictionary
    for classname in all_classnames:
        if classname not in replacements:
            replacements[classname] = generate_random_string()
    for id_name in all_ids:
        if id_name not in replacements:
            replacements[id_name] = generate_random_string()

    logging.debug(f"Replacements dictionary: {replacements}")

    # Second pass: replace class names and IDs in all files
    for file_path in files:
        logging.debug(f"Processing file: {file_path}")
        try:
            with open(file_path, 'r') as file:
                content = file.read()
            if file_path.endswith('.css'):
                content = replace_classnames_ids_css(content, replacements)
            else:
                content = replace_classnames_ids(content, replacements)
            with open(file_path, 'w') as file:
                file.write(content)
            logging.debug(f"Processed file: {file_path}")
        except FileNotFoundError as fnf_error:
            logging.error(f"File not found: {fnf_error.filename}")
        except PermissionError as p_error:
            logging.error(f"Permission error: {p_error.filename}")

    logging.info("Class names and IDs have been replaced with random names in HTML, JS, and CSS files.")

# List of files to process
files = ['index.html', 'script.js', 'style.css']  # Add more files if needed

# Process the files
process_files(files)

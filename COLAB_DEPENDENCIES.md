# Colab-Specific Dependencies in RFdiffusion Notebook

This document identifies all Google Colab-specific features in the notebook and provides alternatives for running locally.

## 1. File Upload/Download (`google.colab.files`)

### Location 1: Cell 2 - Setup Cell (Line ~123)
```python
from google.colab import files
```

**Used in:** `get_pdb()` function (Line ~139)
```python
def get_pdb(pdb_code=None):
  if pdb_code is None or pdb_code == "":
    upload_dict = files.upload()  # ← Colab-specific
    pdb_string = upload_dict[list(upload_dict.keys())[0]]
    with open("tmp.pdb","wb") as out: out.write(pdb_string)
    return "tmp.pdb"
```

**Fix:** Replace with standard file input:
```python
def get_pdb(pdb_code=None):
  if pdb_code is None or pdb_code == "":
    # Use standard file input instead
    from tkinter import filedialog
    import tkinter as tk
    root = tk.Tk()
    root.withdraw()
    file_path = filedialog.askopenfilename(filetypes=[("PDB files", "*.pdb")])
    if file_path:
      return file_path
    else:
      raise ValueError("No file selected")
  # ... rest of function
```

**Or use IPython widgets:**
```python
from IPython.display import FileLink
import ipywidgets as widgets

def get_pdb(pdb_code=None):
  if pdb_code is None or pdb_code == "":
    upload = widgets.FileUpload(accept='.pdb', multiple=False)
    display(upload)
    # Wait for upload and process
    # This requires additional handling
```

### Location 2: Cell 6 - Libraries (Line ~602)
```python
from google.colab import drive
```
**Status:** This import appears but is **NOT USED** in the code. Can be safely removed or commented out.

### Location 3: Cell 8 - Master Runner (Line ~841)
```python
files.download("Master.zip")
```

**Fix:** Replace with:
```python
# Option 1: Use IPython FileLink
from IPython.display import FileLink
display(FileLink('Master.zip'))

# Option 2: Print path for manual download
import os
abs_path = os.path.abspath('Master.zip')
print(f"Results saved to: {abs_path}")

# Option 3: Use Jupyter's file browser to download
```

## 2. Hardcoded Colab Paths (`/content/`)

### Locations:
- Line ~638: `df.to_csv('/content/Master/Best_Results.csv')`
- Line ~784: `get_pdb_path(f'/content/outputs/{path}/best')`
- Line ~788: `get_pdb_path(f'/content/{protein_motifs_pdb}')`
- Line ~792: `get_pdb_path(f'/content/{full_protein_pdb}')`
- Line ~813: `df.to_csv('/content/Master/Best_Results.csv')`
- Line ~814: `os.system(f'mkdir /content/Master/{path}')`
- Line ~815: `copy_tree(f"/content/outputs/{path}", f"/content/Master/{path}")`
- Line ~839: `#!zip -r Master_Results.zip /content/Master` (commented out)

**Fix:** Replace all `/content/` with relative paths or use `os.getcwd()`:
```python
# Instead of '/content/Master/Best_Results.csv'
master_dir = os.path.join(os.getcwd(), 'Master')
os.makedirs(master_dir, exist_ok=True)
df.to_csv(os.path.join(master_dir, 'Best_Results.csv'))

# Instead of f'/content/outputs/{path}/best'
output_path = os.path.join(os.getcwd(), 'outputs', path, 'best')

# Instead of f'/content/{protein_motifs_pdb}'
motif_path = os.path.join(os.getcwd(), f'{protein_motifs_pdb}.pdb')
```

## 3. Colab Form Widgets (`#@title`, `#@param`, `#@markdown`)

These are Colab-specific form widgets that create interactive UI elements.

### Locations:
- `#@title setup **RFdiffusion** (~3min)` - Line ~76
- `#@title Display 3D structure {run: "auto"}` - Line ~406
- `#@param` annotations throughout (Lines ~412, 413, 415, 490, 491, 492, etc.)
- `#@markdown` annotations for documentation

**Status:** These are **comments** and don't break execution, but they won't create interactive forms.

**Fix:** 
- Remove or keep as comments (they're harmless)
- For interactive parameters, use `ipywidgets`:
```python
import ipywidgets as widgets
from IPython.display import display

num_runs = widgets.IntSlider(value=2, min=1, max=10, description='Runs:')
display(num_runs)
# Use num_runs.value to get the value
```

## 4. System Commands That May Need Adjustment

### Location: Cell 2 - Setup
```python
os.system("apt-get install aria2")  # Requires sudo in non-Colab
```

**Fix:** 
- Install `aria2` system-wide before running, or
- Use `wget` or `curl` as alternatives, or
- Check if `aria2` is available before using it

### Location: `/dev/shm/` Usage
The notebook uses `/dev/shm/` (shared memory) for temporary PDB files:
```python
inference.dump_pdb_path='/dev/shm'
```

**Status:** This works on Linux systems (including most local setups), but may need adjustment on other systems.

**Fix (if needed):**
```python
# Use a local temp directory instead
import tempfile
temp_dir = tempfile.mkdtemp()
# Use temp_dir instead of '/dev/shm'
```

## Summary of Required Changes

### Critical (Must Fix):
1. **Remove/Replace `google.colab.files` imports** (2 locations)
2. **Replace `files.upload()`** in `get_pdb()` function
3. **Replace `files.download()`** at end of Cell 8
4. **Replace all `/content/` paths** with relative paths (8+ locations)

### Optional (Nice to Have):
1. Replace `#@param` widgets with `ipywidgets` for interactivity
2. Handle `aria2` installation gracefully
3. Add error handling for file operations

### Safe to Ignore:
- `#@title` and `#@markdown` comments (they're just documentation)
- `from google.colab import drive` (imported but never used)

## Quick Fix Script

You can create a simple replacement function at the top of your notebook:

```python
# Compatibility layer for Colab features
try:
    from google.colab import files
    IN_COLAB = True
except ImportError:
    IN_COLAB = False
    # Create mock files object
    class MockFiles:
        def upload(self):
            from tkinter import filedialog
            import tkinter as tk
            root = tk.Tk()
            root.withdraw()
            file_path = filedialog.askopenfilename()
            if file_path:
                with open(file_path, 'rb') as f:
                    return {os.path.basename(file_path): f.read()}
            return {}
        
        def download(self, filename):
            from IPython.display import FileLink
            display(FileLink(filename))
            print(f"File ready: {os.path.abspath(filename)}")
    
    files = MockFiles()

# Replace /content/ paths
CONTENT_DIR = os.getcwd()  # Use current directory instead of /content
```



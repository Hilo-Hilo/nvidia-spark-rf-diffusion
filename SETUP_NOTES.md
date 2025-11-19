# RFdiffusion Environment Setup Notes

## Virtual Environment Created
- Location: `/home/hansonwen/igem/venv`
- Kernel name: `igem-rfdiffusion`
- Display name: "Python (igem-rfdiffusion)"

## Installed Packages
Core dependencies have been installed including:
- PyTorch (CPU version for ARM64)
- ColabDesign v1.1.1
- NumPy, Pandas, Matplotlib
- Jupyter and ipykernel
- Biopython, py3Dmol, ipywidgets
- Hydra, OmegaConf, and other RFdiffusion dependencies

## Additional Setup Required

### 1. RFdiffusion Repository
The notebook will clone RFdiffusion automatically, but you may need to install additional dependencies:
```bash
cd /home/hansonwen/igem
source venv/bin/activate
git clone https://github.com/sokrypton/RFdiffusion.git
cd RFdiffusion/env/SE3Transformer
pip install .
```

### 2. DGL and e3nn
These packages may need to be installed after RFdiffusion is cloned:
```bash
source venv/bin/activate
# Note: DGL 2.0.0 may not be available for ARM64, you may need to use a different version
pip install --no-dependencies dgl -f https://data.dgl.ai/wheels/cu121/repo.html
pip install --no-dependencies e3nn==0.5.5 opt_einsum_fx
```

### 3. tmtools
tmtools requires Python development headers. If you have sudo access:
```bash
sudo apt-get install python3-dev build-essential
source venv/bin/activate
pip install tmtools
```

### 4. Model Checkpoints and Parameters
The notebook will download these automatically:
- RFdiffusion model checkpoints (Base_ckpt.pt, Complex_base_ckpt.pt)
- AlphaFold parameters
- Schedules for diffusion

### 5. AnAnaS Tool
The notebook will download the AnAnaS symmetry detection tool automatically.

## Using the Kernel

1. Open Jupyter Notebook/Lab
2. Select the kernel: **"Python (igem-rfdiffusion)"**
3. **IMPORTANT:** Before running the notebook, see `COLAB_DEPENDENCIES.md` for required modifications:
   - Replace `google.colab.files` imports and usage
   - Replace all `/content/` hardcoded paths
   - Fix file upload/download functionality
4. The notebook should work after these modifications, but note:
   - GPU support depends on your system configuration

## Notes
- This setup is for ARM64 architecture (aarch64)
- PyTorch is installed in CPU mode (CUDA wheels not available for ARM64)
- Some packages may need manual installation depending on your system


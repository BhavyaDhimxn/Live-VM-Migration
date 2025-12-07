# DVC — Installation

## 🚀 Installation Methods

DVC can be installed using multiple methods depending on your environment and preferences.

## 📦 Method 1: pip (Recommended)

### Basic Installation
```bash
# Install DVC
pip install dvc

# For specific cloud storage support
pip install dvc[s3]      # AWS S3
pip install dvc[gcs]     # Google Cloud Storage
pip install dvc[azure]   # Azure Blob Storage
pip install dvc[all]     # All cloud providers
```

### Virtual Environment (Recommended)
```bash
# Create virtual environment
python -m venv dvc-env
source dvc-env/bin/activate  # On Windows: dvc-env\Scripts\activate

# Install DVC
pip install dvc[s3]
```

## 🐳 Method 2: Docker

```bash
# Pull DVC Docker image
docker pull iterative/dvc

# Run DVC commands
docker run --rm -v $(pwd):/workspace -w /workspace iterative/dvc dvc --version
```

## 🍺 Method 3: Homebrew (macOS)

```bash
# Install via Homebrew
brew install dvc

# Or install with cloud support
brew install dvc --with-s3
```

## 📋 Method 4: Conda

```bash
# Install via conda-forge
conda install -c conda-forge dvc

# Or create new environment
conda create -n dvc-env -c conda-forge dvc
conda activate dvc-env
```

## ⚙️ System Requirements

### Minimum Requirements
- **Python**: 3.8+
- **Git**: 2.8+
- **Storage**: 1GB free space (for cache)

### Recommended Requirements
- **Python**: 3.9+
- **Git**: 2.30+
- **Storage**: 10GB+ free space
- **RAM**: 4GB+

## 🔧 Post-Installation Setup

### 1. Initialize DVC in Your Project
```bash
# Navigate to your project directory
cd your-ml-project

# Initialize DVC (creates .dvc/ directory)
dvc init

# Add DVC files to Git
git add .dvc .gitignore
git commit -m "Initialize DVC"
```

### 2. Configure Remote Storage (Optional)
```bash
# Add remote storage (example: AWS S3)
dvc remote add -d myremote s3://my-bucket/dvc-storage

# Configure credentials (if needed)
dvc remote modify myremote access_key_id YOUR_ACCESS_KEY
dvc remote modify myremote secret_access_key YOUR_SECRET_KEY
```

## ✅ Quick Verification

### Test DVC Installation
```bash
# Check DVC version
dvc --version

# Expected output: dvc, version 3.x.x
```

### Test Basic Functionality
```bash
# Create test data
echo "Hello DVC!" > test.txt

# Add to DVC tracking
dvc add test.txt

# Check status
dvc status

# Remove test file
rm test.txt
dvc remove test.txt.dvc
```

### Test Git Integration
```bash
# Check if .dvc directory exists
ls -la .dvc/

# Expected files:
# - config
# - .gitignore
# - cache/
```

## 🐛 Common Installation Issues

### Issue 1: Permission Denied
```bash
# Error: Permission denied when installing
# Solution: Use --user flag
pip install --user dvc
```

### Issue 2: Git Not Found
```bash
# Error: Git is required but not found
# Solution: Install Git first
# Ubuntu/Debian:
sudo apt-get install git

# macOS:
brew install git

# Windows: Download from git-scm.com
```

### Issue 3: Python Version Incompatibility
```bash
# Error: Python version not supported
# Solution: Use Python 3.8+
python --version  # Check version
python3.9 -m pip install dvc  # Use specific version
```

### Issue 4: Cloud Storage Dependencies
```bash
# Error: Missing cloud storage libraries
# Solution: Install specific extras
pip install dvc[s3]  # For AWS S3
pip install dvc[gcs]  # For Google Cloud
```

## 🔍 Verification Checklist

- [ ] DVC version command works
- [ ] `dvc init` creates `.dvc/` directory
- [ ] Git integration works (`.dvc/.gitignore` exists)
- [ ] Can add files with `dvc add`
- [ ] Remote storage configured (if needed)
- [ ] Cloud credentials working (if using cloud storage)

## 🚀 Next Steps

After successful installation:

1. **📖 Read**: [Configuration Guide](README_Configuration.md)
2. **🛠️ Setup**: Configure your first remote storage
3. **📊 Practice**: Add your first dataset with `dvc add`
4. **🔄 Learn**: Create your first ML pipeline

---

*DVC is now ready to version your data and models! 🎉*
# tqqq-data-utils

## Python Setup

This repo uses a local virtual environment in `.venv`.

### Windows PowerShell

```powershell
Set-Location 'c:\Users\rohmi\tqqq-data-utils'
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### Windows Command Prompt

```bat
cd /d c:\Users\rohmi\tqqq-data-utils
python -m venv .venv
.venv\Scripts\activate.bat
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### Run scripts

After activation, run the repository scripts with the venv interpreter, for example:

```powershell
python synthetic_tqqq_data\scripts\download_tqqq_backfill.py
python tqqq_fee_analysis\calculate_tqqq_fee_yearly.py
```
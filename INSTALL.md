# Installation Guide for `pcat`

## Prerequisites
`pcat` requires **Python 3.6+**. 

For color support (optional but recommended), install `colorama`:
```bash
pip install colorama
```

## System-wide Installation (Linux / macOS)

To install `pcat` so you can run it from anywhere, follow these steps:

1. **Move the script to your local bin:**
   ```bash
   sudo cp pcat /usr/local/bin/pcat
   ```

2. **Make it executable:**
   ```bash
   sudo chmod +x /usr/local/bin/pcat
   ```

3. **Verify the installation:**
   ```bash
   pcat --version
   ```

## One-liner Installation (Direct from GitHub)
You can also install it directly without cloning the whole repo:
```bash
sudo wget https://raw.githubusercontent.com/TheMaster1127/pcat/main/pcat -O /usr/local/bin/pcat && sudo chmod +x /usr/local/bin/pcat
```

## Windows Installation
1. Copy `pcat` to a folder (e.g., `C:\bin`).
2. Add `C:\bin` to your system **PATH** environment variable.
3. You can then run it via `python pcat`.

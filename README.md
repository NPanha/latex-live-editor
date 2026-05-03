# LaTeX Live Editor with Docker Container

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- [VS Code](https://code.visualstudio.com/) installed

No local LaTeX installation is required. Everything compiles inside Docker.

## Step 1: Install LaTeX Workshop in VS Code

1. Open VS Code
2. Press `Ctrl+Shift+X` to open the Extensions panel
3. Search for **LaTeX Workshop** (author: James Yu)
4. Click **Install**

## Step 2: Install and Set Up Docker Desktop

### 2.1 Download Docker Desktop

1. Go to [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. Click **Download for Windows**
3. Run the installer (`Docker Desktop Installer.exe`)
4. When prompted, keep **Use WSL 2 instead of Hyper-V** checked (recommended)
5. Click **OK** and wait for the installation to complete
6. Restart your computer when asked

### 2.2 Enable WSL 2 (if not already enabled)

Open PowerShell as Administrator and run:

```powershell
wsl --install
```

Then restart your computer.

### 2.3 Start Docker Desktop

1. Open **Docker Desktop** from the Start menu
2. Wait until the bottom-left status shows **"Engine running"** (green icon)
3. You can minimize it — it will continue running in the system tray

> Docker Desktop must be running every time you compile LaTeX. It starts automatically on login by default.

### 2.4 Verify Docker Works

Open PowerShell and run:

```powershell
docker --version
```

Expected output (version may differ):
```
Docker version 29.4.0, build 9d7ad9f
```

## Step 3: Pull the LaTeX Docker Image

Open PowerShell and run:

```powershell
docker pull texlive/texlive:latest
```

> This downloads ~5 GB and is only needed once. Make sure Docker Desktop is running before this step.

## Step 4: Configure VS Code Settings

1. Press `Ctrl+Shift+P` → type **Open User Settings (JSON)** → press `Enter`
2. Add the following lines inside the `{ }`:

```json
"latex-workshop.latex.autoBuild.run": "onSave",
"latex-workshop.view.pdf.viewer": "tab",
"latex-workshop.latex.tools": [
    {
        "name": "pdflatex",
        "command": "docker",
        "args": [
            "run", "--rm",
            "-v", "%DIR%:/workdir",
            "-w", "/workdir",
            "texlive/texlive:latest",
            "pdflatex",
            "-synctex=1",
            "-interaction=nonstopmode",
            "-file-line-error",
            "%DOCFILE%"
        ]
    }
],
"latex-workshop.latex.recipes": [
    {
        "name": "pdflatex x2",
        "tools": ["pdflatex", "pdflatex"]
    }
]
```

3. Save the file (`Ctrl+S`)
4. Reload VS Code: press `Ctrl+Shift+P` → type **Reload Window** → press `Enter`

> **Why `docker` as the command?** LaTeX Workshop's built-in Docker mode does not work reliably on Windows. This configuration calls `docker run` directly, which works on all platforms.

## Step 5: Open the Project

```powershell
code "D:\Project\main.tex"
```

Or in VS Code: `File → Open Folder` → select the project folder.

## Step 6: Open Live Preview

1. Open `main.tex` in VS Code
2. Press `Ctrl+Alt+V` — the PDF preview opens in a side tab
3. LaTeX Workshop automatically triggers the first build

> The first compile takes ~15 seconds (Docker startup). Subsequent saves are faster (~5–8 seconds).

## Step 7: Edit and Preview Live

1. Edit anything in `main.tex`
2. Press `Ctrl+S` to save
3. The PDF preview on the right **updates automatically**

**Keyboard shortcuts:**

| Shortcut | Action |
|---|---|
| `Ctrl+S` | Save and trigger recompile |
| `Ctrl+Alt+V` | Open PDF preview panel |
| `Ctrl+Alt+B` | Manually trigger build |
| `Ctrl+Alt+J` | Jump from source to PDF location |
| `Ctrl+Click` on PDF | Jump back to the source line |

## Troubleshooting

**`pdflatex is not recognized` error**
> LaTeX Workshop's built-in Docker mode does not work on Windows. Make sure your settings use `"command": "docker"` as shown in Step 4, not `"command": "pdflatex"`.

**Docker not running**
> Open Docker Desktop before starting VS Code. The build will fail silently if Docker is not running.

**PDF preview not opening**
> Press `Ctrl+Alt+B` to manually trigger a build first, then press `Ctrl+Alt+V` to open the preview.

**Table of contents is empty**
> LaTeX requires two compilation passes to resolve internal references. The recipe `pdflatex x2` handles this automatically. If compiling manually, run the command twice.

**Build not triggering on save**
> Verify that `"latex-workshop.latex.autoBuild.run": "onSave"` is present in your settings, then reload VS Code with `Ctrl+Shift+P → Reload Window`.

##   For every time after (daily use):

> `That's it VS Code auto-builds on save, no other commands needed.`

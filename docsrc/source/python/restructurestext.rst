

Install on VSC:
https://docs.lextudio.com/restructuredtext/

1. Extensions (reStructuredText von lexstudio, Extension Pack for reStructuredText von lexstudio, esbonio extension )
2. python Umgebung auswählen (f1 -> python -> select interpreter) oder neue erstellen
3. https://github.com/vscode-restructuredtext/vscode-restructuredtext/issues/410
    conda create --name myenv to create an environment.
    conda activate myenv to initialize it.
    conda install -c conda-forge esbonio and conda install -c conda-forge sphinx_rtd_theme to install key packages.
    code . to open the project folder in VS Code and select the virtual environment in Microsoft Python extension.
    Use View | Command Palette... menu to locate Python: Select Interpreter.
4. Preview mit ctrl+shift+v
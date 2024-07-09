# Visual Studio Code

Conda - Python mit Terminal PS funktioniert nicht. Hier default auf cmd ändern in VSC:
* Configure your default profile by running the Terminal: Select Default Profile command, which is also accessible via the new terminal dropdown.
  

# C
https://code.visualstudio.com/docs/cpp/config-msvc

Install: 
  C/C++ IntelliSense

Install: 
  gcc 

f1 -> c/c++ Edit configurations ui
    -> hier kann Compiler-Erweiterungen eingetragen werden. 

Es wird im Verzeichnis .vscode angelegt mit c_cpp_properties.json, bspw.

```
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "/usr/src/linux-5.14.21-150400.24.100/tools/lib/bpf/"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "c17",
            "cppStandard": "gnu++17",
            "intelliSenseMode": "linux-gcc-x64"
        }
    ],
    "version": 4
}
```



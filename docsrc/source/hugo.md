# Hugo

https://gohugo.io/

## Installation

`sudo apt install hugo` - installiert go und weitere Abhängigkeiten\
`sudo apt remove hugo` - lösche die alte repo Version\
Download latest hugo from https://github.com/gohugoio/hugo/releases\
apt install /pfad-zu-file/hugo_extended_0.139.2_linux-amd64.deb hugo amd64

## Neue Seite erstellen mit Template

Je nach Installationsanleitung, bsp. Beautiful Hugo

```
hugo new site tecdoc
cd tecdoc
git init
git submodule add https://github.com/halogenica/beautifulhugo.git themes/beautifulhugo
hugo mod init github.com/USERNAME/SITENAME
hugo mod get github.com/halogenica/beautifulhugo
cp -r themes/beautifulhugo/exampleSite/* . -iv

edit hugo.toml
[[module.imports]]
  path = "github.com/halogenica/beautifulhugo"

hugo server
```

## Build your site

```
cd \<project\>; hugo
```

```
Hugo does not clear the public directory before building your site. Existing files are overwritten, but not deleted. This behavior is intentional to prevent the inadvertent removal of files that you may have added to the public directory after the build.

Depending on your needs, you may wish to manually clear the contents of the public directory before every build.
```


## Mermaid 

In beautifulhugo ist folgendes durchzuführen: 

1. mkdir layouts/_default/_markup

2. erstelle Datei render-codeblock-mermaid.html

    ```html
    <pre class="mermaid">
        {{- .Inner | htmlEscape | safeHTML }}
      </pre>
      {{ .Page.Store.Set "hasMermaid" true }}
    ```

3. unter layouts/partials/head_custom.html wurde dann der Code eingefügt

    ```js
    {{ if .Store.Get "hasMermaid" }}
    <script type="module">
      import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.esm.min.mjs';
      mermaid.initialize({ startOnLoad: true });
    </script>
    {{ end }}
    ```





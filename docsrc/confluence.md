# Confluence

## Markdown Integration

### Graphiken

https://medium.com/markdown-monster-blog/getting-images-into-markdown-documents-and-weblog-posts-with-markdown-monster-9ec6f353d8ec

Anonyme User sehen Bilddaten nicht in einem Markdowntext. Eine Möglichkeit ist, die Bildinformationen in Markdown zu embedden. 

1. Bild als png speichern 
2. https://base64.guru/converter/encode/image
3. Output, konvert to Data URI 
4. Im Markdown - Text: 

```
Here's an image:

![][image_ref_a32ff4ads]

More text here...
...

[image_ref_a32ff4ads]: data:image/png;base64,iVBORw0KGgoAAAANSUhEke02C1MyA29UWKgPA...RS12D==
```
# Triclops 
### Font file info collector

Drop font files (`.ttf`, `.ttc`, `.dfont`, `.otf`) into the panel to output JSON of the fonts with Postscript names. For use with [AEUX](https://aeux.io). 

![](./preview.png)


### Why do we need this?
Figma outputs [font info](https://www.figma.com/plugin-docs/api/FontName/) as an object. 

```
fontName = {
  family: string
  style: string
{
```

This works for Figma, but Adobe apps need a Postscript name. For many modern fonts these 2 properties may be combined to create the matching Postscript name like this:
```
fontName.family.replaceAll(' ', '') + '-' + fontName.style.replaceAll(' ', '')
```

```
fontName = {
  family: "Fira Mono"
  style: "Regular"
}
```
Becomes:
```
FiraMono-Regular
```
Which is the Postscript name. Yay!

But a lot of fonts don't match this format. 🤔
- Arial Regular: `ArialMT`
- Courier Regular: `Courier`
- HelveticaNeue Regular: `HelveticaNeue`


Sooooooo we need to manually create a substitution list based on available fonts and their Postscript names.


#### Triclops JSON format
```
{
    "Arial Black": {
        "Regular": "Arial-Black"
    },
    "Arial Narrow": {
        "Bold Italic": "ArialNarrow-BoldItalic",
        "Bold": "ArialNarrow-Bold",
        "Italic": "ArialNarrow-Italic",
        "Regular": "ArialNarrow"
    },
    "Arial Rounded MT Bold": {
        "Regular": "ArialRoundedMTBold"
    },
    "Arial": {
        "Bold Italic": "Arial-BoldItalicMT",
        "Bold": "Arial-BoldMT",
        "Italic": "Arial-ItalicMT",
        "Regular": "ArialMT"
    },
    "Arial Unicode MS": {
        "Regular": "ArialUnicodeMS"
    }
}
```
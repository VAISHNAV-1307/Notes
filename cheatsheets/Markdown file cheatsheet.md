# Markdown Cheatsheet

## Headers
```
# H1
## H2
### H3
#### H4
##### H5
###### H6
```

## Emphasis
```
*italic* or _italic_
**bold** or __bold__
***bold italic***
~~strikethrough~~
```

## Lists

**Unordered**
```
- Item
- Item
  - Nested item
```

**Ordered**
```
1. First
2. Second
   1. Nested
```

## Links & Images
```
[Link text](https://example.com)
[Link with title](https://example.com "Title text")
![Alt text](image.png)
[Reference link][1]

[1]: https://example.com
```

## Code

Inline: `` `code` ``

Block:
````
```python
print("Hello, world!")
```
````

## Blockquotes
```
> This is a quote
>> Nested quote
```

## Tables
```
| Header 1 | Header 2 |
|----------|----------|
| Cell 1   | Cell 2   |
| Cell 3   | Cell 4   |
```

Alignment:
```
| Left | Center | Right |
|:-----|:------:|------:|
| a    | b      | c     |
```

## Horizontal Rule
```
---
```
or
```
***
```

## Line Breaks
End a line with two trailing spaces, or leave a blank line between paragraphs.

## Task Lists
```
- [x] Done task
- [ ] Todo task
```

## Escaping Characters
```
\*not italic\*
\# not a header
```

## Footnotes (supported by some renderers)
```
Here's a claim.[^1]

[^1]: This is the footnote text.
```

## HTML in Markdown
Most renderers allow raw HTML, e.g.:
```html
<br>
<sub>subscript</sub>
<sup>superscript</sup>
```
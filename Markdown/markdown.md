# Markdown

[Back to README.md](../README.md)

*More info at [markdownlang.com](https://www.markdownlang.com/cheatsheet/).*

#### In page links
- [Tables](#tables)<br>
- [Links & Images](#links-images)<br>
- [Markdown In Page Links](#markdown-in-page-links)<br>



## Tables

[Back to Top](#markdown)

```markdown
| Column 1 Title | Column 2 Title |
|-|-|
| Row 1 Column 1 | Row 1 Column 2 |
| Row 2 Column 1 | Row 2 Column 2 |
```

Table content can be aligned with:

- Left align `|-|`
- Center align `|:-:|`
- Right align `|-:|`

## Links & Images

[Back to Top](#markdown)

```markdown
[Link Text](Link or Path/to/content)
[Link with Hover Text](https://www.markdownlang.com "Hover Text")

![Alt Text](image.jpg)
![Image with Hover Text](image.jpg "Hover Text")
![Image Link](image.jpg)](https://www.markdownlang.com)
```

## Markdown In Page Links

[Back to Top](#markdown)

Heading tag is a single `#` with all lowercase text.

```markdown
[Location](#heading)
```

NOTE: All `:`, `&`, and <code>``</code>characters will be stripped, so to link to <code>## Concept for `std::random_access_iterator`</code> use the following.

```markdown
[Concept for `std::random_access_iterator`](#concept-for-stdrandom_access_iterator)
```

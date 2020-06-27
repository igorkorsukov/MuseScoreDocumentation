# Documentation guidelines

Documentation is very important in development, it helps to coordinate efforts, gain knowledge and be effective.  
Treat documentation with the same responsibility as code.

## Where and what documents

If you want to know where to start, or how to build, or how to translate and much more, then look [Developers' handbook](https://musescore.org/en/handbook/developers-handbook)

This is the place for code documentation - code style, architecture, patterns, etc.

When writing code documentation prefer to create new directories for new sections and new files for separate topics so that the documentation structure is clear and one aspect is described in each file.
Each directory (section) must contain a `README.md` file, which is the root of this section.

## Text

For writing documentation we use standard [markdown](https://daringfireball.net/projects/markdown/) syntax (see [Markdown Cheatsheet](#markdown_cheatsheet))  
You can write markdown text in any text editor, in addition, you can use useful tools, for example: VS Code + markdownlint + Github Markdown Preview

## Diagrams

For creating diagrams we use [draw.io](https://www.draw.io)  
To add a diagram, you need to export the diagram in two formats:

* XML (__not compresed__) - used to store the original diagram so that we can edit it in the future. The diagram should be like uncompressed xml in order to be able to resolve conflicts.
* PNG - used to view the diagram in the documentation

The name of both files should be the same and contain the suffix `.drawio`, for example

```text
some_name.drawio.xml
some_name.drawio.png
```

## Markdown Cheatsheet

//! TODO need to write like [this](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)

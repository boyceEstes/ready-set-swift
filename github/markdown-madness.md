
* Headers (tbd)
* [Styling text](#styling-text)
	* [Quoting text](#quoting-text)
	* [Quoting code](#quoting-code)
* [Links](#links)
	* [Inline links](#inline-links)
	* [Section links](#section-links)
	* [Relative links](#relative-links)
* Lists  (tbd)
* [Spacing](#spacing)

Markdown is the syntatic language that is present in all Github platforms used to beautify and style writings. It is called "Markdown" because they were feeling fun and wanted to play on the fact that this is a 'markup' language.


# Styling text

## Quoting text
Use `>` to quote text

Ex:
```
In the words of Abraham Lincoln:
> Man, I love log cabins
```

In the words of Abraham Lincoln:
> Man, I love log cabins


## Quoting code
You can call out code or a command in a sentence with a single backtick.

Ex:
```
Use `git status` to list all new or modified files that haven't yet been committed.
```

Use `git status` to list all new or modified files that haven't yet been committed.


To format code into distinct blocks, use triple backticks \`\`\`.

Ex:
```
Some basic Git commands are:
\`\`\`
git status
git add
git commit
\`\`\`
```

Some basic Git commands are:
```
git status
git add
git commit
```

NOTE: Anything that is wrapped in backticks (\`) does not need an backslash (\) to escape markdown the markdown language. It is already escaped.

# Links

## Inline links
You can create inline links by wrapping text that you want displayed in brackets `[ ]` and wrapping the URL in parentheses immediately after `( )`.

Supposedly you can use `command + k` as shortcut to create a link but I'm not sure how this works from writing in a text editor.

Ex:
```
There are millions of homeless lions in Canada right now, see the full [story](https://fake-news.com)
```

There are millions of homeless lions in Canada right now, see the full [story](https://fake-news.com)


## Section links
Directly link a section in a rendered file by hovering over the section heading to expose the link: (Its the two link chain icon). It should show the URL with the addition of `#styling-text` corresponding to the header that is being called. 

After you have the anchor link, you can make an interactive table of contents like you would make most other links.

```
* [Styling Text](#styling-text)
```

Since github automatically resolves the relative path, it you can simply put the anchor to get your header.


## Relative links
Relative links and image paths can be defined in your rendered files to help readers navigate to other files in your repository.

For exaple, if you have a README file in the root of your repository, and you have another file in docs/CONTRIBUTING.md, the relative link to CONTRIBUTING.md in your README might look like this:

```
[Contribution guidelines for this project](docs/CONTRIBUTING.md)
```

If this looks familiar, its because this is the same format you would use for Inline links to external URLs.

Github will automatically transform your relative link or image path based on whatever branch you're currently on, so that the link or path always works. You can use relative link operands, such as `./` and `../`.

Try not to use absolute links as these might mess up for those that clone your project. 


# Spacing

## Linebreaks
These are only possible (so far in my experiementation) by `<br />`. Even this is not a good solution as it messed up headers for some reason.

Do not use the following when attempting to get more space between lines:
* `  ` - two spaces
* `\` - single backslash at the end of the line
* `&nbsp` - nonbreak space (kinda in the name)

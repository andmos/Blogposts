Ensure consistent Markdown style with Markdownlint
===

Markdown is great. It's easy and flexible, and provides a good markup language even non-technical people can understand and enjoy. But, that flexibility and customizability can come at a cost. Document buildup can be done in many ways, and it can be hard to ensure consistency when working with multiple documents and contributors.

I like to think of markup languages as code, and most code deserves a good style guide. [Markdownlint](https://github.com/DavidAnson/markdownlint) is a good alternative.

`markdownlint` provides [a nice set of standard rules](https://github.com/DavidAnson/markdownlint/blob/master/doc/Rules.md) when writing markdown, like:

* Heading levels should only increment by one level at a time
* Lists should be surrounded by blank lines
* First line in file should be a top level heading
* No empty links
* No trailing spaces
* No multiple consecutive blank lines

just to name a few. It also ensures consistency in headers, like

```markdown
My Heading
===
```

vs.

```markdown
# My Heading
```

Another smart rule is ensuring language description when writing code blocks.
The [VSCode extension](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint) shows a squiggle when a code block is missing a language description:

![MarkdownLint Example](https://user-images.githubusercontent.com/1283556/176104410-42b63ddb-ead2-4a38-b9a6-1a1ffcc82b97.png)

If some rules don't fit your style or project, they can be override with a `.markdownlint.json` file:

```markdown
{
    "MD013": false, // Disable line length rule.  
    "MD024": false // Allow Multiple headings with the same content.
}
```

The easiest way to start using `markdownlint` is to install the extension for [VSCode](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint) or [Atom](https://atom.io/packages/linter-node-markdownlint) (RIP Atom), or integrated with builds using [Grunt](https://github.com/sagiegurari/grunt-markdownlint), [Github Actions](https://github.com/xt0rted/markdownlint-problem-matcher) etc. My preferred way is directly with the [markdownlint-cli](https://github.com/igorshubovych/markdownlint-cli).

For my [Coffee recipes](https://github.com/andmos/Coffee) I use a simple container with Github Actions:

<script src="https://gist.github.com/andmos/a32940491b540ff5a1bf487ac0b26046.js"></script>

If any rules are broken, it breaks the build.

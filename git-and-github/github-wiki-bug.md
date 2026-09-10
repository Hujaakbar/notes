# GitHub Wiki Bug

GitHub Wiki is not rendering for various reasons.

Once it could be fixed by adding a new page locally and pushing it to GitHub.

```txt
git clone github-wiki.git
touch wiki_debug.md
git add .
git commit -m 'adding new page for debugging purposes'
git push
```

However, in early September of 2026, similar rendering issue happened again. The error was something like "Page is too big to load". Adding a new page and pushing it didn't resolve the issue. Upon trial and errors, the cause for the bug seems to be the utf-8 characters in the link. In my case, the link had Japanese characters.

my links had below characteristics:

- Japanese characters
- referring to other github wiki pages

example:

```markdown
[some link](github.com/xyz/wiki/サムリンク.md)
```

I temporarily *solved* the issue by disabling the links.

```markdown
`[some link](github.com/xyz/wiki/サムリンク.md)`
```

Note: There is nothing wrong with the links. It is a bug that GitHub should fix. But in the meantime above trick can be used.

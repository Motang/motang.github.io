---
layout: post
title: MarkdownExample
---

<meta charset="utf-8" />

> ## 这是一个标题。
> 
> 1.   这是第一行列表项。
> 2.   这是第二行列表项。
> 
> 给出一些例子代码：
This is an H1
=============
This is an H2
-------------

# This is an H1
## This is an H2
###### This is an H6
     return shell_exec("echo $input | $markdown_script");


[I'm an inline-style link](https://www.google.com)

[I'm an inline-style link with title](https://www.google.com "Google's Homepage")

[I'm a reference-style link][Arbitrary case-insensitive reference text]

[I'm a relative reference to a repository file](../blob/master/LICENSE)

Reference-style links use a second set of square brackets, inside which you place a label of your choosing to identify the link:

This is [an example][id] reference-style link.
You can optionally use a space to separate the sets of brackets:

This is [an example] [id] reference-style link.
Then, anywhere in the document, you define your link label like this, on a line by itself:

[id]: http://example.com/  "Optional Title Here"

# Inline
![Alternative text](/path/to/img.jpg "Optional title")

# Reference
![Alternative text][id]
[id]: url/to/image  "Optional title"


To produce a code block in Markdown, simply indent every line of the block by at least 4 spaces or 1 tab.

For example:

This is a normal paragraph:

    This is a code block.
    
This is a `inline code block`
You can define the language to be used for syntax highlighting by adding the name on the opening tag. Example:

```js
var a = {};
```

Here is an example of table with the output below:

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

Markdown | Less | Pretty
--- | --- | ---
*Still* | `renders` | **nicely**
1 | 2 | 3
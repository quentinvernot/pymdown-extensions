# Arithmatex

## Overview

Arithmatex is an extension that preserves LaTeX math equations during the Markdown conversion process so that they can be used with libraries like [MathJax][mathjax]. If you prefer to use something other than MathJax, Arithmatex can output a more generic format suitable for other libraries like [KaTeX][katex].

Arithmatex searches for the patterns `#!tex $...$` and `#!tex \(...\)` for inline math, and `#!tex $$...$$`, `#!tex \[...\]`, and `#!tex \begin{}...\end{}` for block math. By default, all formats are enabled, but each format can individually be disabled if desired.

## Input Format

By default, [`smart_dollar`](#options) mode is enabled for the `#!tex $...$` inline variant. With `smart_dollar` it is expected that the opening token (`#!tex $`) is to be followed by a non-whitespace character, and the closing to be preceded by a non-white-space character.  This is to help avoid false positives when using the dollar sign in traditional ways such as: I have $2.00 and Bob has $10.00.  The previous statement requires no escaping of the `#!tex $` character.  But when needed, the `#!tex $` character can be escaped using `#!tex \$`. `smart_dollar` can be disabled and will capture any `#!tex $...$` whose dollar symbols are not escaped (`#!tex \$`).

!!! example "Inline Examples"

    ```tex
    $p(x|y) = \frac{p(y|x)p(x)}{p(y)}$, \(p(x|y) = \frac{p(y|x)p(x)}{p(y)}\).
    ```

    $p(x|y) = \frac{p(y|x)p(x)}{p(y)}$, \(p(x|y) = \frac{p(y|x)p(x)}{p(y)}\).

!!! tip
    When using MathJax, for best results, it is advised to not use [`generic`](#options) mode, and configure MathJax without the `text2jax` extension since MathJax automatically detects Arithmatex's default output.

    If using generic mode (for libraries like KaTeX), Arithmatex will convert dollars to the form `#!tex \(...\)` in the HTML output. This is because `#!tex $...$` is extremely problematic to scan for, which is why MathJax and KaTeX disable `#!tex $...$` by default in their plain text scanners, and why Arithmatex enables `smart_dollar` by default when scanning for `#!tex $...$`. It is advised, if outputting in in `generic` mode, to not configure your JavaScript library to look for `#!tex $...$` and instead look for `#!tex \(...\)`, and let Arithmatex's handle `#!tex $...$`.

For block forms, the block must start with the appropriate opening for the block type: `#!tex $$`, `#!tex \[`, and `#!tex \begin{}` for the respective search pattern. The block must also end with the proper respective end: `#!tex $$`, `#!tex \]`, and `#!tex \end{}`. A block also must contain no empty lines and should be both preceded and followed by an empty line.

!!! example "Block Examples"

    ```tex
    $$
    E(\mathbf{v}, \mathbf{h}) = -\sum_{i,j}w_{ij}v_i h_j - \sum_i b_i v_i - \sum_j c_j h_j
    $$

    \[3 < 4\]

    \begin{align}
        p(v_i=1|\mathbf{h}) & = \sigma\left(\sum_j w_{ij}h_j + b_i\right) \\
        p(h_j=1|\mathbf{v}) & = \sigma\left(\sum_i w_{ij}v_i + c_j\right)
    \end{align}
    ```

    $$
    E(\mathbf{v}, \mathbf{h}) = -\sum_{i,j}w_{ij}v_i h_j - \sum_i b_i v_i - \sum_j c_j h_j
    $$

    \[3 < 4\]

    \begin{align}
        p(v_i=1|\mathbf{h}) & = \sigma\left(\sum_j w_{ij}h_j + b_i\right) \\
        p(h_j=1|\mathbf{v}) & = \sigma\left(\sum_i w_{ij}v_i + c_j\right)
    \end{align}

## MathJax Output Format

The math equations will be wrapped in a special MathJax script tag and embedded into the HTML. MathJax can already find these scripts, so there is no need to include and configure the `tex2jax.js` extension when setting up MathJax. The tag will be `#!html <script type="math/tex"></script>` for inline and `#!html <script type="math/tex; mode=display"></script>` for block.

By default, Arithmatex will also generate a preview span with the class `MathJax_Preview` that MathJax will hide when the math content is actually loaded. If you do not want to see the preview, simply set `preview` to `#!py3 False`.

## Generic Output Format

If [`generic`](#options) is enabled, the extension will escape necessary symbols and normalize all output to be wrapped in the more reliable `#!tex \(...\)` for inline math and `#!tex \[...\]` for display math (unless changed via `tex_inline_wrap` and `tex_block_wrap` in the [options](#options)). Lastly every everything is inserted into a `span` or `div` for inline and display math respectively.

With the default settings, if in your Markdown you used `#!tex $...$` for inline math, it would be converted to `#!html <span class="arithmatex">\(...\)</span>` in the HTML. Blocks would be normalized from `#!tex $$...$$` to `#!html <div class="arithmatex">\[...\]</div>`.  In the case of `#!tex \begin{}...\end{}`, begins and ends will not be replaced, only wrapped: `#!html <div class="arithmatex">\[\begin{}...\end{}\]</div>`.

## Loading MathJax

Arithmatex requires you to provide the MathJax library and provide and configure it to your liking.  The recommended way of including MathJax is to use the CDN. Latest version at time of writing this is found below.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js"></script>
```

Generally, it is best to add your own configuration to get exactly what you want. Here we show some simple examples of configurations done in JavaScript. We've provided two basic configurations below: one that is configured for Arithmatex's [MathJax Output Format](#mathjax-output-format), and one that works with the [Generic Output Format](#generic-output-format) by using `tex2jax`. These are a good starting point,so feel free to take them and configure them further. Please see the [MathJax][mathjax] site for more info on using MathJax extensions/plugins and configuring those extensions/plugins.

```js tab="Default"
MathJax.Hub.Config({
  config: ["MMLorHTML.js"],
  jax: ["input/TeX", "output/HTML-CSS", "output/NativeMML"],
  extensions: ["MathMenu.js", "MathZoom.js"]
});
```

```js tab="Generic"
MathJax.Hub.Config({
  config: ["MMLorHTML.js"],
  extensions: ["tex2jax.js"],
  jax: ["input/TeX", "output/HTML-CSS", "output/NativeMML"],
  tex2jax: {
    inlineMath: [ ["\\(","\\)"] ],
    displayMath: [ ["\\[","\\]"] ],
    processEscapes: true,
    processEnvironments: true,
    ignoreClass: ".*|",
    processClass: "arithmatex"
  },
});
```

Notice that in our generic configuration, we set up `tex2jax` to only load `arithmatex` classes by excluding all elements and adding an exception for the `arithmatex` class. We also don't bother adding `#!tex $...$` and `#!tex $$...$$` to the `inlineMath` and `displayMath` options as Arithmatex converts them respectively to `#!tex \(...\)` and `#!tex \[...\]` in the HTML output (unless altered in [Options](#options)). But we do have to enable `processEnvironments` to properly process `#!tex \begin{}...\end{}` blocks.

## Loading KaTeX

In order to use KaTeX, the generic output format is required. You will need to include the KaTeX library:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.9.0/katex.min.js"></script>
```

And the KaTeX CSS:

```html
<link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.9.0/katex.min.css">
```

Though KaTeX does have its own auto load script, we want to ensure it *only* loads math content from elements with the `arithmatex` class. Below is a script that would do just that. Notice we check for and strip wrappers `#!tex \(...\)` and `#!tex \[...\]` off the content of the elements and send it through the renderer. We also don't bother adding `#!tex $...$` and `#!tex $$...$$` to the `inlineMath` and `displayMath` options as Arithmatex converts them respectively to `#!tex \(...\)` and `#!tex \[...\]` in the HTML output (unless altered in [Options](#options)).

```js
(function () {
'use strict';

var katexMath = (function () {
    var maths = document.querySelectorAll('.arithmatex'),
        tex;

    for (var i = 0; i < maths.length; i++) {
      tex = maths[i].textContent || maths[i].innerText;
      if (tex.startsWith('\\(') && tex.endsWith('\\)')) {
        katex.render(tex.slice(2, -2), maths[i], {'displayMode': false});
      } else if (tex.startsWith('\\[') && tex.endsWith('\\]')) {
        katex.render(tex.slice(2, -2), maths[i], {'displayMode': true});
      }
    }
});

(function () {
  var onReady = function onReady(fn) {
    if (document.addEventListener) {
      document.addEventListener("DOMContentLoaded", fn);
    } else {
      document.attachEvent("onreadystatechange", function () {
        if (document.readyState === "interactive") {
          fn();
        }
      });
    }
  };

  onReady(function () {
    if (typeof katex !== "undefined") {
      katexMath();
    }
  });
})();

}());
```

## Options

Option            | Type     | Default                               | Description
----------------- | -------- | ------------------------------------- |------------
`inline_syntax`   | [string] | `#!py3 ['dollar', 'round']`           | Syntax to search for: dollar=`#!tex $...$` and round=`#!tex \(...\)`.
`block_syntax`    | [string] | `#!py3 ['dollar', 'square', 'begin']` | Syntax to search for: dollar=`#!tex $...$`, square=`#!tex \[...\]`, and `#!tex \begin{}...\end{}`.
`generic`         | bool     | `#!py3 False`                         | Output in a generic format suitable for non MathJax libraries.
`tex_inline_wrap` | [string] | `#!py3 ['\\(', '\\)']`                | An array containing the opening and closing portion of the `generic` wrap.
`tex_block_wrap`  | [string] | `#!py3 ['\\[', '\\]']`                | An array containing the opening and closing portion of the `generic` wrap.
`smart_dollar`    | bool     | `#!py3 True`                          | Enable Arithmatex's smart dollar logic to minimize math detection issues with `#!tex $`.
`preview`         | bool     | `#!py3 True`                          | Insert a preview to show until MathJax finishes loading the equations.

!!! warning "Deprecation"
    `insert_as_script` has been deprecated in 4.6.0. If you are still setting this, it will do nothing.  This option will be removed in the future so it is strongly advised to stop setting it.

    MathJax is, and has always been the default, and inserting as script is the default as it is the most reliable insertion format. With the new way previews are inserted, there is no need for the old text conversion method via `tex2jax`.

    To output for other math libraries, like [KaTeX][katex], enable `generic`.

---8<--- "links.md"

---8<--- "mathjax.md"

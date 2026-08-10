# Code Syntax Highlighting — Design

## Goal

Give fenced code blocks per-language syntax highlighting that renders on
**extremely old browsers** (think Netscape 4 / IE 5 era), while staying true to
the blog's minimal, plugin-free, "works without CSS" ethos.

Concretely:

- Keywords are shown in **bold**.
- Comments are shown in *italic*.
- Nothing else is styled (no colour, no background, no borders).

## Why this works on ancient browsers

The highlighting is done **at build time by Rouge**, Jekyll's default
highlighter (already available on the GitHub Pages native builder — no new
plugin). The browser never runs any JavaScript and never fetches a highlighting
library. It receives plain HTML: `<span class="k">def</span>` and friends,
wrapped in `<pre><code>`.

The only client-side requirement is CSS that maps two token classes to two font
effects:

```css
font-weight: bold;   /* keywords  */
font-style:  italic; /* comments  */
```

Both declarations are **CSS1** (1996). Grouped selectors and descendant
selectors are CSS1/CSS2. There is no CSS3, no custom property, no media query,
no `@import`. Any browser that understands CSS at all understands this; any
browser that does not simply shows the code as readable monospace text. The
feature therefore degrades perfectly with no CSS at all — matching the blog's
core promise.

## Constraints

- Built by **GitHub Pages' native builder**; only whitelisted plugins allowed.
  Rouge is the built-in highlighter, so **no plugin is added**.
- No client-side JavaScript, no external stylesheet, no web font.
- The site keeps its single inline `<style>` block in `_includes/head.html`;
  the highlighting CSS lives there so there is still exactly **one HTML file per
  page and zero extra requests**.

## Authoring

Authors write standard Markdown fenced code blocks with a language tag. Jekyll
(kramdown + Rouge) turns them into class-annotated HTML:

    ```python
    # greet the world
    def hello():
        return "hi"
    ```

`# greet the world` renders italic; `def` / `return` render bold.

Languages are whatever [Rouge supports][rouge-langs] (python, ruby, c, js,
bash, yaml, html, …). An unknown or missing language tag still produces a plain,
readable code block.

## Configuration

`_config.yml` states the highlighter choice explicitly (these match the GitHub
Pages defaults, so behaviour is unchanged — the point is to document intent and
pin the wrapper class the CSS targets):

```yaml
highlighter: rouge
kramdown:
  syntax_highlighter: rouge
  syntax_highlighter_opts:
    css_class: highlight
```

## Token classes

Rouge emits Pygments-style short class names inside a `.highlight` wrapper. The
CSS scopes everything under `.highlight` so it never affects prose.

- Keywords → **bold**: `k kc kd kn kp kr kt kv`
- Comments → *italic*: `c ch cm cp cpf c1 cs cd`

## Files touched

| File                         | Change                                              |
|------------------------------|-----------------------------------------------------|
| `_config.yml`                | Explicit `rouge` highlighter + `css_class: highlight`|
| `_includes/head.html`        | Fill the inline `<style>` with the two rule groups   |
| `posts/_posts/2016-04-01-my-first-blog-post.md` | Add a demo code block            |
| `README.md`                  | Document the highlighting feature                    |

## Out of scope

- Colour themes / syntax colouring beyond bold + italic.
- Line numbers, copy buttons, or any interactive affordance.
- Highlighting inline `` `code` `` spans (only fenced blocks are highlighted).

[rouge-langs]: https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers

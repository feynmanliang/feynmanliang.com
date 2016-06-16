+++
date = "2013-10-27T00:29:39+01:00"
draft = true
title = "MathJax and Syntax Highlighting (with Live Preview) on Ghost"

+++

## MathML Support (MathJax)
Thanks to [Patrick Edelman](http://www.patrickedelman.com/latex-ghost/) for the hot tip!

Adding LaTeX to Ghost is very simple. Open up `ghost/content/themes/YOUR_THEME_NAME/default.hbs`. Before `</body>`, insert:

```prettyprint lang-js
{{! Mathjax configuration}}
<script type="text/javascript" 	src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

<script type="text/x-mathjax-config">
MathJax.Hub.Config({
    tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}
});
</script>
```

Now test it out: `$\sum \frac{1}{n} = H\_n$` should yield $\sum \frac{1}{n} = H\_n$.

The configuration is not perfect, however. Action items include:

  - <s>Overridding Markdown parsing for `$...$` enclosed text. In particular, `$\sum_{n=1}^\infty \frac{1}{n} = H_n$` fails to render properly because `_..._` is used by Markdown for italics.</s> **edit 3/14**: Thanks to Filip Allberg for helping me figure this out, _you need to escape your underscores_. $\sum\_{n=1}^\infty \frac{1}{n} = H\_n$ renders fine if the subscripts are escaped (`\_` instead of `_`) like in `$\sum\_{n=1}^\infty \frac{1}{n} = H\_n$`
  - <s>Math embedding in live preview. I suspect this is not too difficult.</s> **edit**: I've posted directions on how to do this below.

## Syntax Highlighting
  Did you notice that the HTML/JS code block above was syntax colored?? Fortunately, this is super easy to do.

- Open `ghost/content/themes/YOUR_THEME_NAME/default.hbs`.
- Before `</body>`, insert:
```prettyprint lang-html
<script src="https://google-code-prettify.googlecode.com/svn/loader/run_prettify.js"></script>
```
- Specify highlighting and prettyprinting classes using MarkDown:

		```prettyprint lang-python
		def main():
		print(
    		"Finally... technical writing is as easy as %s" %
            	' '.join(map(lambda x: str(x), range(1,4))))
        ```

- **Appreciate** my function from last week in its colorful glory:

```prettyprint lang-python
def main():
    print(
        "Finally... technical writing is as easy as %s" %
        ' '.join(map(lambda x: str(x), range(1,4))))
```

## Live Previews
[Peter Schmalfeldt](http://blog.peterschmalfeldt.com/adding-syntax-highlighting-to-ghost/) wrote a great post on getting syntax highlighting to work in the publisher view as well.

#### Mathjax Live Preview
I was able to adapt Peter's to enable MathJax live previewing. To do so, edit `ghost/core/server/views/default.hbs` and before `</body>` add

```prettyprint lang-js
{{! Load and configure mathjax }}
<script type="text/javascript"     src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}
});
</script>

{{! Re-render MathJax in live preview}}
<script>
var timeout,
    entry = document.getElementsByClassName('entry-markdown')[0];

function mathjaxify()
{
  var preview = document.getElementsByClassName('rendered-markdown')[0];
  if (typeof(typeset) == "undefined" || typeset == true) {
    MathJax.Hub.Queue(["Typeset", MathJax.Hub, preview]);  // renders mathjax if 'typeset' is true (or undefined)
    typesetStubbornMath();
  }

  // Render the bits of math that have inexplicably still failed to render, while
  // leaving the rest alone. (If you try to typeset the whole page, it will break
  // other things)
  function typesetStubbornMath() {
    $(".MathJax_Preview").each( function() {
        if($(this).text() != "") {
        MathJax.Hub.Queue(["Typeset", MathJax.Hub, $(this).attr("id")]);
        }
        });
  }
}

// Listen for Key Presses if on Editor
if(entry) {
  jQuery(document).keypress(function(event) {
      clearTimeout(timeout);
      timeout = setTimeout(mathjaxify, 2001);
      });
}

// Check for Change of Post Selection
setTimeout(function(){
    jQuery('.content-list-content li').click(function(){ mathjaxify(); });
    }, 500);
</script>

```
Enjoy!


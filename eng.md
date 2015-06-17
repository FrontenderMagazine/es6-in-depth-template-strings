<article class="post">
*[ES6 In Depth][1] is a series on new features being added to the JavaScript
programming language in the 6th Edition of the ECMAScript standard, ES6 for 
short.*

Last week I promised a change of pace. After iterators and generators, we would
tackle something easy, I said. Something that won’t melt your brain, I said. We’
ll see whether I can keep that promise in the end.

For now, let’s start with something simple.

### Backtick basics

ES6 introduces a new kind of string literal syntax called template strings.
They look like ordinary strings, except using the backtick character`` ` ``
rather than the usual quote marks`'` or `"`. In the simplest case, they really
are just strings:

    context.fillText(<span>`Ceci n'est pas une chaîne.`, x, y);
    </span>

` ` 
But there’s a reason these are called “template strings” and not “
boring plain old strings that don’t do anything special, only with backticks”. 
Template strings bring simple[string interpolation][2] to JavaScript. That is,
they’re a nice-looking, convenient way to plug JavaScript values into a string.

There are one million ways to use this, but the one that warms my heart is the
humble error message:

    <span><span>function <span>authorize<span>(user, action) {
      <span>if (!user.hasPrivilege(action)) {
        <span>throw <span>new <span>Error(
          <span>`User <span>${user.name} is not authorized to do <span>${action}.`);
      }
    }
    </span></span></span></span></span></span></span></span></span></span></span>

` ` 
In this example, `${user.name}` and `${action}` are called template
substitutions. JavaScript will plug the values `user.name` and `action` into
the resulting string. This could generate a message like
`User jorendorff is not authorized to do hockey.` (Which is true. I don’t
have a hockey license.
)

So far, this is just a slightly nicer syntax for the `+` operator, and the
details are what you would expect:

*   The code in a template substitution can be any JavaScript expression, so
    function calls, arithmetic, and so on are allowed. (If you really want to, you 
    can even nest a template string inside another template string, which I call
   template inception.)

*   If either value is not a string, it’ll be converted to a string using the
    usual rules. For example, if
   `action` is an object, its `.toString()` method will be called.

*   If you need to write a backtick inside a template string, you must escape
    it with a backslash:
   ``` `\`` ``` is the same as ``"`"``.

*   Likewise, if you need to include the two characters `${` in a template
    string, I don’t want to know what you’re up to, but you can escape either 
    character with a backslash:
   `` `write \${ or $\{` ``.

Unlike ordinary strings, template strings can cover multiple lines:

    $(<span>"#warning").html(<span>`
      <h1>Watch out!</h1>
      <p>Unauthorized hockeying can result in penalties
      of up to <span>${maxPenalty} minutes.</p>
    `);
    </span></span></span>

` ` 
All whitespace in the template string, including newlines and indentation, is
included verbatim in the output.

OK. Because of my promise last week, I feel responsible for your brain health.
So a quick warning: it starts getting a little intense from here. You can stop 
reading now, maybe go have a cup of coffee and enjoy your intact, unmelted brain.
Seriously, there’s no shame in turning back. Did[Lopes Gonçalves][3]
exhaustively explore the entire Southern Hemisphere after proving that ships can
cross the equator without being crushed by sea monsters or falling off the edge 
of the earth? No. He turned back, went home, and had a nice lunch. You like 
lunch, right?

### Backtick the future

Let’s talk about a few things template strings *don’t* do.

*   They don’t automatically escape special characters for you. To avoid 
    [cross-site scripting][4] vulnerabilities, you’ll still have to treat
    untrusted data with care, just as if you were concatenating ordinary strings.
   

*   It’s not obvious how they would interact with an 
    [internationalization library][5] (a library for helping your code speak
    different languages to different users). Template strings don’t handle language-
    specific formatting of numbers and dates, much less plurals.
   

*   They aren’t a replacement for template libraries, like [Mustache][6] or 
    [Nunjucks][7].
    
    Template strings don’t have any built-in syntax for looping—building
    the rows of an HTML table from an array, for example—or even conditionals. (Yes,
    you could use template inception for this, but to me it seems like the sort of 
    thing you’d do as a joke.
    )

ES6 provides one more twist on template strings that gives JS developers and
library designers the power to address these limitations and more. The feature 
is calledtagged templates.

The syntax for tagged templates is simple. They’re just template strings with
an extratag before the opening backtick. For our first example, the tag will be
`SaferHTML`, and we’re going to use this tag to try to address the first
limitation listed above: automatically escaping special characters.

Note that `SaferHTML` is *not* something provided by the ES6 standard library.
We’re going to implement it ourselves below.

    <span>var message =
      SaferHTML<span>`<p><span>${bonk.sender} has sent you a bonk.</p>`;
    </span></span></span>

` ` 
The tag here is the single identifier `SaferHTML`, but a tag can also be a
property, like`SaferHTML.escape`, or even a method call, like 
`SaferHTML.escape({unicodeControlCharacters: false})`. (To be precise, any ES6
[MemberExpression or CallExpression][8] can serve as a tag.)

We saw that untagged template strings are shorthand for simple string
concatenation. Tagged templates are shorthand for something else entirely:*a
function call*.

The code above is equivalent to:

    <span>var message =
      SaferHTML(<var>templateData</var>, bonk.sender);
    </span>

` ` 
where `<var>templateData</var>` is an immutable array of all the string
parts of the template, created for us by the JS engine. Here the array would 
have two elements, because there are two string parts in the tagged template, 
separated by a substitution. So`<var>templateData</var>` will be like 
`<a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze" target="_blank">Object.freeze</a>(["<p>", " has sent you a bonk.</p>"]`

(There is actually one more property present on `<var>templateData</var>`
`<var>templateData</var>.raw` is another array containing all the string
parts in the tagged template, but this time exactly as they looked in the source
code—with escape sequences like`\n` left intact, rather than being turned into
newlines and so on. The standard tag[`String.raw`][9] uses these raw strings
.)

This gives the `SaferHTML` function free rein to interpret both the string and
the substitutions in a million possible ways.

Before reading on, maybe you’d like to try to puzzle out just what `SaferHTML`
should do, and then try your hand at implementing it. After all, it’s just a 
function. You can test your work in the Firefox developer console.

Here is one possible answer (also available [as a gist][10]).

    <span><span>function <span>SaferHTML<span>(templateData) {
      <span>var s = templateData[<span>0];
      <span>for (<span>var i = <span>1; i < <span>arguments.length; i++) {
        <span>var arg = <span>String(<span>arguments[i]);
    
        <span>// Escape special characters in the substitution.
        s += arg.replace(<span>/&/g, <span>"&amp;")
                .replace(<span>/</g, <span>"&lt;")
                .replace(<span>/>/g, <span>"&gt;");
    
        <span>// Don't escape special characters in the template.
        s += templateData[i];
      }
      <span>return s;
    }
    </span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span></span>

` ` 
With this definition, the tagged template 
`` SaferHTML`<p>${bonk.sender} has sent you a bonk.</p>` `` might
expand to the string`"<p>ES6&lt;3er has sent you a bonk.</p>"`
`Hacker Steve <script>alert('xss');</script>`, sends them a bonk.
Whatever that means.

(Incidentally, if the way that function uses [the arguments object][11] strikes
you as a bit clunky, drop by next week. There’s*another* new feature in ES6
that I think you’ll like.
)

A single example isn’t enough to illustrate the flexibility of tagged
templates. Let’s revisit our earlier list of template string limitations to see 
what else you could do.

*   Template strings don’t auto-escape special characters. But as we’ve
    seen, with tagged templates, you can fix that problem yourself with a tag.
   
    
    In fact, you can do a lot better than that.
    
    From a security perspective, my `SaferHTML` function is pretty weak.
    Different places in HTML have different special characters that need to be 
    escaped in different ways;
   `SaferHTML` does not escape them all. But with some effort, you could write
    a much smarter
   `SaferHTML` function that actually parses the bits of HTML in the strings in
   `templateData`, so that it knows which substitutions are in plain HTML;
    which ones are inside element attributes, and thus need to escape
   `'` and `"`; which ones are in URL query strings, and thus need URL-escaping
    rather than HTML-escaping; and so on. It could perform just the right escaping 
    for each substitution.
   
    
    Does this sound far-fetched because HTML parsing is slow? Fortunately, the
    string parts of a tagged template do not change when the template is evaluated 
    again.
   `SaferHTML` could cache the results of all this parsing, to speed up later
    calls. (The cache could be a
   [WeakMap][12], another ES6 feature that we’ll discuss in a future post.)

*   Template strings don’t have built-in internationalization features. But
    with tags, we could add them.
   [A blog post by Jack Hsu][13] shows what the first steps down that road
    might look like. Just one example, as a teaser:
   
    
        i18n<span>`Hello <span>${name}, you have <span>${amount}:c(CAD) in your bank account.`
        <span>// => Hallo Bob, Sie haben 1.234,56 $CA auf Ihrem Bankkonto.
        </span></span></span></span>
    
    ` ` 
    Note how in this example, `name` and `amount` are JavaScript, but there’s
    a different bit of unfamiliar code, that
   `:c(CAD)`, which Jack places in the *string* part of the template.
    JavaScript is of course handled by the JavaScript engine; the string parts are 
    handled by Jack’s
   `i18n` tag. Users would learn from the `i18n` documentation that `:c(CAD)`
    means
   `amount` is an amount of currency, denominated in Canadian dollars.
    
    *This* is what tagged templates are about.

*   Template strings are no replacement for Mustache and Nunjucks, partly
    because they don’t have built-in syntax for loops or conditionals. But now we’re
    starting to see how you would go about fixing this, right? If JS doesn’t provide
    the feature, write a tag that provides it.
   
    
        <span>// Purely hypothetical template language based on
        <span>// ES6 tagged templates.
        <span>var libraryHtml = hashTemplate<span>`
          <ul>
            #for book in <span>${myBooks}
              <li><i>#{book.title}</i> by #{book.author}</li>
            #end
          </ul>
        `;
        </span></span></span></span></span>

The flexibility doesn’t stop there. Note that the arguments to a tag function
are not automatically converted to strings. They can be anything. The same goes 
for the return value. Tagged templates are not even necessarily strings! You 
could use custom tags to create regular expressions, DOM trees, images, promises
representing whole asynchronous processes, JS data structures, GL shaders.
..

**Tagged templates invite library designers to create powerful domain-specific
languages.** These languages might look nothing like JS, but they can still
embed in JS seamlessly and interact intelligently with the rest of the language.
Offhand, I can’t think of anything quite like it in any other language. I don’t 
know where this feature will take us. The possibilities are exciting.

### When can I start using this?

On the server, ES6 template strings are supported in io.js today.

In browsers, Firefox 34+ supports template strings. Chrome 41+ with the “
Experimental JavaScript” preference, which is off by default. For now, you’ll 
need to use[Babel][14] or [Traceur][15] if you want to use template strings on
the web. You can also use them right now in[TypeScript][16]!

### Wait—what about Markdown?

Hmm?

Oh. ...Good question.

(This section isn’t really about JavaScript. If you don’t use [Markdown][17],
you can skip it.
)

With template strings, both Markdown and JavaScript now use the `` ` ``
character to mean something special. In fact, in Markdown, it’s the delimiter 
for`code` snippets in the middle of inline text.

This brings up a bit of a problem! If you write this in a Markdown document:

    To display a message, write `alert(`hello world!`)`.
    

it’ll be displayed like this:

To display a message, write `alert(`hello world!`)`.

Note that there are no backticks in the output. Markdown interpreted all four
backticks as code delimiters and replaced them with HTML tags.

To avoid this, we turn to a little-known feature that’s been in Markdown from
the beginning: you can use multiple backticks as code delimiters, like this:

    To display a message, write ``alert(`hello world!`)``.
    

[This Gist][18] has the details, and it’s written in Markdown so you can look
at the source.

### Up next

Next week, we’ll look at two features that programmers have enjoyed in other
languages for decades: one for people who like to avoid an argument where 
possible, and one for people who like to have lots of arguments. I’m talking 
about function arguments, of course. Both features are really for all of us.

We’ll see these features through the eyes of the person who implemented them
in Firefox. So please join us next week, as guest author Benjamin Peterson 
presents ES6 default parameters and rest parameters in depth.</article>

 [1]: https://hacks.mozilla.org/category/es6-in-depth/
 [2]: https://en.wikipedia.org/wiki/String_interpolation
 [3]: https://en.wikipedia.org/wiki/Lopes_Gon%C3%A7alves

 [4]: http://www.techrepublic.com/blog/it-security/what-is-cross-site-scripting/
 [5]: http://yuilibrary.com/yui/docs/intl/
 [6]: https://mustache.github.io/
 [7]: https://mozilla.github.io/nunjucks/

 [8]: https://people.mozilla.org/~jorendorff/es6-draft.html#sec-left-hand-side-expressions

 [9]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/raw
 [10]: https://gist.github.com/jorendorff/1a17f69dbfaafa2304f0

 [11]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments

 [12]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap
 [13]: http://jaysoo.ca/2014/03/20/i18n-with-es6-template-strings/
 [14]: http://babeljs.io/
 [15]: https://github.com/google/traceur-compiler#what-is-traceur

 [16]: http://blogs.msdn.com/b/typescript/archive/2015/01/16/announcing-typescript-1-4.aspx
 [17]: http://daringfireball.net/projects/markdown/basics
 [18]: https://gist.github.com/jorendorff/d3df45120ef8e4a342e5
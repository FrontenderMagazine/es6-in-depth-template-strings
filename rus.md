# ES6 в деталях: Шаблонные строки

_[ES6 в деталях][1] — это цикл статей о новых возможностях языка
программирования JavaScript, появившихся в 6 редакции стандарта ECMAScript,
кратко — ES6._

На прошлой неделе я обещал сбавить темп. Я говорил, что после итераторов и
генераторов мы возьмёмся за что-нибудь полегче. Я говорил, что это будет что-то,
что не плавит ваши мозги. Посмотрим, смогу ли сдержать своё обещание.

А пока начнём с чего-нибудь простого.

## Основы обратных кавычек

В ES6 появился новый вид синтаксиса строкового литерала под названием
шаблонные строки. Они выглядят как обычные строки за исключением того, что
обёрнуты символами обратных кавычек `` ` `` вместо обычных кавычек `'` или `"`.
В простейшем случае они действительно всего лишь строки.

    context.fillText(`Ceci n'est pas une chaîne.`, x, y);

Но они неспроста называются «шаблонные строки», а не «старые скучные
обыкновенные строки, которые ничего особого не делают, но только с обратными
кавычками».
Вместе с шаблонными строками в JavaScript появляется простая [строковая
интерполяция][2]. Иными словами, это способ аккуратно и удобно подставлять
значения JavaScript в строки.

Их можно применять в миллионах случаев, но моё сердце греет такое скромное
сообщение об ошибке:

    function authorize(user, action) {
      if (!user.hasPrivilege(action)) {
        throw new Error(
          `Пользователю ${user.name} не разрешено ${action}.`);
      }
    }

В этом примере `${user.name}` and `${action}` называются шаблонными
подстановками. JavaScript вставит значения `user.name` и `action` в получившуюся
строку. Так можно сгенерировать сообщение вроде
`Пользователю jorendorff не разрешено играть в хоккей.` (Что между прочим,
правда. У меня нет хоккейной лицензии.)

Пока что это просто слегка более опрятный синтаксис оператора `+`, и есть
несколько деталей, на которые следует обратить внимание:

*   Код в шаблонной подстановке может быть любым выражением JavaScript, так что
    вызовы функций, арифметика и т.п. разрешены. (Если вы действительно хотите,
    то можете поместить в шаблонной строке другую шаблонную строку, я это
    называю шаблонным Началом.)

*   Если какое-то значение не строкового типа, оно будет приведено к строке
    при помощи обычных правил. К примеру, если `action` — объект, у него
    вызовется метод `.toString()`.

*   Если вам нужно использовать символ обратной кавычки в шаблонной строке,
    её нужно заэкранировать обратным слэшем:
    ``` `\`` ``` — это то же самое, что ``"`"``.

*   Точно так же, если вам нужно указать в шаблонной строке два символа `${`,
    я не знаю зачем вам это может понадобиться, но вы можете заэкранировать
    любой из символов обратным слэшем: `` `пишите \${ или $\{` ``.

В отличие от обычных строк, в шаблонных строках можно использовать символы
переноса строк:

    $("#warning").html(`
      <h1>Внимание!</h1>
      <p>Несанкционированная игра в хоккей может повлечь пенальти
      на срок до ${maxPenalty} минут.</p>
    `);

Все пробельные символы в шаблонной строке, включая переносы строк и отступы,
включаются «как есть» в результат.

Хорошо. Из-за того, что я пообещал на прошлой неделе, я чувствую свою
ответственность за ваше мозговое здоровье. Вы можете прекратить читать прямо
сейчас, возможно, пойти выпить чашечку кофе и насладиться своим невредимым,
нерасплавленным мозгом.
Серьёзно, нет ничего постыдного в том, чтобы отступить. Что,
[Лопес Гонсальвес][3] ринулся целиком исследовать южное полушарие после того,
как убедился, что судна могут пересекать экватор не будучи разбитыми морскими
чудищами и не падая с края Земли? Нет. Он повернул обратно домой и хорошенько
пообедал. Вам же нравится обедать, верно?

## Backtick the future

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

    var message =
      SaferHTML`<p>${bonk.sender} has sent you a bonk.</p>`;

The tag here is the single identifier `SaferHTML`, but a tag can also be a
property, like`SaferHTML.escape`, or even a method call, like 
`SaferHTML.escape({unicodeControlCharacters: false})`. (To be precise, any ES6
[MemberExpression or CallExpression][8] can serve as a tag.)

We saw that untagged template strings are shorthand for simple string
concatenation. Tagged templates are shorthand for something else entirely:*a
function call*.

The code above is equivalent to:

    var message =
      SaferHTML(templateData, bonk.sender);

where `templateData` is an immutable array of all the string
parts of the template, created for us by the JS engine. Here the array would 
have two elements, because there are two string parts in the tagged template, 
separated by a substitution. So`templateData` will be like 
`<a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze" target="_blank">Object.freeze</a>(["<p>", " has sent you a bonk.</p>"]`

(There is actually one more property present on `templateData`
`templateData.raw` is another array containing all the string
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

    function SaferHTML(templateData) {
      var s = templateData[0];
      for (var i = 1; i < arguments.length; i++) {
        var arg = String(arguments[i]);
    
        // Escape special characters in the substitution.
        s += arg.replace(/&/g, "&amp;")
                .replace(/</g, "&lt;")
                .replace(/>/g, "&gt;");
    
        // Don't escape special characters in the template.
        s += templateData[i];
      }
      return s;
    }

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
   
    
        i18n`Hello ${name}, you have ${amount}:c(CAD) in your bank account.`
        // => Hallo Bob, Sie haben 1.234,56 $CA auf Ihrem Bankkonto.
        
    
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
   
    
        // Purely hypothetical template language based on
        // ES6 tagged templates.
        var libraryHtml = hashTemplate`
          <ul>
            #for book in ${myBooks}
              <li><i>#{book.title}</i> by #{book.author}</li>
            #end
          </ul>
        `;
        

The flexibility doesn’t stop there. Note that the arguments to a tag function
are not automatically converted to strings. They can be anything. The same goes 
for the return value. Tagged templates are not even necessarily strings! You 
could use custom tags to create regular expressions, DOM trees, images, promises
representing whole asynchronous processes, JS data structures, GL shaders...

**Tagged templates invite library designers to create powerful domain-specific
languages.** These languages might look nothing like JS, but they can still
embed in JS seamlessly and interact intelligently with the rest of the language.
Offhand, I can’t think of anything quite like it in any other language. I don’t 
know where this feature will take us. The possibilities are exciting.

## When can I start using this?

On the server, ES6 template strings are supported in io.js today.

In browsers, Firefox 34+ supports template strings. Chrome 41+ with the “
Experimental JavaScript” preference, which is off by default. For now, you’ll 
need to use[Babel][14] or [Traceur][15] if you want to use template strings on
the web. You can also use them right now in[TypeScript][16]!

## Wait—what about Markdown?

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

## Up next

Next week, we’ll look at two features that programmers have enjoyed in other
languages for decades: one for people who like to avoid an argument where 
possible, and one for people who like to have lots of arguments. I’m talking 
about function arguments, of course. Both features are really for all of us.

We’ll see these features through the eyes of the person who implemented them
in Firefox. So please join us next week, as guest author Benjamin Peterson 
presents ES6 default parameters and rest parameters in depth.

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
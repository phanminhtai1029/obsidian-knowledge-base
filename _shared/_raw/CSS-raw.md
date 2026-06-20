# CSS Fundamentals — Raw Transcript

_Tổng cộng 29 section — nguồn LinkedIn Learning transcript_

## Time to learn some CSS

CSS, or Cascading Style Sheets, is a core building block of the web. It takes plain old HTML and
turns it into something visually appealing. Basically, it's the secret sauce that makes the internet
a prettier place. In this course, I'll introduce you to the basics of CSS, how it works with HTML
and browsers, key concepts you need to know, and the most common ways you'll use it in real
projects. My name is Carrie Dils, and I've been wrangling CSS for over 20 years. If you're ready to
get comfortable with CSS and start styling the web like a pro, you are in the right spot. Ready to
go? Saddle up your horses and let's hit the trail.

## What is CSS?

If you're a designer or developer trying to stay on top of current web technology, you may feel
overwhelmed at the sheer volume of languages, and technologies out there. I frequently hear folks
say that they're not even sure where to begin, and I remember thinking the same thing when I started
out. I'd like to introduce you to CSS, and show you how it fits into the bigger picture of web
development. CSS stands for cascading style sheets. It's a style language used to control the
appearance of market languages like HTML. So you can think of HTML as controlling the structure of
the web while CSS controls the presentation of it. In other words, CSS is what makes the web pretty.
Think of the HTML as the structure of a new building. You can see the structure as it's being built,
but you don't really know what it's going to look like. That's where CSS comes in. It's the skin
that shows what the outside of the structure will look like. By separating structure and
presentation in this way, you can change how things look simply by changing the CSS. You don't have
to change the underlying structure. Now, when it comes to syntax, CSS is not a markup language like
HTML or a scripting language like JavaScript. It's a style sheet language. That means that CSS comes
with its own set of formatting rules to identify elements in a document and specify what style
should be applied. When it comes to using CSS with HTML, there are a few ways to apply CSS to a
file. First, you can apply the styles inline directly to an element. Next, you can apply styles in a
style block directly inside of a document. Or you can apply styles from an external style sheet that
ends in a .CSS extension. You can also use multiple style sheets for a single web page or website.
The cascading part of CSS means that style information from multiple sources can be combined. For
instance, you could have several external style sheets as well as inline styles. The tricky part
comes into play when there are multiple style rules for a particular piece of HTML. Which style
takes precedent? There has to be a way of figuring out which specific style sheet rule wins. The
rule used is chosen by cascading down from the most general rules to the most specific. The most
specific rule is always the one that wins the contest. A simple way to think of it might be
government laws. There are broad laws at a federal or a country level that apply. There are
additional state or province laws, but ultimately it's the local laws that rule. Similarly, external
style sheets tend to offer the broadest style roles. However, it's the inline styles that ultimately
win if they exist. Talking about the benefits of CSS probably the biggest one is that it provides a
separation between style and structure. Going back to our building example, we could paint it a
different color without changing a thing about its structure. You can also speed development, and
site maintenance by controlling the formatting for an entire site through a single style sheet.
Another great benefit of CSS is that it enables us to define and tailor styles based on how the
content is being consumed. For instance, you could optimize the presentation based on whether the
webpage was being printed or someone's viewing it on a phone or a tablet or desktop or whatever
other ways technology gives us to experience the web. So how does CSS fit into the bigger picture of
web development? Well, it's an essential part of the process. So if you want to learn front end web
development, learning CSS is a must.

## What are default browser styles?

Have you ever opened an HTML file that didn't have a style sheet? If so, it might have looked
something like this. Now, some people would refer to this as unstyled content, and while it
certainly does look that way it's not entirely accurate. Before users were allowed to control HTML
pages through cascading style sheets, browsers had to have some way of displaying HTML content
visually. These were called default browser styles or user agent styles. Essentially, those default
styles provide some basic structure to elements on a page so that you're not just looking at a big
jumble of text and images and such. There is some order here. Now, each browser has its own default
styles, which means that the same markup can appear inconsistently from browser to browser. Ask any
developer who's been around a while about the struggle of getting a design to look the same across
multiple browsers. And they'll tell you the old war stories of their battles against Netscape and
early versions of Internet Explorer. So why should you care about this? Well, when you write CSS,
one of the things you're actually doing is overwriting those default browser styles and replacing
them with how you want the elements to look. A lot of developers use something called CSS
normalization or CSS resets to address these browser inconsistencies. In essence, normalization
helps provide more consistency across browsers while the CSS resets approach strips out all default
browser styling so that you can style elements precisely the way you want. The other important thing
to remember is that users can override your styles anytime they want via most browser preferences,
they can choose a default font or font size to use, or even change the color contrast. Modern
browsers, even allow users to specify a dark mode. If you're a designer that spent hours perfecting
the layout, typography and color palette for a design that might make you cringe. But the important
thing to remember is this, the goal of any website should be to make the content accessible to the
individual user. Some people might have colorblindness or trouble reading low contrast text. Others
could be using a screen reader or a translation tool that doesn't care about the styles. That said
is design still important? Absolutely. And should you use CSS to control the presentation of your
content? You betcha. Just be aware that default browser styles are a thing as well as a user's
ability to customize how a browser displays content. These are the facts of life for a front end
developer.

## What is basic CSS lexicon?

Like any programming language, CSS has its own syntax or set of formatting rules to use. Thankfully
it's pretty minimal. So it's not too difficult to learn. The most basic description of the syntax is
that it includes a selector and a declaration block. The selector points to the element that you
want to style. And in this case, it's the HTML paragraph element represented by P. And the
declaration block, which is enclosed in these curly braces and can contain one or more declarations.
These declarations are simply the formatting instructions. So here, we've got a couple of
declarations or rules, one telling the browser which font to use, and another setting the font size.
The rules themselves are made up of two parts, the property and the value. These are separated by
colons, and there's a semicolon to tell the browser to stop evaluating and move on to the next role.
That's really the basics of CSS syntax. Even though the syntax itself is pretty straightforward,
there are tons of selectors and selector combinations that make it possible to get really granular
control over page elements. You'll also need to learn the various properties you can set for each
HTML element and the values that are allowed for that property. Now, while we're talking about
syntax, I do want to mention the use of spacing. As a general rule of thumb, spacing inside the
declaration block doesn't matter. In the example you see here, we get the same result with each of
these options. The spacing is primarily used for readability while you're writing your styles. And
then, once you've finished, you could minimize your styles, meaning that you can take all of the
white space out. When it comes to the selector, however, spaces do matter and are used intentionally
to target specific elements. You'll see more examples of this as we move through the course, but
know that it's important to be familiar with when spacing is important and when it's not. CSS also
lets you group selectors together if they share the same declarations. This is done by separating
each selector with a comma. In this example, both our P and our H6 elements were 18 pixels. So we
could go ahead and group those together and use a single declaration block. Like other programming
languages, CSS allows for comments. Comments are used to explain code and come in really handy when
you edit your CSS at a later date, or when someone else is viewing your code or trying to understand
what's going on. Comments are ignored by browsers, meaning they're purely there for documentation.
They don't impact the styles in any way. There are a couple of ways that you can write these
comments in CSS. There's the single line and multi line, as you can see in these examples. Both
start with the back slash asterisk, and then end with an asterisk and a back slash. So beyond what
we've covered so far in this movie, there are certain syntax rules you'll need to learn, such as
shorthand notation, pseudo elements, pseudo classes, and inline style rules. You'll see plenty of
examples of these during the course. My recommendation is to focus on these most basic elements of
CSS syntax first. You'll be surprised at how quickly you pick up the rest.

## What are basic selector types?

CSS selectors are what we use to find or target particular bits of HTML. The foremost common
selectors are type selectors, you may also hear these referred to as element selectors, the class
selector, the ID selector and the descendant selector. And the first one I want to start with is the
most basic, and that's the type selector. Type selectors are global in nature, meaning they're going
to affect every element of that type in a document. You just need to know what the tag is for a
given HTML element in order to style it. For example, to style paragraphs, you'll use p, for heading
ones, you'll use h1, for an unordered list, you'll use a ul and so forth. While these selectors are
very efficient, they're also pretty broad, which is why they're most often used to set global site-
wide styles. Another basic selector is the class selector. These match elements based on the
contents of their class attribute. So in this example, I've got a CSS class called error, and in my
HTML, I've got a paragraph tag with the class attribute containing the word error. And the output of
this code combo will give us the word oops in red. The syntax for a class selector is a period
followed by the class name. You can name a class anything you want and use it on as many elements
and as many times on a page as you need. This makes CSS classes pretty popular and one of the most
common selectors that you'll encounter. Next, we've got ID selectors. Unlike classes, IDs can only
be used once on a page. In this example, the browser would find the element on a page with an ID
attribute set to content, and then it would apply the styling specified by that ID selector. IDs are
identified by the pound symbol in front of the ID name. Lastly, let me introduce you to the
descendant selector. The descendant selector matches all elements that are descendants of a
specified element. So in this example, the browser would find any span elements nested inside of a
div element, regardless of how many levels deep down that span tag is. That's an important thing to
note about descendant selectors. They can apply to any nested element, regardless of how deep it's
found within the page structure. The syntax for this type of selector is written by stringing the
selectors together and separating them with a space. The parent selectors are added first, followed
by each successive nested selector. While there isn't a limit as to how many of those you can string
together, you don't want to go crazy, more than four starts to get in efficient, as in there are
performance implications. And being over specific with your styling makes it really difficult to
override that style later if you need to. That said, let's take a look at a couple of examples of
how you could use descendant selectors. In this first example represented by the orange here, the
browser would locate any paragraph found within a div tag and then apply the styling. In the second
one here represented by the blue, we're going to look for any anchor element inside of a paragraph
which also happens to be inside of a div, and then we'll apply the styling. If we use both of these
together, you can see that the style from the more specific selector combination is used. And that's
the anchor inside the paragraph inside the div. As you can see, descendant selectors allow you to be
very specific. Although there are certainly more selectors available than the ones I've shown here,
most of your styles will probably be written using the ones we've just covered. Learning how to
write efficient selectors based on your document structure gives you the power to style every
element on the page exactly how you want.

## What are CSS combinators?

In CSS lingo, a Combinator is something we can use to explain the relationship between selectors.
There are four of them, starting with the descendant selector that you learned about in the previous
lesson. It's noted by a space between selectors. Next there's the direct child selector, indicated
by the greater than sign. The adjacent sibling selector, indicated by the plus sign. And the general
sibling selector, indicated by the tilde. This will make more sense when you see it in action. So
let's take a look. So here's the descendant selector, and this says that any span inside of a div
matches. It doesn't matter if it's directly after the div or nested a few levels deep, all the spans
inside of this div are targeted. Let's turn this into a child selector by adding the greater than
sign between our div and our span. In this case, only span elements that are a direct child of the
div are selected. There's another span there inside of a paragraph, but it's not a match since it's
not a direct child of the div. Unlike descendant in child selectors which deal with parent-child
elements, sibling elements are on the same level, meaning they're not nested. The adjacent sibling
selector uses a plus sign to separate two selectors and adjacent means immediately following. So
this selector means that we get a match, only if we find a span after we've closed our div. The last
Combinator I want to take a look at is the general sibling selector. Unlike the adjacent sibling
selector, which only grabs the first instance after the specified element, the general sibling
selector targets all instances. So in this case, we've got all of the span elements adjacent to that
div that would be a match. Now, all in all, working with Combinators is definitely more on the
advanced side of CSS, but I wanted to introduce it to you at a high level, in case you find yourself
looking at some CSS and wondering what these characters mean, the plus sign, the tilde, et cetera.
In short, those characters help explain the relationship between selectors to help you target
exactly the bit of markup you want.

## How does CSS work with HTML?

I'd like to take a few minutes to talk about how CSS works with HTML structure. Now remember, that
to style elements in your documents, you'll need to write selectors that target those elements. The
more efficient and logical your HTML, the easier it is to write efficient CSS. CSS works with the
structure of your HTML document, allowing the browser to parse the HTML code, find the right element
and style it accordingly. Having well-structured, consistent HTML, and being familiar with that
structure, makes it a lot easier to write CSS for it. So let's talk about some best practices that
you can follow when writing HTML and how it impacts your CSS. For starters, try to use meaningful
names for IDs and classes. What do I mean by that? Well, let's take a look at this in my code editor
here. Here, I've got lots of classes and some IDs to define styling for all of these elements. Now,
as far as functionality goes, this would work just fine, but in terms of good HTML structure, this
is definitely not it. Well-structured semantic HTML should help you define your content and give it
meaning. Take this class here, "red-text". By itself, it means absolutely nothing other than I
intend for this text to be red. But what if later on, maybe in a design update, this bit of text
should become blue. Then I have a class that doesn't match my intention for that text. Classes and
IDs should be meaningful. Let me show you maybe a better way to go about this. So here on the left,
I've got the original and then here on the right is my attempt to clean this up a little bit. You
can see at a glance that I've removed a lot of the extraneous classes and IDs and replaced it with
some more meaningful markup. For instance, here, we have a div and it's got an ID of "wrapper", and
that's an indicator that this div will contain all of our HTML. Now here, I've got a section with an
ID of "main-content" instead of "section1". And that tells me a little bit more about what's going
to go into this section. Now here, instead of this div with an ID of "section2", I'm using the HTML5
markup of a side and giving it an ID of "sidebar" because that's what it is. So right off the bat,
you can see that aside with that ID selector of "sidebar" is much more meaningful than a div with an
ID of "section2", and so forth, and so on. Now, in terms of getting rid of all of these extraneous
classes, that's because I don't need them to style these elements. For instance, let's say that all
of my paragraphs are set to display at 16 pixels. So I don't need to give a special class to each
paragraph element, unless I want something to stand out or be different than the default. The next
rule of thumb is to make sure that your HTML is structured consistently across your site. There's
more than one way to structure the same kind of content. And sometimes, different authors do
something differently on different pages and that can make it considerably more difficult to write
global styles for an entire site. Let's take a look at this quote, for example. On one page, it's
structured with a div tag. On another page, it's structured with a block quote. And on that last
page is just represented in a normal paragraph. But both a div and the paragraph have a class of
quote applied. To style this consistently across the site, you'd need several different rules that
are essentially doing the exact same thing, but if you have a guideline for how content should be
structured and then making sure that you're consistent in following those guidelines, you can make
the process of writing styles much easier for yourself. My final piece of advice is to try and
structure the code so that you're adding meaning to the content without even thinking about the
styling. This approach might seem a little weird, but remember, the purpose of HTML is to provide a
structure for your content that helps convey meaning. And if you're able to do this for your
content, you'll find that you end up with much leaner, more descriptive code. That's a lot easier to
style. If you're still getting your feet wet with HTML and want to dig deeper into learning semantic
markup, there's a great course in the library I'd recommend.

## What options are available to write your styles?

When authoring CSS, you have a few options available as to where you can write your styles. In this
lesson, I want to cover those options and how they'll impact your overall site. Styles can
essentially be located in one of three locations. First, you can write CSS in a separate file, which
is called an external style sheet. Second, you can place styles in the head of an existing HTML
document, this is referred to as embedded styles. And finally, you can apply styles directly to an
HTML element, which is referred to as inline styles. Let's take a look at each of these in more
detail. External style sheets are simply text files with a .css extension. You'll commonly see them
named style.css or styles.css, but the name doesn't matter, so long as it has a .css extension. You
can author them with any text editor and they'll generally hold multiple styles that are used to
control an entire site or maybe a section of a site. And these styles are applied to pages by using
a link tag in the head of an HTML document. External style sheets are probably the most efficient
way of applying styles globally across the site. And then we have embedded styles, sometimes
referred to as an internal style sheet. And this is only applied to the documents that they're found
in, which makes them pretty inefficient and repetitive for site-wide styling, but perfect for
targeted styles or for a particular page. And finally, we've got inline styles. These are styles
added to an element as an HTML attribute. The syntax involves starting with the style attribute and
then following that with a semicolon separated list of CSS roles. I'm not a huge fan of inline
styles because they're very hard to overwrite and maintain over time. Editing an inline style means
that you have to track down the element that the style's applied to and edit the HTML code directly.
It's cumbersome and just plain inefficient. In terms of the cascade, inline style rules must always
win so it makes them very hard to overwrite. While there are some use cases for inline styles. be
intentional about how you use them. For the most part, you'll find that the majority of your
projects rely on external CSS files, with some occasional embedded styles used to overwrite those
global styles. Regardless of where you place your styles, the most important thing is to have an
overall strategy that controls site wide styling with consistency. That'll make it easier for you to
maintain or edit styles when necessary.

## How do browsers render styles?

When a browser displays a document, it's combining the document's content with its style
information. This process happens in stages. Now, if you're on a high speed internet connection,
you'll probably never even notice a delay, but if you're loading up, say, on a slow mobile
connection, well, you've probably seen times where the text of a webpage rendered before all the
styles or images did. All the same browsers do handle documents in a streaming fashion, and as your
learning CSS, it's important to understand how browsers render styles and apply them to documents.
First, I want to look at how browsers read CSS Selectors. As a native English speaker, I would look
at this and read it from left to right, so I'd be looking for an ID of main-content, a p-element,
and then a span element. CSS actually reads this from right to left, however. This tells the browser
to find any text within a span tag, which is also inside of a paragraph tag, which is also found
with an element with an ID of main-content. Next, I want to talk more about the concept of the
Cascade. Cascading style sheets get their name from the source order of the styles as they're
applied within the browser. And this usually means that user agent styles are applied first,
followed by external style sheets, then embedded styles, finally followed by inline styles. Now, if
these rules conflict with each other, the most recent set of styles will override the earlier
styles. Really, the Cascade can be summed up in one sentence, the last, most specific rule applied
wins. Styles are applied in the order that they're found, and recent styles always overwrite earlier
styles in the event of a conflict. That's the Cascade. Take a look at this style sheet, and notice
that we've got two paragraph selectors and they've got conflicting styles. One has a margin bottom
of 14 pixels and the other has a bottom margin of 18 pixels. So of the two of these, the last
selector, the one with the margin bottom of 18 pixels, that'll be the style that's applied. Just
remember that no matter where the styles are found, the last rule applied wins. Take this example,
since the external style sheet link appears after the embedded styles, the external styles would
actually overwrite any of the conflicting embedded styles. This concept even applies to style sheets
themselves. Next, I want to talk about an important concept called Inheritance, which means that
child elements will inherit some of the properties applied to a parent, though, not all of them, but
take this very common rule, for example, here we've got a body selector that has a font color, a
font family, and a font size. You can overwrite this, however. Let's say that I wanted all my main
headings to be blue. Well, I could simply write a rule for that H1 tag and set the color. When
there's a conflict, the most specific rule wins. Now imagine writing separate rules for every single
element on the page to define a font and a font color. Since these are all the same, we could simply
assign that color to the body selector and let all the other elements inherit it, that's a lot more
efficient. The last thing I want to touch on is Specificity. If you have two or more conflicting CSS
rules that point to the same element, there's some basic rules that a browser follows to determine
which one is the most specific and therefore wins. Basically, if the Cascade or Inheritance can't
resolve a conflict, then we look to Specificity to make a decision. We're going to dig deeper into
this a little later.

## Why CSS reset and normalize matter

Earlier, we talked about user agent style sheets, or default browser styles. I also briefly
mentioned that developers have options when it comes to, quote-unquote, taming those default styles.
Namely, Reset CSS, created by Eric Meyer, and normalize. There are actually a variety of other
browser resets in the wild, but for the sake of simplicity, we'll limit the discussion to these two.
Both of these approaches attempt to solve the problem of cross browser inconsistencies, but they go
about it in really different ways. Let me show you an example. Using CodePen, I've dumped some test
HTML in with all sorts of elements. I've not actually applied any styles to this, but it does look
like styled content. In this case, I'm using the Firefox browser. So what we're seeing is actually
just Firefox user agent styles. The reset CSS approach is meant to completely wipe out the built-in
styling for HTML elements. And that's done via this short and sweet bit of CSS. Now, if I go back to
my CodePen, I'm going to open up the settings, and there's an option here for CSS where I can apply
reset. I'm going to go ahead and do that. And whoa, that is a major difference. That's stripped out
all the styles. So this approach basically gives you a completely clean slate, but the downside is
that you would have to go back and style each of these elements, which could create more work for
you. Next, let's check out normalize. It's a more modern approach to reset, and it's under ongoing
development. So instead of wiping out all styles, it specifically targets only the styles that are
inconsistent across browsers. For instance, here are some styles for the H1 element, but not for any
of the other heading levels. That's because the other heading levels already behave consistently
across browsers. So normalize only resets the styles for the H1. Now let's go back to our CodePen,
and let's go back to just the user agent styles. Here we go. And now, let's try applying normalize.
While there is a visible difference with normalize, it's not nearly as drastic as CSS reset. So
should you use normalize or reset CSS or one of the many other variants out there? My friend Morton
asked the question of should you even be using a reset at all? I believe developers should use the
tools that fit them and their projects' needs best. So I'd encourage you to try out both, and see
the results to get a better feel for how it's impacting your design.

## Working with fonts

If you're a designer, you know that typography is incredibly important to a design and the message
you're trying to communicate. So it's no surprise that when learning CSS, a lot of folks start with
fonts. The CSS specification gives us all sorts of controls when working with fonts. But for this
course, I want to zero in on the most basic font properties, specifically the four most common ones
you'll see when working with CSS. First, we've got font-family, next font-weight, font-style, and
finally font-size. Now let's take a look at each one in turn. So here's the font-family and what
we've got, we're specifying the font for an element. In this example, it's the paragraph element.
Then we've got a prioritized list of fonts and we're also including a fallback font. So in this
example here, the first font that we want would be Times New Roman. Should that not be available,
then we'll go to Times. And finally, if Times wasn't available, then we'll just fall back to
whatever the system's default serif is. Next up, we've got font-weight and that lets us specify how
heavy or light a font is. Now, if we head over to the specification for the font-weight property, we
can see that it's got quite a few values here, normal, bold, bolder, lighter. And then these
numerical values where 100 is the very lightest and 900 is the heaviest. Now not every font has
every weight available. So let's say you specify a medium weight for a font and it's not available,
well, it would fall back to normal. Next up we've got font-style. And as the name implies, this lets
us specify the style for a font. We've got three possible values here, normal, which is just
default, italic and oblique. Now just like all the font-weights aren't available for every family.
So not every font family has an metallic or oblique version. So if you specify one of those and the
style is not available, then the browser will try to effectively fake it by tilting it. And this
might not bother most people, but if you love typography, this could drive you nuts. So just be
aware of that. The last of these most common properties is the font size. And as you've probably
guessed, this is how you can specify the size for a given element. Now when it comes to choosing a
font for your website, I want to talk about a couple of options. First, there are system fonts, and
then there are web fonts. In the past, we were limited to using a small set of fonts that come pre-
installed on almost every machine. These are called system fonts or fallback fonts. And here we're
looking at a list that the W3C, the World Wide Web Consortium recommends using for fallback fonts.
You're probably familiar with quite a few of these, Times New Roman, Georgia, Ariel, Tahoma, et
cetera. CSS also gives us the ability to display fonts that are not on a client's machine. These are
called web fonts. Some of them are free such as Google fonts and others require payment or licensing
like Adobe fonts. Web fonts are basically external style sheets that you can include in your project
and there's a few ways you can go about that. First, there's the CSS font face rule. This allows you
to download a particular font from your web server to render a webpage if the user doesn't have that
font installed. Similarly, we also have the import rule. You can use this from within an existing
CSS file to import an external font resource. I don't actually recommend this method as it's not the
most efficient in terms of performance. That brings me to the last couple of options for including
web fonts. You can link to an external style sheet or you can bring in a style sheet via JavaScript.
The latter method is what you'll commonly see if you're working with web fonts via Google fonts,
Adobe fonts, or a similar service. So those are some of the ways that you can use fonts in your CSS.
External fonts are really powerful as they give us the ability for brands to match typography across
both the website and printed assets. Be mindful however that for every font style and weight that
you load on a site, there's a corresponding performance implication. If you want to dig in further
on working with web fonts and optimizing them on a site, there's a whole chapter dedicated to web
fonts in this course.

## Understanding the box model

The CSS Box Model is the foundation of layout on the web. The box model is the term used to describe
the rules surrounding the physical properties of all HTML elements. Basically, you can think about
it this way. Every single element on your webpage is considered to be in a rectangular box, images,
text, forms, everything. And the properties of each box its width, height, border, margin, and
padding. These define not only the size of an element, but also, how it relates to the elements
around it. Let's start with the basics. So here we've got a visual representation of a paragraph.
Moving from the inside out, the first property I want you to notice is simply the element's edge. If
you don't specify a width or a height, then that's determined automatically based on how much space
the content takes up. You could also just specify a width and the height would be calculated
automatically. So in this case, let's say that the element is 100 pixels. By default, in the CSS Box
Model, the box-sizing property is set to content box. That means that if you set an element to 100
pixels, as in this case, then just the content box equals that 100 pixels. Any borders or padding
would be added on top of that. So the next thing we encounter is padding. Padding is the space added
to an element outside the element's edge. And note that you can define padding individually for the
top, right, bottom or left sides of an element. When you add padding, you increase the size of the
box accordingly. So here we've got 100-pixel wide box with 20 pixels of padding on all sides that
would give us a total of 140 pixels of total width. To change that behavior, you could change the
box-sizing property to a border box, which tells the browser to account for any border and padding
in the overall width that you specify for an element. Wrapping around the padding is the border.
Now, while padding is just clear space, a border is visible and can have color and thickness applied
to it. Like padding, a border can be applied specifically from top to right to bottom to left. It
can also be styled as solid, dotted, dashed, or a few other options. Finally, we have margin. This
represents the space outside the border and it's used to define the space between elements on a
page. Like padding, it's simply space and can be defined specifically for each side of the element
top, right, bottom, left. Now, unlike padding and borders, margin do not factor into the overall
width of the element. The box model itself, really isn't too difficult once you start to look at it,
but there are a few gotchas that can get you if you're not careful. First, just because a property
isn't declared don't assume that its value is zero. Remember that all browsers have default styles
and if you don't override them the default value will be applied. It's also quite common to see
authors set the width of an element to 100% in order to try and make it fit fully within apparent
element. For the most part that's okay, but unless you're using that box-sizing property any values
for borders and padding will force that total width to be over 100%. There are certainly other
aspects besides these to the box model, but these are the most basic ones that allow you to control
the dimensions of an element on a page. If you want to get another instructor's take on how the box
model works, here's an entire course dedicated to it. The title of this lesson also mentions
something called normal flow or flow layout. What the heck is that? Well, if the box model
determines how elements on a page relate to one another, then think of normal flow as the way
elements are laid out on a page, according to the HTML DOM. Since this is an intro course, I don't
want to get into the weeds of the ways that you can manipulate the flow of elements, but I did want
to simply introduce you to the term. We'll touch a little more on that later when we discuss
layouts.

## Setting margins and padding

As you work with CSS, you'll undoubtedly work a lot with margins and paddings. In this lesson, I'd
like to look a little closer at these properties. Padding is that clear or transparent space that
sits between the edge of an element and its border. And you can specify padding in several ways.
First, you can define padding individually for each side of the element, moving from clockwise from
top, right, bottom, and left. Now keep in mind, however, that setting padding in only one direction
is not the same as setting padding to zero and the others. For instance, if I set the padding top to
40 pixels, that doesn't necessarily mean that the right, bottom, and left paddings are set to zero.
Remember, there may be some default browser styles at play. As an alternative to specifying the
padding for each side individually, you can also use a shorthand notation, and there are four
variations on this. First, we can define four separate values separated by white space. Now these
values are going to apply in the order of top, right, bottom, and left, respectively. Next, you can
use three values. In this case, the first value will be applied to the top, the second value for the
left and right sides, and the last value will be applied to the bottom. Now next, if your top and
bottom padding are the same values and your right and left padding are the same values, you can use
a shorthand notation with just two values. In this case, the first value applies to the top and
bottom, while the second value will apply to the left and right sides. Finally, if all sides share
the same value, then you can note that via shorthand by just setting a single value for the padding
that's used for all sides equally. The rules for padding are actually pretty straightforward, and if
you're using a browser reset, then most elements start with no padding at all. Now let's move on to
margins. Margins are applied outside of an element's border, and represent the space between
elements. And while it's not factored in to the total width of an element, it does affect the amount
of space that an element takes up on the page. It's an important part of achieving proper spacing
between elements. Many elements have a default margin applied by user agents that you might need to
account for when structuring your layouts. Of course, if you've used a CSS reset or similar to
default your margins to zero, then you don't have to worry about it. But margins use the same syntax
as padding. So you can either specify margins for each side individually, or use the shorthand
notation that works just the same as padding. For the most part, margins are pretty straightforward
as well, but there is one aspect of margins that tends to trip up even the most experienced of
developers. Unlike horizontal margins, vertical margins are sometimes combined or collapsed into a
single margin whose size is the largest of the individual margins. That means that if you have two
elements stacked on top of one another, only one of the elements' top and bottom margins are
applied. I cannot tell you how many times this has tripped me up. Whenever you're working with CSS,
if something doesn't behave as expected, either you have incorrectly targeted an element or there's
a nuanced rule that you might not be aware of. That said, as you begin working with margins and
padding, you'll get the hang of it, and there are always developer docs you can search when it comes
to understanding those more nuanced behaviors.

## Working with color

When you're working with color in CSS, there are three primary properties you'll use. We've got our
background color property, border color, and color property. Many people often refer to this last
one as the foreground color. So in terms of elements like paragraphs or headings or lists, this
refers to the color of the text. Now, whether you're defining color for the borders, backgrounds, or
an element, you have a number of options in terms of how you want to define the color. I'll
highlight them quickly, and then we'll look at each one in turn. First, you can use color keywords,
next there's hexadecimal or hex notation. Then there's RGB color system, that stands for red, green,
blue. And lastly HSL, which stands for hue, saturation, and light. There are additional ways to
define color values in CSS, but these are the primary ones that you'll come across. Let's start by
using keywords. You can choose from one of the many color keywords supported by the CSS color
module. These keywords map to a predetermined set of values that browsers will use to display the
color, and note that these keywords work with all browsers. Now, if you want a specific color and
there's not a keyword for it, which is very likely, there are a few ways to represent colors as
numbers. The first is RGB, which generates all the colors from the combination of red, green, and
blue. The color is indicated by the letters, RGB, and then it's followed by three numbers separated
by commas, where the first number indicates the intensity of the red, the second indicates the
intensity of the green, and the third is for blue. Intensities are measured on a zero to 255 scale,
where zero measures the absence of that color, and 255 is the highest. So in this case, we've got a
box which is red at its highest intensity, which is 255. However, we can see that there are zero
elements of either the green or the blue at play here. Now, if we reduce the intensity of the red
all the way down to zero, the color gets progressively darker and darker as we get to the absence of
all colors, which is black, written as RGB 0, 0, 0. On the other hand, if we represent the color
with red and green at their highest values, we go through different shades of orange up to the color
yellow, which is a mixture of red and green at their highest intensities with no trace of blue. And
then if we increase the intensity of blue all the way up to 255, we get all three colors mixed at
their highest intensity, which creates the color white. So, by changing these intensities, we can go
through the whole spectrum of colors in this RGB notation. Next we've got the hexadecimal notation
to represent color values. Essentially, hex notation is another way of representing RGB color. It's
noted as a pound sign followed by three hex pairs representing the red, green, and blue values of a
color. With RGB, this full saturation of red is represented as 25, 0, 0. With hex notation, it's
represented as #FF for red, 00 for no green, and 00 for no blue. And similar to what we saw with the
RGB color picker, we can use the scale to help us determine exactly what the notation would be. Now,
because the calculations involved in converting RGB to hex, I think the hex color value is not quite
as intuitive, but you will still see it commonly used. And then there's HSL, which is a mixture of a
color's hue, saturation, and lightness. This is a more intuitive method of defining color, I think.
So the HSL notation is specified by the letters HSL, and then the parenthesis and three values that
correspond to hue, saturation, and lightness. Now, the color's hue is represented as an angle on the
color wheel, which ranges from zero to 360. Red is zero. Green is 120, and blue is 240. And other
colors such as yellow, orange, and violet are found in between those values. The saturation
represents the amount of color with 100% being fully saturated and 0% being gray scale. And then
lightness tells us how brighter dark the color is. So here, let's pick a particular blue and let's
say the saturation is maybe going to be around 50% and our lightness is going to be, oh, around 30%.
So that's how you would note that. And actually in this example, you can actually see how the color
value is referenced with both RGB and hex. Now, one more thing I want to show you about colors. For
both RGB and HSL, CSS gives us an additional alpha channel, which specifies the opacity for a color.
To do this, we use the RGBA notation instead of RGB, and that fourth parameter represents the alpha
on a scale of zero to one, where zero is fully transparent and one is fully opaque. And here, you
can see how those values translate to varying degrees of opacity for this red color. The idea is
similar for HSL with alpha. The notation becomes HSLA with the fourth parameter representing the
alpha value. Adding this fourth value to both RGB and HSL definitions allows you to have individual
transparency settings for backgrounds, borders, and foreground colors. I know that's a whole lot,
but just wanted you to see that CSS gives us tremendous flexibility when it comes to the way that we
define colors for borders, backgrounds, and elements.

## Applying borders and backgrounds

In this lesson, I'd like to talk with you about borders and backgrounds. And we'll start with
borders. The CSS border properties allow you to specify the style, width, and color of an element's
border. Similar to margins and paddings, borders can be defined from the top, right, bottom, and
left sides individually. Borders syntax can get really wordy if you're specifying the style, width,
and color individually. That's why I recommend going with the shorthand notation. This lets you set
all three border properties at the same time for each individual side. Now, if all four sides are
going to be identical, you can actually condense this shorthand further and just simply say border
and then give it the properties that you want. One important thing to remember about borders and
that's that they do affect the overall width of an element. Border widths begin at the edge of
padding widths and then extend outward. So in this instance, we've got an element that's 100 pixels
wide. It's got 20 pixels of padding on either side, bringing the width to 140 pixels. And then a
five pixel border on either side, bringing the total width for this element to 150 pixels. Now let's
talk a little bit about border styles. Border styles can be set in one of 10 values. There's none
and hidden, which are very similar, except when tables come into play, and then there's solid,
dotted, dashed, double, groove, ridge, inset, and outset. Now solid, dotted, and dash styles create
a single flat line. And correspondingly, it's either solid broken into dots or broken into dashes.
These remaining values create a dimensional line effect that often require a width value of more
than one in order to see the result. Now how about that border color property? This allows you to
define a specific color for your border that's independent of an element's foreground or background
color. Now, before we move on, I want to take a moment to discuss the difference between a border
and an outline. Their syntax is very similar, but there's an important distinction. An outline is
drawn just outside of the border's edge. While you'll see the border property used more frequently,
outlines play a very important role too, specifically, in the realm of usability and accessibility.
Let me show you an example. So here I've got just a super basic HTML form and I've applied an
outline that will display whenever a form field comes into focus. And by focus, I mean that either,
a user has clicked their cursor on a field or has navigated to a field via keyboard. Now the
outline, which I have super exaggerated for this example, draws attention to the element. This might
seem minor in this instance, but imagine a really crowded page or a page with maybe a bunch of form
fields, and it's easy to lose your cursor or your focus. In this case, the outline provides a visual
indicator showing me where I am on the page. And that's pretty helpful. Here are a few notes if
you're using outlines. It draws a line outside all sides of the box. So you can't specify particular
sides like you could with a border. Also, it's not part of the box model. So it does not affect the
position of the element or nearby elements. And lastly, assigning outline a value of zero, or none,
will remove the browser's default focus style. If an element can be interacted with, it needs a
visible focus indicator. Moving on, let's talk about backgrounds. The background property lets us
add visual sophistication and style to even basic content. The word background is actually shorthand
CSS, letting you define all background style properties at once such as color, image, size, et
cetera. Now the default style for most elements is to have a transparent background, meaning you can
see right through 'em. By explicitly setting the background property, you can choose to display
solid colors, gradients, or even images behind an element's content. They're really pretty cool.
When specifying a background image versus, say, a color, you can control its position; whether it
repeats along an axis, the background image size, and more. I should point out as well that it's an
entirely possible for an element to have both a background color and a background image. Background
colors are layered underneath background images and allow you to combine the two for interesting
visual effects. To sum up, borders and backgrounds are a great way to both add visual interest and
draw attention to specific elements on a page.

## Understanding the units of measurement

There are times when you need to set a link for a CSS element. CSS offers support for many units of
measurement, including those for both onscreen and print. Some of these units are absolute or fixed
length while others are relative. I'm going to limit our discussion here to the foremost common
types that you'll run across as you're learning CSS. First, we have pixels. Then we have ems, rems,
and finally, percentages. So pixels are referred to as an absolute value and just like an inch takes
up a physical amount of space, so does a pixel. With CSS, we can use pixels to define heights and
widths for things like block-level elements, padding, margin, font sizes, and here's a couple of
examples of those things in action. Next we have ems, and unlike pixels, these are considered
relative values. They're relative to the font size of a parent element. So the name actually comes
from print topography and it roughly means the value of text at its default size. What does that
mean? Well, let's say that we set our parent container to a 20-pixel font size. That would be equal
to one em. Two ems would be 40 pixels and so on. From there, you could set your paragraphs to one
em, maybe your headings to three em, and you could even use that em value for other properties like
margin or padding. The cool thing about ems is that you can change the starting point of the parent
element and the overall size of em units will get bigger or smaller as you've designated, but the
relationship between those elements will stay exactly the same. Next up, we have rems, or root ems,
which are very similar to ems in that they are also a relative value, but instead of being relative
to the value of their parent element, they're relative to the font size of the root element, which
is the HTML element. And if the HTML element doesn't have a specified font size, the browser default
value of 16 pixels is used. So one rem would be 16 pixels, two will be 32 pixels, et cetera. This
gives you a little more predictable control than ems, since we're relative to the root versus
relative to a parent container. Now, in the development and design communities, there's a lot of
debate over which unit of measurement is best for font sizes and ultimately, that's going to be up
to you and your design. Finally, we have percentages. And these are great for responsive layout, as
they let containers expand or shrink based on the size of the parent element. And just a note here,
percentages are typically used on block level elements and not used for font sizing. I'm just
scratching the surface. There's really a lot more when it comes to working with these and other
units of measurement. So just be aware that there are various options available to you when it comes
to setting the link of elements in CSS.

## Understanding cascading order

When I first started working with CSS, one of the hardest concepts I struggled with was the cascade.
It made sense in theory but then I'd find myself struggling to get my code to do what I wanted. In
this lesson, I'd like to demystify the cascade. So the CSS specification defines the cascade like
this. The cascade takes an unordered list of declared values for a given property on a given
element. It sorts them by their declarations precedence and then outputs a single cascaded value.
There's a lot there, so let's unpack that. We know that we can assign values to CSS properties and
then target the element that it applies to. So that's the first part of the definition. So in this
example, we've got multiple declared values for CSS properties that all apply to the site title. The
next part of that definition is that it sorts this list of styles by the declaration's precedence.
We'll talk shortly about what all goes into that but just make a mental note here that if any of
these selectors existed in isolation, the resulting output would be different for each. The final
part of that definition is that it outputs a single cascade value. Essentially, it weighs the
options and determines how to combine those. How does it do that? Well, the cascade is actually an
algorithm. It factors in the various parts of a declaration and assigns each one a weight. The
algorithm continues to cascade down until it finds a winner. Here are the things that the cascade
algorithm checks, listed in order from highest weight to least weight. We've got origin and
importance, specificity, the order of appearance, and inheritance. The highest weighted attribute
that the cascade checks is a combination of the origin and importance of a rule. Origins can be
either the user agent styles, any browser overrides set by an individual user, and then the CSS that
you, as the developer, bring to the party. The importance of a CSS declaration is determined by the
aptly named important syntax. Adding important to a CSS rule automatically jumps it to the front of
the cascade algorithm. The next part of the algorithm involves specificity. This is how the browser
decides which rule applies if multiple rules have different selectors, but could still apply to the
same element. For instance, here I've got the H1 element selector saying that any H1s should be
orange. But then I've got a class selector called entry header and it's applied to that H1 and it
says that the color should be blue. Well, what wins? In this case, the blue wins. That's because a
class selector is more specific than an element selector. Now, source order refers to where the
style rule is coming from, whether that's from a user agent styles, an external style sheet, et
cetera. But it can even refer to the order that rules are written within a single style sheet. For
example, here are two rules that could apply to the H1 element. The H1 ends up being colored orange
because both rules have an identical element selector and therefore carry the same specificity, but
the last one in the source order wins. The next concept is inheritance. Some CSS property values set
on parent elements are inherited by their child elements. For example, font color or font family can
be set on parent elements and then inherited by child elements. In this case, we're saying that
we're going to set all text in the document to black, and then that's going to be inherited through
all child elements unless a more specific rule pops up. And in this case, we have set spans to the
color orange. Now, child elements don't always inherit properties from the parent. For example, if
we set a width of 50% on a parent element, we wouldn't want all descendants to be 50% of that width.
We'd get incrementally smaller and smaller. So in a nutshell, that's how origins and importance,
specificity, source order, and inheritance work together to control how CSS is applied to HTML. And
that my friend is the cascade.

## Calculating specificity

Fun fact, each CSS rule actually has a weight score based on the elements that make up the selector.
So the score is comprised of how many IDs are used, classes, and elements. And you get points
respectively for each of those. So let's say we have an ID. Well, that gives us a point in this
column. So our specificity score will be 1,0,0. But you could generally just read this as the
number, 100. Now, if we have a Class that's going to be 010 or you could read it as 10. And just a
regular ole HTML element is going to get you one point. Let's play ~with some combinations here and
see how this works. So here, I've got an ID of sitetitle and that's worth 100. Now, what if I want
to specify that that only belongs on the heading one? Well, that's adding an element which adds a
point. And then what if we want to take this further and attach a class to it? Well, then we get
another point there for a specificity score of 111. So you can see that if you were using say just
the class of logo to target a specific element, but then somewhere else in your CSS, you had the
even just the ID that would weigh more. Or if you had the HTML element in the mix that would weigh
more as well. And when we talk about weighing more, well, it's the highest score that wins, which
makes it the most specific of the bunch. Practically speaking, I rarely find myself doing this math
but I did want to show you these calculations to show you that there actually is a method to the
madness that is CSS specificity.

## Considering the rule !important

In CSS, you can apply the !important property to add more weight or importance to a property value
pair than normal. For instance, in terms of the cascade, we know that, selectors being equal, the
last rule specified takes precedence over any previous declarations. So in this instance, our text
is orange because the second h1 declaration overrides the first. But what if we add !important to
the declaration? Well, in this case, our heading is blue. Applying !important to the declaration is
a very powerful way to ensure that a particular style is applied. So when should you use it? A
common-use case is when you're working with code that you don't have control over. For instance,
maybe you're embedding some code from a third party site, like a HubSpot form, a YouTube video, or a
real estate listing. The in embed code may contain styles that you have no control over. Having
!important in your toolbox allows you to target the elements you want in style and override styles
included with that embed. But with great power comes great responsibility. Generally speaking, do
everything you can to avoid using !important. It's considered a bad practice because it breaks the
natural cascade algorithm making debugging much more difficult. So when you're tempted to use
!important, here are some rules of thumb to consider. Always look for a way to use greater
specificity before resorting to !important. In other words, make a better use of the cascade by
using more specific rules. Next only use !important on page specific CSS that overrides CSS from an
outside source. Completely avoid !important if you're writing code for other developers to use and
avoid !important on site-wide CSS as it'll make your job harder in those instances where you want to
override a style. So !important is a tool you have available in your CSS toolbox, but try to use it
only if you have to.

## Structuring a page with CSS

In this movie, I want to introduce you to some layout concepts so that you'll be familiar with them
as you begin to experiment with and learn the various nuances around controlling page layouts with
CSS. So a browser reads an HTML document from the top down. Now if there's no CSS intervention, that
HTML will appear on the page as a bunch of stacked elements. So the question is, how do you take all
of these elements stacked on top of each other like little blocks and arrange them into rows and
columns to make an attractive layout? Well, there are a few approaches to taking an HTML document
and bending it to your will. The first is tables. Back in the day, there was no CSS. We just used
tables to create row and column-based layouts. This was terrible in so many ways as that's not
really what HTML table markup was meant for and it left us with a mess of unsemantic HTML,
inaccessible layouts, and code that was very difficult to maintain. Then came along cascading style
sheets with its box model to modify normal flow with floats and positioning. While this wasn't
really a layout system, it did give us the ability to manipulate where elements appeared on a page
relative to their neighboring elements. In the late 2000s, we started to see CSS grid systems
appearing either on their own or as part of larger CSS frameworks like Bootstrap or Foundation. Now
I'm calling all of these HTML layouts because they primarily rely on the actual structure of the
HTML document to put content where you'd want it on the page. Even with grid systems, a lot of them
relied on excessive CSS classes added to the HTML markup in order to achieve the desired layout. In
2016, the World Wide Web Consortium officially made a recommendation for Flexbox. This is a one-
dimensional flexible box module that lets us distribute content evenly and flexibly to fill all the
available space. This was a huge step forward in CSS layouts and is commonly used today. We also now
have something called CSS Grid. This is the first true two-dimensional layout tool in CSS that lets
us treat the view port like a canvas. The CSS grid radically changes how we design page layouts for
the web. Now other than tables, you'll still find the rest of these methods used in some degree to
another to create page layouts. You'll even find them working together in harmony. For instance,
sometimes CSS Grid may be used for overall structure of a layout while Flexbox might be used for
smaller modules within the overall layout. Really the main thing to remember is that when you're
starting off a new web project, be strategic about which approach you plan to take so that you can
be consistent with how you're applying CSS to your HTML markup.

## Understanding how flexbox works

Flexbox is shorthand for flexible box. It's a layout module for CSS that gives you an efficient way
to arrange, organize and size website elements to create responsive designs. Front end web
developers have embraced Flexbox for web-based layouts because it lets us arrange items within a
parent container, and then get pretty specific about how we want those items to behave in
relationship to one another. The way Flexbox works is pretty straightforward. You have a container
it's called the flex container or parent then you have the elements within the container and these
are referred to as flex items or children. The flex items are placed along flex lines which are
either a vertical or horizontal access. These items can be manipulated in layout, size, spacing and
even change order along the flex line. Let me say that one again. You can manipulate the order of
flex children with Flexbox without touching the HTML structure. Very cool. This allows you to take
advantage of the available view port space and lets elements arrange themselves automatically as the
view port dictates. Flexbox is a one dimensional presentation model. That means that layouts are
processed in one dimension at a time, either as a row or a column. You can define how space is
distributed between elements and how they should be aligned in relationship to the parent container.
There's a lot of benefits to Flexbox. Out of the gate, it's responsive and mobile friendly. It gives
you a lot of control over how you position child elements and you can easily change the presentation
order without ever touching the HTML. Also, you can easily alter and control both the width and the
height to best work within the parent element. So while Flexbox can do some really cool things it
generally works best on small scale layouts. Fully tackling Flexbox is out of the scope for this
course but just in case I've got your curiosity peaked, I'll share with you one of my very favorite
resources for Flexbox and that's this guide on "CSS Tricks". There's even a poster depicting Flexbox
layouts that I bought from here and hung in my office. My friends call it my nerd poster and I'm
pretty sure it's a compliment.

## Understanding how the CSS Grid works

CSS Grid is a newer CSS specification that gives us the most powerful way yet to control page
layouts. Unlike Flexbox, CSS Grid is a two-dimensional layout system. This allows developers to
place HTML elements in a Grid based layout using both rows and columns simultaneously. With Grid, we
can apply a Grid to a container and then put content wherever we want within that Grid. Completely
independent of semantic HTML markup. Very cool. It's the equivalent of a print based Grid layout
with the flexibility of responsive design. So let's take a look at some basic CSS Grid concepts and
terminology. So a Grid is an element with its display properties set to Grid. Grids have rows, and
those are horizontal lines, and columns, which are the vertical lines. Next we've got Grid lines.
Grid lines are the lines that make up the Grid and they can be horizontal or vertical. And a Grid
track is the space between two Grid lines. Grid tracks can also be horizontal or vertical,
representing rows or columns. A Grid cell is the space between four Grid lines. Top, right, bottom,
and left. And then a Grid area is any area within the Grid defined by its four outer most Grid
lines. And a Grid area can cover multiple lines and contain multiple cells. In short, you define a
Grid and you either allow items to be automatically placed into cells defined by that Grid, or
control their positioning. This is a huge step forward from where we've been with layout options. I
mentioned earlier that Flexbox was a huge improvement over previous methods of organizing and
positioning elements, but that it wasn't quite perfect as a full layout tool, and that's where CSS
Grid really shines. And actually, CSS Grid and Flexbox can work quite well together. There's an in-
depth course in the library showing how to combine these technologies.

## Understanding CSS frameworks

Earlier in the course, we talked about CSS reset and normalized CSS. Those are bits of code other
developers have written that you can use in your code to save you the time of writing it yourself.
Well, imagine that concept on a broader scale and you've got yourself a CSS library or framework. A
CSS framework is a collection of CSS files with predefined rules for commonly used things like
typography, UI components, layouts, and yes, even browser resets. There are benefits to using a
framework, such as cross-browser compatibility, consistent styling, responsive layouts, speeding up
your development time, and even learning CSS just by studying other people's code. There are a ton
of CSS frameworks out there and many have come and gone over the years. Some of the more popular
ones today include foundation sites, bootstrap, and tailwind. Now as flex box and grid have matured,
I see CSS frameworks playing a lesser role in web development than maybe they did a few years ago.
That said, CSS frameworks are certainly still alive and well. And it's just up to you whether you
choose to use one.

## Creating responsive layouts

The way people visit and interact with the web has changed significantly as technology has evolved.
In the past, web development was focused mainly on desktop computers and large monitors, but now we
have smartphones, smart TVs, wearable technologies, and all other sorts of ways to consume the web.
Responsive web design is a term used to describe an approach to web design or a set of best
practices that are used to create a layout that can respond to a device. CSS gives us multiple ways
to achieve this. One of the oldest ways to do this is through the use of CSS media queries. Media
queries can be used to check many things, such as the width and height of a viewport, the width and
height of a device, the orientation, like whether it's in landscape or portrait mode, as well as the
resolution of a device. The media role can even detect whether the device is a printer, an actual
screen, or speech, for screen readers that read a page out loud. Media queries let us perform these
conditional checks and then write CSS accordingly. Next there are modern layout methods such as
Flexbox and Grid and these are responsive by default. In Flexbox, flex items will shrink and
distribute space between the items according to the space available in their parent container. You
can specify values to indicate how you want those items to behave when there's either more space or
less space around them. CSS Grid allows the distribution of available space in a container across
grid tracks. So Grid responds natively to whatever the container size is. Using media queries, we
can also make typography responsive. For example, let's say we have a base font size of 20 pixels.
We could then scale that up or down accordingly using rem units. Alternatively, you could use the
viewport width to dynamically calculate font size. While an in-depth look at creating responsive
layouts is outside the scope of this course, I wanted you to know that one, it's a thing, and two,
that it's a very important thing. Creating responsive sites has become so important to web design
that in 2015, Google started actively penalizing sites that weren't mobile-friendly. As you're
learning CSS, be sure to keep in mind that while you're making beautiful websites, you also want
them to be accessible to as many people as possible. If you'd like to dig deeper into responsive web
design, I'd encourage you to search the library. There are plenty of resources.

## Organizing your code

It's entirely possible to write valid, working CSS but have the actual code be a hot mess of
confusion and inconsistency. I'd like to share some tips for keeping your CSS clean and well
organized. My first piece of advice is to use comments liberally. Comments help other people
understand your code, and frankly they help you understand your code if you've been away for a while
and come back to it. Sometimes I'll look at old code that I've written and thank my past self for
leaving my future self comments. Let's take a look at an example. So here we've got normalize CSS
and if we look at the source code we can see that indeed, the author has commented things liberally.
The comments are used to break code up into sections, and then within sections we've got
descriptions of exactly what's happening. CSS comments can be single or multi-line. They just need
to start with a forward slash asterisk and end with the same. But the characters in between can be
whatever. Various developers have their own systems for commenting CSS. So you'll see some different
styles as you look through other people's code but the bottom line is whatever you choose do
definitely use comments. The next thing I want to mention is to use consistent naming conventions.
And when I'm talking about naming conventions, I'm talking about how you name your CSS selectors
like your classes and your IDs. Are you using dashes, underscores, camel case, uppercase, lowercase?
Developers can get pretty opinionated over which conventions are best but really whatever you use,
the key is just to be consistent. The next thing I want to mention is to use consistent formatting.
What do I mean by that? Well, here in this example, we've got some CSS written and it's kind of all
over the place. Now, all of this is actually legitimate CSS. It works totally fine, but it's hard to
read and a little bit ugly. For instance, here we are defining a color for our paragraphs. It's all
on a single line and we've got no spaces. On the next line, we also have a single line declaration,
but we are using spaces. Moving further on, we've got this h1 that's on multi-line but we've got one
line indented, another line not indented and additional extra line that's not used at all and so
forth and so on. These sorts of inconsistencies in formatting makes CSS really difficult to read. So
again, different developers have different opinions for whether they want to do tabs, spaces, two
tabs, four tabs, whatever. Unless you're trying to follow a specific standard, it doesn't matter
which one you choose. The key is just to do it consistently. I also wanted to mention that you
should group like styles whenever possible. So taking this example, I've got my header one and my
header two further down the document. Ideally, you would group all your headings in a single place.
Likewise, you would group your header components maybe your site title, your logo, your navigation
in a certain area and likewise for other sections on your site. So that's just a general rule of
thumb that I prefer to follow when writing my CSS. My last step is to consider splitting up your
files. Now this isn't always necessary. It just depends on how much CSS you write or how large the
project is you're working on. By splitting up your CSS files into smaller chunks that can make it
easier to maintain, because you can jump really quickly into say the typography style sheet and make
adjustments rather than having to look through a really long style sheet to maybe find those
typography settings. The bottom line here is that by keeping your CSS organized, you'll ensure that
your code is easier to read and maintain over time.

## Using browser inspection tools

Modern browsers have built in developer tools that are incredibly helpful when building websites. As
you're learning CSS, you'll find the inspection tool to be something of magic. Let me show you what
I mean. So here I am on the wordpress.org site, and the website really doesn't matter, I just want
to illustrate the tool. So I'm using Firefox, and if I right click anywhere on the page, I get this
popup menu and I want to click Inspect. Now, one part of this tool is actually seeing the CSS and
HTL markup behind the page. So on this left panel, I have the HTML markup, and on this right side, I
have the corresponding styles. So for instance, if I wanted to see this main container with a class
of site header, I could go over here to the right in my CSS and see the styles for that. And here
we've got the padding, we've got the background color, et cetera. Now here's the really fun part.
The CSS on this is a live editor, meaning that you can test things out right here in the browser. So
for instance, let's say that I wanted to change the padding, we'll make it something absurd. And
maybe we want to change the background color, let's make it red. Now, I'm not actually saving this
page or making changes to the actual document, it's just letting me manipulate it here in the
browser. This is a really cool way to play with different CSS properties. It also shows you the
total computed width of elements on the page. You can also scroll through the HTML and it'll
highlight the corresponding element on the page. If nothing else, I find this tool is incredibly
helpful just for getting some of the basics of working with CSS properties. Here, I'm using Firefox,
but different tools exist for all the major browsers. And I should note, there's no need to install
these developer tools, they just come with the browser. So here they are for Firefox. Here's how to
access the DevTools in Chrome. Here's how to access the developer menu in Safari. And lastly, the
DevTools that are part of Microsoft Edge. Now, if that went too fast, just pause, and rewind this
video to see the URL for the browser that you'd like to use. As you start to style websites with
CSS, you'll find these tools one of your most favorite in your toolbox.

## Using the "Can I use..." for a CSS property

CSS is continually evolving. As a developer, you may want to take some new or experimental features
for a spin before they're fully supported in all browsers. What we're looking at here is the
official documentation for various specification modules, as well as their status. For instance,
here we have the Flex Box module and we can see that it's got a status of CR. Now, if we go down the
page further we can get an explanation for these status codes. And CR in this case means Candidate
Recommendation. These are basically stages that modules can move through from their very initial
development, all the way through to being a recommendation and fully supported or mostly fully
supported in browsers. Now, this is some pretty dense documentation on this page and odds are you
don't want to have to dig through it to find out whether you can use a particular CSS property. I'd
like to introduce you to your new best friend when it comes to understanding adoption of CSS
features across browsers. Caniuse.com provides up-to-date browser support tables showing which
frontend web technologies, like JavaScript, HTML5, and CSS, are available for use. What does that
mean? Well, you can simply type in the feature that you want to use, in this case we'll type in
flexbox. And what I get is a table here showing support across various browsers and browser
versions. There's also a statistic you can look at quickly to gauge how well a feature is supported.
In the case of flexbox we're over 95% usage. So that's pretty darn good. We can still see that the
status code is in CR or a Candidate Recommendation, so does that mean you shouldn't use flexbox? No,
it just means that it's still moving through the process of consensus. So let's take a look at
another one, the animation property in this case. And here you can see pretty widespread usage and
it's only actually in the status of Working Draft. Can you use it? Well, unless you happen to know
the majority of your users are on ancient versions of Firefox, Safari, or Opera, I'd say it's pretty
safe to use it. Of course, there are a lot of other resources on this page but basically I just
wanted to introduce you to caniuse.com and to share that if you're ever in doubt about using a
particular CSS property, this is your easiest source of truth.

## Validating your CSS with tools

Validating CSS or any code, for that matter, is simply a process of making sure that there are no
syntax errors or invalid code. It's important that before you add your styles to a live website,
that you're sure everything is error-free and working as you intended. I wanted to show you a couple
of online tools that you can use to validate your CSS. So here we have the w3c.org CSS validator and
you can use it simply by typing in a web address. You could actually upload a CSS file or just paste
your CSS directly in there. There's another similar tool called CSS Lint and when you hear the term
lint or linting or linter, we're not talking about what's deep in your pocket. We're talking about a
code cleaner of sorts, a code check or validator. Depending on which code editor you use, you can
generally find linters to add directly to your editor. So while these online tools are great for
validating CSS, it's even better if you could test it right there in your code editor. Now along
with validating your CSS, there are also some tools for helping you clean up your styles and make
sure that your formatting is consistent. One of them is this CSS Beautifier tool. Now I'm going to
take some ugly CSS and paste it into this spot and I get some options for how I want this indented,
and you can see over here on the right that it's cleaned up the output. It's made the spacing more
consistent and the indentation consistent, et cetera. Again, there are similar tools like this that
you could add directly to your code editor if you didn't want to use an online tool. It doesn't
matter how you go about it. The bottom line is that it's up to you to produce valid, clean CSS.

## Learning more about CSS

Thank you so much for joining me on this learning journey. We've covered a lot of territory and now
you are equipped to continue exploring on your own. In addition to the other resources I've
mentioned throughout the course, I'd like to highlight some resources in the library to help better
equip you to work with CSS. The first is CSS Essential Training. This course takes the topics I've
introduced to you here and expands on them in much greater detail. Next, there's HTML & CSS:
Creating Forms. As a web professional, it's almost guaranteed that you'll work with forms at some
point. This course teaches the ins and outs of building forms properly with HTML and then styling
them with CSS. Finally, there's Semantic HTML and CSS Code Challenges. I love this course. You'll
get down and dirty writing code and putting your knowledge into practice. This short course can
really dial in your skills. All right. I think I've given you enough to keep you busy, so I'm going
to leave you to it. I appreciate you watching and I hope to see you in another course soon.

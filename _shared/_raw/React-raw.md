# React — Raw Transcript

_Tổng cộng 34 section — nguồn LinkedIn Learning transcript_

## Building modern user interfaces with React

React is one of the most popular libraries for building user interfaces, and companies like Airbnb,
PayPal, and Netflix use it to power their frontend experiences. Having a good command of the
fundamentals will help you build projects of all sizes. It'll also prepare you to work with React
frameworks, like React Native, Vite and Next.js. In this course, we'll get hands-on experience with
React's core concepts, and we'll also explore concepts like data fetching, routing, and React server
components. So, let's get into it and start building some awesome composable apps with React.

## How to use Codespaces and what you should know

One of the ways that we can interact with the files in this course is to use GitHub Code Spaces. You
can just go to the GitHub repo for the course and add /codespaces to the end of it, and this will
take you to a location where you can start to create your own code space. You do that here with
create code space on main. It'll take a second for this to load up, but basically what this does is
it loads a VS code editor right here in your browser. So all of the files will take a little moment
to load up, but once they do, you're going to see them here on the left-hand side of the screen. So
this left nav is just what you would expect, and you also have the terminal embedded here so that
you can use that, too. This is going to give you the option to check out exact code at different
states. So let me show you an example. If we wanted to check out the exact code state at the
beginning of, let's say chapter four, video two, what we could do is run git check out O4_O2b, and
then this is going to go to that state. So it will have the react project at the beginning of that
video. Same goes for, say, git checkout 04_02e. If you want to jump forward at any moment you can.
So once you're in that directory, you can move to the react project for each one of these. CD React
project. You can run an NPMI or NPM install. This is going to install those dependencies in the
background so that you can actually run your project. All right, so this is the command that you'll
run to make everything work in code spaces, and then you can also take a look at the files and make
your edits right here in the browser. Nice. So that should be everything you really need to do. Now
that you know how to work with code spaces, you can follow along with me as we write these apps with
React.

## What is React?

React is a JavaScript library that you can use to create user interfaces. It was created at Facebook
and open sourced in March of 2013. Since then, react has expanded beyond the web with React Native,
a tool that allows you to create native mobile applications using React. One of your best resources
that you have at your disposal when learning React is the React documentation. Here, if you click on
Learn, you can see all of the latest ways of creating a project. You can check in with the
community, and there's also a blog where you can see the latest announcements about the library.
React is a popular library with hundreds of thousands of stars on GitHub and is used in production
by huge companies like PayPal, Netflix, and Microsoft. But the point of using React is not just
because it's popular, but because it's fun to use and because its component architecture makes it
faster and easier to build amazing products for the web and even for native platforms.

## How React works

Before we get into the course, let's think a bit about how React works. The core building block of a
React application is a React element. An element describes what you see on the screen, like an HTML
tag. So, if I create an element using React .createElement, I'm creating an h1, I pass that as the
first argument to that function, I pass null for the second because I don't want it to have any
properties, and then any child elements of that like Hello World, I'm going to add as the third
argument so that that displays when I come to the page. So, this is a DOM node that displays Hello
World. React apps are made up of React components. React components are typically one or more React
elements nested together. We create a React component using a function that returns part of our UI.
In this case, we're returning an h1. Here, we're using a syntax called JSX. This is a tag-based
syntax that compiles to plain JavaScript before we run it in the browser. We then create an app that
is made up of a bunch of different components. Think of these components as making up the DOM tree.
So, I have a header, a main, and a footer. All of these can be rendered together from the app. React
has a concept called props, where we can display dynamic data inside of a component. So, here, in
our function called header, we pass in props as an argument, and then props.title will display
whatever property that I've passed in to the header. In this case, it's vite. Then when the data
changes, our component will re-render, but React knows how to re-render in the most efficient way
possible so that we can hold onto the parts that don't change, and just make updates to the parts
that do. We will be getting hands-on experience with all of this, but this will give us a good
vocabulary to get us started.

## Understanding the React compiler

The new React Compiler, which is code named React for Git, is one of the most anticipated features
in React 19. And they built this playground that you can use to see how your React code goes from a
component to a function that is really optimized for the browser to be as fast as possible. The tool
that you can use to work with this is found at playground.react.dev. As soon as we head over there,
it's going to append this long, unique string to the end of it, and you also can share these with
other people. So we can use this to create our own React Component over here. So we'll say return,
and we want to return a header. Inside that header, we return an H1 that says, "Hello New React
Compiler." (keys clicking) So as soon as I do that, we're going to see that it spits out this
function over here. And this is really what our browser is going to be running versus this over
here. So think of React as being a compiled language. It's something that takes our React function
that's written with JSX or JavaScript as XML, and it transforms it into something that is optimized
for the browser. If you've used previous versions of React, where we used useMemo and useCallback,
these were hooks that we used to prevent unnecessary re-renders. But writing those hooks was error
prone and sort of annoying honestly because we would have to think about those every step of the
way. React is abstracting a lot of that back to the language layer, so that when we compile, it'll
automatically be optimized for production so that we can focus on just writing our components and
letting React do the heavy lifting of transforming our code into something that's as fast as
possible. So the React Compiler is already in production and has been at Instagram, and we'll see
this more and more as we adopt React 19 in our projects.

## Setting up Chrome tools for React

To get ready to work on React Projects, a great place to start is to install the React Developer
Tools. These are available for both Chrome and Firefox. So in Chrome here we're going to navigate to
the Chrome Web store, and from here I need to search for and install the React Developer Tools. I'll
select Add to Chrome, I'll say add extension, and these have been added to our extensions now. Up in
the upper right hand corner, you can see this. If you see it go away, you can always click on it and
expose that. What this is telling us is that this page is not using React. But if I go to a website
that I know uses React, like the React documentation, I'm going to see that the React Developer
Tools, this little logo is filled in now. And if I also open up my developer tools by clicking on
Inspect. I'll right click inspect. I'll click on the little arrows over here. And this will open up
Components. All of these components, these are the little building blocks of this page. We can
always inspect these elements and this can help us with debugging. We also see another tab here
called the Profiler. Sometimes it says Profiling is not supported because we aren't currently in
development, but this is going to be useful for us as we continue to work on these projects. Again,
you'll always see this logo filled in if you're on a page that has React running. And of course you
won't see it if you're on a page that doesn't. There also are React Developer Tools for Firefox. If
you're a user of Firefox, you can always come over here to the Mozilla add-ons. This will take you
through the process of installing those there. All right, there we go. Our Chrome browser is now
ready to develop some real projects using React.

## Working with VS Code

To work on the files in this course, you will need a code editor of some sort. You can use whatever
code editor feels right to you, but I will be using VS Code. So something that might be useful. You
can go up to code, settings, and then extensions. Then you can search for Prettier and I already
have this installed, but you can click on the install button and then this will be installed.
Basically Prettier is a code formatter that you can use to ensure that your style is consistent
across all of your projects. From here what we can do is go to code, settings, settings, and you can
search for format on save. Just make sure that format on save is selected. So anytime you click
save, it will format for you. So the final thing I want to do, the question that I get asked the
most in courses is, what is the code theme that you're using? The theme I use is something called
Night Owl. If you install Night Owl and you say set color theme, this will be your theme across your
projects. It's a nice accessible theme that looks nice and is highly readable. So once you install
this, there is also a Light Owl theme. So if you'd prefer light mode, this will give you that
option. While I'll be using VS Code, that's certainly not a requirement. You can always go to the
browser and go to react.new. This will open up a new CodeSandbox that you can use to create your
project. So think of CodeSandbox as just being an in-browser editor that you can use to do all the
same things. This will generate a new React project that's going to match exactly what we are
working on in our local version. So you can go to the source and go to index. From here, you can
always collapse that background with command B, so that'll collapse all of your files and then you
can zoom in and see the latest changes that you add. So all of this is our code. This is the window
that's going to give us a preview of what that code looks like, and you can always sign in with
GitHub, Google, or Apple to make sure that you hang on to those latest changes. All right, so that
is everything that we'll need to get started. In the next video we're going to generate our project.

## Initializing a project with Vite

The tool that we're going to be using to set up our project is called Vite. Vite is a tool that will
allow us to generate a React application and have some built-in tooling set up for us. So let's go
to VS Code and make sure we're in the right directory. I'm going to type cd to change directory and
then I'll drag O2_01b over here. Hit Enter and now I know I'm in the right spot. Next, I'm going to
run the command that's going to create the Vite project. So we'll type npm create vite@latest. We'll
call this react-project. We'll use a dash dash and then dash dash again this time followed up with
template react. So Vite has a bunch of different templates that you can use to get you started. It's
asking you to do a couple more things now, so we're going to do some of them. We'll say cd react-
project. And then I want to make sure that over here in 02_01b, I go to the React project. So I want
to go over here to 01_01b inside of the React project and I want to open the package JSON. So the
package JSON has all of the dependencies that I'm using in this project. I want to delete the
dependencies for React. I want to hit Save. Then I want to reopen the terminal. So let me clear the
terminal. We'll run npm install dash dash save dash exact react@rc and react-dom@rc. Okay, so now
that I've done that, this has installed the most recent versions of React, so React 19, the release
candidate, and now I can run npm install to run the rest of the packages. And then I can run based
on these scripts that are here. I can run npm run dev. This is going to start my project over here
on localhost:5173. In the browser, if I go to localhost:5173, this is going to show me my React
project working. How cool is that? So then if I go back to our source, meaning if I open the files,
I open the source directory and I open the app.jsx file. Now what I can do is I can say My React
Project here inside of the h1 and then I'll see that that's changed. So this is going to be the file
that we start to work with first as we go through the process of developing our React app. But this
is pretty cool, we have something started and now we can take it from here.

## Creating React elements

We've created our project using Vite, and I want to make some changes to the project so that it is
rendering what I expect it to. So over here in the REACT project, inside of the src directory, we
want to open the file that says App.jsx. So let's strip this way back and we're going to get rid of
everything that's inside of the App component for the most part, and we'll just write an <h1> that
says Hello React! Then, because we're not using these resources at the top, let's go ahead and
remove them. Nice. Let's also open up the main.jsx file. And just to take a look at this, what the
main.jsx file is doing is it's selecting the getElementById("root"), in other words, it's going into
this index.html file, It's finding the div that has the id of "root", and it's using the rules of
JavaScript over here to inject whatever should be added to the page. So if we take a closer look at
this, we can delete StrictMode for now. Let's delete it here and here. There we go. And all we're
really saying with this main.jsx file is go find that element and render this component. This is an
example of a REACT component that we're adding to the page. Then if we look at this file, it's just
a component. It's just an <h1> that we've added. So now if we open up localhost:5173, we need to
make sure our project is running. So from the react-project folder, we'll run npm run dev. Then if
we go to the browser, this is telling us that the project is working. We have our <h1> added to the
page. From here, we could make any changes that we want. Hello world!! Back here, automatically our
code will update, our browser will update to reflect all of those changes. So this is our first
example of creating a REACT element. We've added a return statement that wraps around the Hello
World!! This is an <h1> that is added to the page right here. So if we take a look at the elements
in the browser by hitting Command + Option + J and we go to Elements, you'll notice that there's the
div with the ID of "root", and this <h1> has been injected right there into that page. In the next
video, we'll talk a little bit more about JSX and all the powers that it gives us to create these
REACT elements to hold dynamic values.

## Working with JSX syntax

Right now within this app function, we are returning a little blip of JSX. So JSX or JavaScript is
XML is a syntax that you can use to write HTML like syntax right here in a React component right
here in a JavaScript file. So not only can this handle hardcoded text, but it also can handle
variables. So let's go ahead and create, I'll say language, I'll say JavaScript. Now, if I wanted to
take this value and place it into this string, I could. I'll just replace World with an open and
closed curly brace, and then I'll say language. So now if I run this project again, npm run dev,
this is going to take whatever that value is and dynamically insert it into that string, so that if
I change it, it will change here too. This works with all sorts of different things. So we could say
let moon equal, I'll hit Control, Command, space. That will open up my emoji keyboard. I can type
moon here, and then plug that in as a string. And then add that in another set of curly braces.
(laughing) Okay. And then finally we can think of these as being a place where code can run. So what
I mean by that is I could say language.toUpperCase. Add a string method here. And then that's going
to start shouting that at me. So this is how I use JSX as sort of a templating language for my React
code. I can create a variable, and then these variables can be injected into the body of the
component using those curly braces.

## Creating a React component

Now that we have an understanding of how to create our React elements, we want to create our first
component from scratch. Inside of this src folder, we have a file called App.jsx, and App is really
our first component. Think of a component as being like a little building block. It's a piece of the
user interface that we're going to use to describe one part of our application. So if I have a
function called App here, then I can create another component. So let's go ahead and clean this up.
We'll call this Hello React. And now on line 3, we'll create another component as a function. So a
React component, all it is is a function that returns a little bit of UI. These are JSX tags, but we
can think about these as being little HTML tags to wrap around our content using the same syntax
that we're familiar with. So now what I can do is within my App, I can take the contents of the
Header and place them here inside of this component. So check this out. I'm going to say return.
Typically we wrap multiple levels of these tags with a set of parentheses, and then we're going to
use the App component to render the Header. Nice. So now let's make sure that our App is running.
We're going to cd into the React project. We're going to run npm run dev. This is going to spin up
our Vite project here on 5173, and now we see, okay, we have our header here. If I right-click, I
select Inspect, I can go to the React Components tab over here in my Developer Tools, and this is
going to show me that this is the Header and the Header lives inside of the App. So let's add on to
this a little bit. Right now our Header is nice, but we might want to have some additional content
here. So to me it would feel like, all right, if we're going to render a couple things, then we
probably can do so, like so. We could say, here we got our h2 that says something. Okay, so spoilers
appear here in the fact that we have a bunch of squiggly red underlines under everything, letting us
know that probably something is wrong. So if we go to the browser, let's see if there are any
helpful hints for us. It says, adjacent JSX elements must be wrapped in an enclosing tag. So the
reason for this is because we're trying to render these two elements at the same level. In order for
this to work, all I need to do is I'm going to wrap this in those parens, and then I'm going to wrap
the whole thing inside of a div. So now if we go back, we see all of these components. Again, it's
useful to keep checking out your React Components Developer Tools just to show you what's what. So
we're following the rules here. We're wrapping these adjacent tags in an enclosing tag, and then the
App function is just going to render everything inside of that main component.

## Adding component properties

In the previous video, we created this Header component, and in this video, we want to take a look
at how we can use components to display dynamic data. So let's take a look. We know that when we're
using JSX, we can inject the names of variables into our components, but it's also possible to
display dynamic data from a very important object called props. So let me show you what I mean. Here
we have two components, App and Header. App is the parent component, It displays the Header
component, which in this case is the child component, and what we want to do here is use properties
to display some data. So check it out. We're going to say in the Header, we'll say name equals, and
then we'll set that name to "Alex". You can use whatever name feels right. It can be your kitchen,
absolutely. Now, before when we were using those curly braces, we just were able to inject the
variable name, but it's already crossing it out. It seems like that's not going to work. Let's just
prove that it won't over here for good measure. Let's run npm run dev, and at 5173, we see 's
Kitchen, which isn't quite what we're trying to pull off. So instead, let's take a look at this.
Let's console.log something called props, which we will pass in from the Header function on line
three, we'll go back to localhost:5173. And let's open up the browser console. So this is showing us
Object has a name, "Alex". So the way that we want to access this object is just by object literal
notation. So let's say {props.name}'s, Kitchen. Our strikethrough is gone, so I feel like we're back
in business. And now this value is being injected into the app. So when we render the component, we
pass the properties into the component like we've done here on line 15. And then we can reference
those properties on the props object that is passed in to each one of these values. So these can be
anything. We could say, <p>Copyright {props.year}</p> and then we could say year equals, because it
is a number, we'll wrap it in curly braces again this time when we pass that in as a property,
unlike the string, which we pass in in double quotes, and now this is going to say Copyright 2024 or
whatever year feels right. You could even do this, you could say new Date().getFullYear, and make
that dynamic as well. So now we are referencing the properties in the right location. We have our
child component. Let's get rid of that console.log so that we don't have to get rid of it later, but
just know that we're passing down properties here, we're accessing those properties here as an
argument that is sent to that function.

## Destructuring component properties

Let's take a look at a quick simplification that I can add to our header. Right now, it's taking in
this entire props object so that every one of our properties that's sent in can be accessed by using
dot notation, so we have props.name and props.year. So, another thing that we can take advantage of
is destructuring these properties. So, instead of passing in the entire object, I'm going to say
open and close curly brace, and we'll say name and year, then we can access them without having to
use props. in each location, so we're being really granular about which fields we actually want to
use. So, back to our application that's running, we should see that everything is being rendered
appropriately, because I'm grabbing it off of the object with destructuring. So, if you ever see
this syntax, just know that we're digging out these properties by their name, and then we can access
them by name and year here in the body of our component. So, this is just a shorthand syntax so that
we can destructure those values and make our code a little cleaner.

## Working with lists

Let's make an adjustment here to our component. We have a main tag here, but realistically I want
this to be a more dynamic component that's being rendered here on line 16. So if I create main
inside of the app, I also need to create main as its own component. And here we're going to return
an unordered list, and then the list should have a few different items. 1, 2, 3. So, assuming my
project is running with NPM Run Dev, I should see that on local host 5173, all of our list items are
appearing. But this would be more fun if this was dynamic data. So let's create a list of items. So
we'll say items, we'll set that equal to an array with square brackets, and then macaroni and
cheese, and salmon with potatoes, and tofu with vegetables. And obviously this is your own
restaurant, so feel free to add your own items there. Don't let me tell you how to run your
restaurant. What we want to do with these items is display them inside of this main function
component. So, here inside of the list items, we want to get rid of these, and instead pass those
in. So the place I'll pass those in, I'll say dishes is equal to items, so it's equal to whatever
this list of items might be. And think of that as being whatever dynamic data you want to create.
Nice. So from here what I can do is I'll pass this in. So we'll say dishes. We'll destructure those
dishes from the props object. Within our unordered list, we're going to open and close a set of
curly braces. This is a JSX expression that we're going to use here. So we can say dishes.map. For
each one of our dishes, we want to return a list item. And this list item should display whatever is
in this list. So whatever is here should be displayed as the list item. So now if we go back to our
application running, we see it's working. And the last thing that I can do here is for each list
item, I could say style equals, and then we're going to pass in a CSS style. So we'll use two sets
of curly braces. That's a fun little syntax thing that we can know about. And then we can say, list
style type none, back here, our bullets are gone. Now finally, if we open the console Command +
Option + J, this is going to let me know that each child in the list should have a unique key
property. So this is a rendering error that's very common, and we'll fix it in the next video.

## Adding keys to list items

In the previous video, we rendered a list of menu items dynamically, meaning that if I added another
item to this list, like minestrone soup, we should see that added automatically. We also saw in the
browser console though, that there was an error here. That each child in a list should have a unique
key property. A key is like an ID that helps you keep everything in sync. It's possible with React
that things might get out of sync, particularly if something is added to the DOM at the beginning or
the middle, instead of at the end of the list. So a quick way to fix this here on line 21 would be
to add a key called I, some sort of value that we can assign here on line 22 so that every time we
go through this little loop here, it's going to add this key dynamically. Now, as soon as I do that,
we see, let me clear out the console and refresh, we see that these items are added, but we no
longer have that key warning in the console. But the React documentation actually recommends against
this practice. It says that there can still be problems with rendering, which that's not really the
point of keys in the first place. So here's one way that I can handle this. I could create a value
called dishObjects, and then we're going to take these items, items.map, and we're going to use the
map too for each one of these items, create an object. So that's going to look like this. We'll say
dish and I, we want to return an object. So we're going to wrap the return in parentheses, and we'll
say ID is whatever I is. And then the title is whatever dish is. So let's console log dishObjects
just to see what this gives us. Ideally, if we go back to our browser console, we should see an
array of four items, each with an ID and a title. Pretty cool. What we can do with our dishObjects
then is pass it in to this property. So we'll say dishes is equal to dishObjects. Now back to our
project. There's some sort of an error, and that's because we're not referencing the properties
appropriately. So check this out. Dishes is being passed in, but we're going to say instead of I
here, we will say dish.id and dish.title. Back here, we should see everything rendering okay. Pretty
cool. So you might be thinking, "Isn't this the same thing as using an index again?" Not really,
because the data is stable. We've created it before our item is being rendered, not when it's being
rendered. So your data already might look like this too. You can just use an ID or any other unique
field as the key if you're dealing with an array of objects. So key is here, keep our data in sync
as our applications state, and our applications data changes throughout time.

## Displaying images with React

Now that we know how to add text to our React application, let's add an image. This can feel a
little confusing, but we can add them just like we would in an HTML file. So to start, I want to
point out that I have a new folder here called images. And in it there is a file called chef.jpg
that we are going to render. So, we're going to collapse this and decide where in the app this
should be placed. So, let's first import this. So we'll say import chef from "./images/chef.jpg".
Then we're going to place it inside of the project. So in the main, we want to make a few little
adjustments to the main component. First we want to wrap everything in a <main> tag and then closing
tag here. And from here we will add our image. So we'll say image src={chef}. So we'll use that
little JSX expression there. You could also set a property like height here. Remember that if we're
passing in a number of any kind, this will be passed in within our curly braces. A string would pass
in based on double quotes. So back here we see our image is being displayed right here in the
browser. Another thing that we always want to do is to give some sort of image description as an alt
attribute here to the image. So we'll say "A photo of a smiling chef owner". So this is Alex, our
smiling chef owner. And if anyone happens to use a screen reader for this, it will read the
description of that image. Okay, so from here, what we can do also, let's say I wanted to pass in
the URL of a different image. We're just going to use this little GitHub thing. A tricky little fun
thing you should know about GitHub is you can always grab your image by using whatever your GitHub
name is .png. And that's just a fun little test to make sure that that is working. But we don't want
this to be my restaurant. We want it to be this chef's restaurant. So we'll go back to that JSX
expression there. So we can always place these images inside of the images folder. We're going to
import that value, and then we can display it simply by using an image tag. And don't forget to add
an alt attribute so that your image is accessible.

## Using Fragments

Let's open up our app JSX so that we can demonstrate breaking something. So what we want to do first
is we're going to add a div here to our main component, and we're going to add a description of some
sort. So welcome to this beautiful restaurant. And if I'm being a good HTML-ian, I can wrap that in
an H2. So as soon as I do that, we notice a problem, because we see that squiggly underline. But
let's run our app to make sure that we understand why we see that. So if we type NPM run dev, so we
should see some sort of error. And this is letting us know that adjacent JSX elements must be
wrapped in an enclosing tag. Did we want a JSX fragment? They're spoiling the answer, but yes, I
totally do want a JSX fragment. And a JSX fragment exists to keep us from doing this. In the old
days of React before we had a JSX fragment, we would always be wrapping these adjoining tags in
usually a div. So that would make all of this code work, but then that starts to clutter up the DOM
with a bunch of extra divs. So if we take a look at our React components here, these are showing our
components, but we do in the React elements have a bunch of these enclosing divs. So this starts to
junk up the DOM tree here. So couple things we can do. We can replace this with React.Fragment, and
we would replace the opening tag with that too. And then we'll see React is not defined. That is
asking me to import it here at the top of the file. So if I refresh, clear that out, we should see
that we don't have any errors in the console and everything is working okay. But it's more than
likely that the real fragments that you'll see look like this. We'll get rid of React.Fragment, and
we'll just use this opening and closing little shorthand tag. We'll do the same here on line 48, and
then we should see everything working okay. And that will also not require me to import React from
React. So everything is working as we expect it to. So just to recap what we did here, we had a new
div and a main tag. And removing the fragment would break the app, because we're trying to render
two adjacent JSX elements. We're trying to render two different elements here at the same level. And
what React requires is that we wrap everything in an enclosing tag. If we use a fragment, this is
sort of like an enclosing tag that doesn't get recorded in the DOM. So there we go. We've used our
first JSX fragment.

## Destructuring arrays

Before we start embarking on our state management journey with React, it's going to be useful to
understand some syntax you'll be facing when working on these projects. So we've already talked
about destructuring in the context of the header here. We have the props object that we're going to
use object destructuring and dig out each key, name, and year from that object. But what happens if
we're dealing with an array? If I go over here to main.jsx, this is a good spot to just play around
with this. I'm going to create an array of animals. We'll say giraffe, zebra, and bear. Now, if I
want to access these values, I can't access them by key, because our items don't have a key.
Instead, I access them by position. So I could say console.log animals 0, and this will grab that
first item. So let's make sure that that's true. We're going to get into the 04_01c/react-project.
We're going to run npm run dev, and then over here in our browser console, we should see the first
item, giraffe. Now due to array destructuring, that's part of the ecma script spec, the JavaScript
spec, we can actually handle this slightly differently. So I could say here on line five, I'll open
and close a set of square brackets, and if I wanted to access the first item, I'll create a variable
name for it. So I'll say firstAnimal. We're going to replace this firstAnimal. And now back to our
browser, we should see giraffe. So all of these are accessed by position. So if we console logged
third, for example, that's going to give me bear. It's also possible to skip over these items by
using just a comma. So comma, comma third would still return bear. Nice. So this is helping me with
array destructuring. I open and close a set of square brackets, and then I can create a variable
name that's going to stand in for any one of these values. This is going to be super important in
the next video.

## Understanding the useState hook

Managing state in a React application is really important, and the most modern way to handle state
variables in an app is to use a function called useState. Let's take a look. The first thing I want
to do here on line one in our app.jsx file is import useState from react. This function is going to
be used to update the state of our application. So let's go ahead and collapse the header, collapse
the items, collapse our dishObjects, and collapse main, because what we really want to focus on
right now is our app component. This is where we're going to use that function. Now on line 53 here,
before we are returning anything inside of this component, I want to call the useState hook. So if
useState is this function, it needs to be set to something, and just out of curiosity, let's see
what this returns. So we'll say console.log what. So when I call this, what is that? So we can pop
this open, and we can see there is an array returned. The first item in the array is undefined. The
second item is this function. So this is telling me that I have a little container that I can put a
property and a function that's going to update that state property. So that's going to use array
destructuring, as you might expect. We open and close a set of square brackets, and then the first
item we place in here will be, let's call it status, and then we'll call this setStatus, and then
we'll set that equal to useState, and then let's go ahead and say Open. So at this point, if we go
back to our console log, there's an error, of course, because what is not defined? What is defined
is status, so if we run this, we can always clear out our console and hit refresh, and this is
telling us that our initial status is open. Let's also find, where am I logging there? I feel like
we can let that go, right? So let's go over to our main JSX file, and get rid of that. Little
cleanup, and now let's make use of our status here. So if I have a variable called status, setStatus
is a function that updates that variable, and then whenever I pass into useState is going to be the
default or the initial state whenever my application, whenever this component first loads. So check
this out. We're going to add here to the div, we'll say h1, and we'll say the restaurant is
currently status. So now here we can see the restaurant is currently open. Now if I change the
status, so the way that I would do that is we can add a button here. The button is going to say,
we'll say Close Restaurant, and then we will add an onClick. This onClick is going to call the
setStatus, and it'll take in whatever that new status is, so we'll say Closed. Once we give that a
save, we go back to our browser. We have a button here that says Close Restaurant. This is
displaying whatever the current value is, but if I update the state, the restaurant is currently
closed. So of course, there's way more that I can do with this. I probably want this eventually to
be a toggle so I can go back and forth, but at the moment, we're seeing how we set an initial state,
we create a state variable, we have a function that updates that state, and then anytime the state
changes, we're going to re-render this particular component, and then the set status function is
going to take in whatever we want that new status to be.

## Toggling state with a button

Right now, our app component is being rendered and when I click on the close restaurant button,
that's going to update the state with the new state that I pass into the set status function. But
this is kind of a one time thing where we can only click the button once and then any other time I
try to click it, we don't see anything happening. So let's make a change to this so that we get the
full experience. What we can do is repeat a button if we'd like to. Create a button that's going to
set the state to open and we'll set the button text to read open restaurant. Nice, so back to our
browser. The restaurant is currently open, that's the initial state as soon as I render. I then can
click on close restaurant, open restaurant and have that happen back and forth when I click on each
one of those buttons. Now let's take this a step further and actually replace this button. We're
going to try to make all of the same stuff happen, but we'll only use one button. We'll set the
initial state to true because our initial state was open before, then we're going to do this. So
we'll say if the status exists. So if status is true, we want to return open, otherwise we want to
return closed. When I go back, we refresh and we see that button isn't working at all because I need
to change the value that's going to be passed in here to set status, check it out. We'll say call
set status with whatever the opposite of status is. Then I can get fancy with my button, use another
JSX expression to say, all right, whenever the status is true, we want to close the restaurant,
otherwise we want to open it. So let's make sure I did that right. We'll hit refresh. The restaurant
is currently open, that does match our initial state. I can then click on close restaurant. The
restaurant is currently closed. Open restaurant, the restaurant is currently open, nice. So we've
created this little Boolean toggle here where our initial state is true. We call set status whenever
we want to make that change, and we're going to send into this function, whatever the opposite of
status is so that we can reset that state.

## Handling state with nested components

Right now we have our app component. This is managing state, and there are some child components
that this is rendering as well. So we have header and we have main. A best practice that is often
talked about in React is how we want as much as possible to lift state up in our application. And
what that means is we want it to be as high up in the tree as possible. As much as possible we want
to minimize the number of times we're calling useState in these smaller components because then it
can make it harder to track these changes and things get out of sync pretty quickly. So a
recommendation that I have for you is to create your state within, say, the app component or
whatever the root component is in your application. And then you can pass these values down as
properties to child components. So what we want to do here is quickly show a demo of this. So here
in our app we have the status. Now we can pass this status down to a child component using a
variable name. So let's go ahead and call this openStatus and we'll set that equal to status. So
whatever this value is is going to be passed down to the child, which is the main component. So once
I've done that, I can reference that on this main component. And we could say, here, we'll call this
openStatus. So here, let's just try to display openStatus, see what we get. Here in the app it says,
Welcome to this beautiful restaurant! I suppose that's not doing that much for us. So we'll say
"Open" : "Closed". So if this is true, return open. If it's not, return closed. So here we're seeing
open. Then when I close the restaurant, this is going to update to reflect that state change. So we
don't use useState for this. Instead we're just passing that down via props. So we're saying
openStatus is the prop, but it refers to that state variable. Now another thing that I want to show
you is how we might pass this state back up to the parent component. So let's go back to our main
component. We're going to say onStatus, and then here we're going to lock that into setStatus. Now
if we go to our main component and we say onStatus, and then we add an onClick to this button here,
we can call this function. So we'll say onClick equals onStatus, and then we need to pass in here
whatever we want that value to be. So let's just say true. So now if we go back to our app that's
running, the initial state is open. If I close the restaurant, it's closed. But if I click, and this
is from the child component, this is from the main component. This is going to call that function
and it's going to, from this event, change the state of the entire app. So this is how the
relationship between these two components will work. We create a state variable at the highest level
we can. This is going to keep track of whatever the status is, and then I pass that down as
properties. So we say openStatus and onStatus, and then these functions are then called inside of
the main component. And then it's going to actually change the status of the entire app simply by
calling them from props.

## Managing state with useReducer

Let's make some adjustments here to our App.jsx so that we can handle our state management in a
slightly different way. So here we're going to import useReducer from React, and we will use that
function. This is just another hook that's supplied to us by the React library. We're going to use
it here on line 60 within the App component. So I'm just going to comment out our status here, our
useState hook, and we're going to refactor it using useReducer. So a couple things to know about
whenever we call useReducer, it's going to return an array of two items again. This time the two
items are going to be that State variable, like you might expect, as well as some sort of a function
that's going to dispatch an event. So the useReducer function is taking in that reducer as the first
argument. All right, so we're going to take in a status, we're going to return whatever the opposite
of that status is. All right, so all of that being said, we can take this dispatch function and give
it a name that's a little more clear. In this case, we'll call it toggle. I'm going to pass the
toggle in to onStatus, so we'll pass that down the tree, to the main component, and then I don't
have setStatus anymore. Instead of having to pass this and making sure the right variable goes in
and all of that, all I can say here is toggle. So that's potentially a little less error prone
because you're asking people to just write the name of the function versus creating a whole bit of
logic there. All the logic is actually built into the reducer itself. So let's go back over to the
app. The initial state is working correctly. We see that as the page loads, our restaurant is open.
Now when I click Close Restaurant, it's going to close it, and everything still works from the
button down here as well. The button that is part of that child component is still honoring all of
the rules that have been set up with useReducer. So useReducer is a really useful function. It is
one that manages a more complex state, and I really think the benefit here is that we're able to
pass in just the name of the function without having to worry about the logic that is built here on
line 61.

## Working with the useEffect hook

Another important hook that's part of the React library is useEffect. useEffect can feel a little
tricky because it's typically used to manage side effects that aren't really related to a
component's render. So things like console messages, loading data, working with animations, these
often can benefit from useEffect. So let's take a basic look at this by first importing it. We'll
say useEffect. And then within the body of the component, in this case, we'll use useReducer for our
status, and then let's go ahead on line 64 and say useEffect and then useEffect should take in a
function. This function is going to return a console log. The console log is going to read the
status. How about the restaurant is status? Again, if true, return this. If false, return this.
Nice. So remember we're doing this within the template string. These are the backticks that you see
here. Now if I go over to my browser here, nothing seems to have changed too much, but the real
interesting part is the browser console. So I'm going to hit Command + Option + J. It'll say the
restaurant is open. And then when I click Close Restaurant, it's going to fire again. By default,
it's going to run whenever that state value changes, but you can be a little bit more granular with
that. So if you pass an empty array as the second argument to this, this is going to be called on
first render. So if I go back here and we refresh, it says the restaurant is open. Let me clear this
out so we can actually see it. I'll hit Refresh. The restaurant is open. If I click Close
Restaurant/Open Restaurant, everything is being affected on the page, but the side effect isn't
happening again. That's because we've passed in this empty array that says just call this function
when the app first renders. It's also possible to specifically ask for the function to be called if
certain values change. So if we say status and then we start closing and opening the restaurant
again, that function is going to fire every time that value changes. So this is one that is
important to know about. Again, useEffect will take in two things, a callback function as well as
what's called the dependency array. We can use it to define nothing, like this should only run once,
or to define that it should be called whenever certain variables, certain values change.

## Creating a project with Next.js

If we take a look at 05_01b, you might notice that it's now an empty folder. This is by design
because we're going to be creating a new project inside of this folder using Next.js. Next.js is one
of the most common React frameworks out there, and it's one that's recommended by the official React
team for creating your projects. So I want to show you this framework because within it is baked in
a lot of those best practices in the world of React. And using Next.js will allow us to take
advantage of it without having to do too much setup. More on that in a little bit, but let's first
create our project. We're going to go to 05_01b. So let's make sure that we're in the right
directory there and we're going to initialize a Next.js project. So let's go ahead and run npx
create-next-app@latest. Now this is going to walk me through a whole wizard of putting this
together. So what is your project name? We'll call it next-react-project. You then can decide
whether to use TypeScript. I'm going to choose No for now. ESLint? I'll say No for now. Let's go
ahead and incorporate Tailwind. We're also going to incorporate the source directory. We want to
yes, use the App Router. Wnd then we'll say No, we don't want to customize that default. So when
this is all being installed, it's going to generate a starter project for me. So from here, what I
can do is migrate to that project, next-react-project. So once I've done all of that installation,
I'm going to run npm run dev. This is going to stand up a Next.js project at localhost:3000 So I can
go back to the browser, go to localhost:3000, and this is going to pull this up. So now we have
Next.js in the browser, and this is sort of our starter for this project. Now, within this project
there is a source folder. Inside of that source folder is a folder called app. Now, all of our
files, by default in this Next.js project are going to live inside of the app router. And this is
going to allow us to create routes for different endpoints within our application. If we look at the
layout.js file, this is sort of the metadata of the project as well as the HTML. So just like we
injected our React code into the body of our V app, we're doing the same thing here. This time it's
being added under this children property. And the main kind of page, for lack of a better term, is
here underneath app page. So inside of this file, the page.js file, is the component that we're
going to render whenever we go to the slash route. In other words, whenever we go to the homepage at
localhost:3000. So let me go back here and we're going to remove some of this code here. We're going
to remove all of the div from this project so that all we have is a main project here. Let's go
ahead and add More React to Come. I'm going to also get rid of the image 'cause it looks like that
is not being used anymore. And then if I go back here, More React to Come. So we've created our
Next.js application. In the next video we're going to take a look at how we can set up routes and
start to work with this production-ready React app.

## Using the App Router

As I mentioned in the previous video, we're using Next.js, and specifically, we're using the App
Router. What this means is that within my project, my Next.js project, we have a folder called app
that's going to create all of the different routes that are part of our application. So, this is a
file-based routing structure that we can use so that anytime I create a folder, like mountain, this
is going to create a new route for me. Within that mountain folder, I can create a new file called
page.js, and the page.js file is going to display whatever content we want to see when I hit this
route. So, for example, I can create a new function called Page, and then here we can return a main
component. Inside the main component, we'll add an h1, Lift Status Info. So, within the next React
project, we're going to run npm run dev, go to localhost:3000, and we see our homepage here, but
we're here really to look for mountain, right? And this is going to show our Lift Status Info. Very
cool. So, this is our first example of creating a new route. A couple things to know about this.
Sometimes your app folder will be inside of the source directory, sometimes it won't. That's just an
option that you can choose as you're configuring your project. The app folder though, will wrap
around this mountain folder and any other routes that we want to create, so, within this page,
that's where we'll make that happen. Another thing that we can do, if we go to the layout file, you
can add any sort of component that you'd like to right here in the layout if you want it to appear
on multiple pages. So, let me show you what I mean here. We'll say return a header, inside of that
header, we want a div. Inside of that, we want a nav element. So, within this div, this is where
we're going to use our first component from Next.js, and this component is called Link. Nav is going
to help us navigate between different components and different routes. So, let's go ahead and add an
href to this. So, we'll say href/ and then let's actually make this a full link so that it has some
text, so we'll open and close the link component, and then we'll say Snowtooth Mountain. Now, from
here, what I want to do is, we'll create another div. This other div, we'll say mountain, and we'll
call this Mountain Info. So, now once I've created the header, I can place it inside of the
RootLayout. So, this'll prevent me from having to import this in absolutely every page, and remember
where it goes every time. Instead, we'll just use the header on every single page. So, with that
being added, I also need to remember to import this Link. So, Link is going to come from next/link.
Okay, back over to LocalHost:3000. Now, if we go to just the home route, we have Snowtooth Mountain,
Mountain Info. If I click on Mountain Info, that will take me to this, and then if I click back to
Snow Tooth Mountain, it'll take me to the homepage. So, the link component works harmoniously with
the App Router. Anytime we add a link there, it's going to keep track of where we are and help me
move from page to page with ease. In a later video, we'll make some adjustments to the style so that
looks a little bit better, but for now, we should understand. Use the link whenever you need to
navigate between pages, and then place any components that you want to render on every single page
inside of the RootLayout.

## Building a server component

Within our React project here, within our next JS React project, to be specific, we have our app
directory that's going to contain all of the files that we're going to render in the app. So what's
kind of hiding in plain sight here? So if I run this app on localhost:3000, I can see that these
routes are being created for mountain, for our home route, and then we have links in between them.
Anytime I create a component using Next.js, by default, this is a server component. So think of a
server component as a component that allows you to write some UI that can be rendered and optionally
cached on the server. So what we can see here, if we open up our developer tools, so we'll go to the
Elements tab and then we'll start to investigate some of these tags that are in the head. You're
going to see a lot of these chunks, these pieces of the app that are being imported also at the end
of the body because they want to make this as efficient as possible. We want to render something and
then load our JavaScript in the background. All of this JavaScript is being rendered as this string.
So this is considering performance every step of the way. And we're also getting some benefits like
security and caching and all sorts of things are happening behind the scenes that we as developers
don't have to worry about too much, but we can understand that it's going to make things much
faster. So we can automatically implement server rendering without configuring anything. All we need
to do is create these folders and create these components that live inside of them. Something to
note about creating these pages or these components is that each one of the pages, each one of the
files has to have a default export. So this is why we're export default function Page, that is how
we're creating this component so that it can be properly consumed. So basically anything that's on
the page is this interactive preview of HTML, and then anything that's interactive can be hydrated
with that JavaScript. So let's go ahead and make a couple adjustments here. So we're going to create
another directory called hotels. Inside of the hotels directory we'll create a page.js file. The
page.js file, again, export default function Page, and then we want something to be displayed here.
So let's go ahead and return a main element and then we'll return a div that says Hotel Details.
Nice, so this is, as a preview, going to be a container for some data that we're going to load in a
future video, but we're rendering a server component here by default with our Hotel Details. I don't
love how that menu looks and I promised that we would change it. So let's go ahead and do that now.
We're going to within our header component that's inside of the layout, make some adjustments here.
Configured in our app router project, we have selected Tailwind as the CSS tool for this app, and
Tailwind works with different classes. So let's go ahead and add a few things. We're going to say
bg-slate-500. P-Y stands for padding on the Y-axis for about 16 pixels of padding. Let's add a
little bit more markup here to our div. We'll say class="container mx-auto px-4". Same amount of
padding, but on the X-axis. Then we'll add class="flex items-center justify-between". So this is
going to help us incorporate Tailwind's Flexbox to order these elements about the page. Here we'll
say className="flex items-center" again. Let's see how we're looking. All right, cool, so now we
have these elements here. That's looking pretty good. Mountain Info and Snowtooth Mountain, all of
those are appearing. So in a subsequent video we'll work on the look and feel of the lift status
info, but we'll have to load some data first. But for now, we see these. You can always add
additional links if you want to. Let's link to our hotel, for example. As soon as I add that link
component, it will be added here and then I can click on it and that'll take me to that page. All
right, so we have React server components supported. All of that is working in the background to
make these apps, make these components as efficient as possible so that our app can be super fast.

## Challenge: Creating a route

(upbeat music) - [Instructor] I have a little challenge here for you. Within the next React project,
there is a README file with some instructions for a challenge. I'd like you to create a new route at
/contact. Then I'd like you to render a page component at /contact. Then you can run the app and
make sure that that is rendering appropriately if you go to that route. And then for extra credit,
which, of course, counts towards your final React grade, add a link to the contact route from the
header. Okay, that's it. In the next video, I will go over the solution. If you get stuck at any
part in that, the solution will tell you how to fix it.

## Solution: Creating a route

(upbeat rhythmic music) - [Instructor] Alright, let's go over our solution. So we wanted to create a
new route, we wanted to render a page component, and we wanted to make sure that the link is to the
contact route from the header. So anytime I want to create a route, I go to the app folder, I create
a new folder called "Contact." Within it, we have a default export typically in a file called
"page." (keys clicking) And we export a component. In React, we want to always make sure that our
component name is capitalized. That is a React thing, not a Next.js thing necessarily. Let's go
ahead and return a main element, and we'll say, "Contact us!" Great. So this should be exported from
that route. If I go to the correct folder, I make sure all of my node modules are installed, and I
run "npm run dev." This should run. We go back to localhost:3000, it looks like it's on the hotels
route, that's okay. But if I go to contact, we should see "Contact Us!" Nice. What happens if I go
to a route that doesn't exist? Well, I could see like, "contact-us," that's not a route I've
created, that's not a folder. It's just going to 404 out. We can always customize that page of
course, if we'd like to. I also added the extra credit, right? So all these errors are because I
added that contact-us 404 route. So let's go ahead and go back to the header, which lives in the
layout.js file. What I can do here is just copy and paste one of these divs, add the appropriate
href to contact. We can change the title to "Contact Us." And then when I go back here, we see
Mountain Info, Hotels, Contact Us, I click on it. And there we go, how cool is that? So that is how
I add that link, that is how I create the page. And we can always add additional routes as our
content demands. So awesome job, everybody. We've done it, we have created even more server
components inside of our app.

## Fetching data in React applications

Whenever possible in a Next.js application, the best way to fetch data on the server is in a server
component. So what this is going to mean for us is that we're going to fetch and render data in the
same location. So what we can do here is we can go to our Mountain file. So the page file inside of
Mountain is going to be the place where we can write an asynchronous function to get some data into
the app. We're going to grab the response from await fetch. So fetch here, you might've seen fetch
in other contexts. You might have seen it in the browser. You might have seen it in Node. There's
all these different types of fetches that you can do to ask a REST API for some data and to get it
back. So just for the sake of clarity here, I want to say that this fetch is a Next.js wrapper
around fetch, but it does much the same thing. All I need to do here is pass in the API endpoint,
which in this case is snowtooth-api-rest.fly.dev. So that's the location of this REST API. So what
I'll do is return response.json and that will take whatever the response is here and turn it into a
JSON string. So from here what I can do is fetch this data directly in the page. So we'll say const
data = await getData. Then what I can do is make sure that my function here is asynchronous. So we
could make the entire component that way, and then we can use that data somewhere within our
component. So let's just add a div here. We're going to open and close a set of curly braces, and
say JSON.stringify(data). So back to our browser, we should see, okay, we have all of this lift
status information coming back, and that is good. We can see it. So let's go back here. We're going
to take our div, and we're going to turn that into a table. Within the table, we're going to add a
header, thead. Inside of this, we're going to add a row, and then inside of the row, we're going to
add a th element that's going to be a table header. It'll have a little bit of bold style here, lift
name, and current status. Okay, from here we'll add a table body, and the tbody, we should say
data.data.allLifts.map. And here we're going to do the following. We'll say lift. This should return
a table row with a key that says lift.id, and then we'll use this table data cell lift.name, and
lift, td meaning a table cell lift.status. Okay, let's see how we're doing. So it looks like we're
having trouble reading all undefined. So let's get rid of all lifts. Make sure we're parsing the
data appropriately. A good way to really figure out what's going on here is we can say, let's go
back to where we were before. JSON.stringify(data). Okay, so everything looks like it's at this main
level, so I may not need all of that, so back to the drawing board. Let's get rid of those. We'll
say data.map. Okay, there we go. Feeling good. Half the battle is making sure we're looking at the
right key. So we've created this table. A server component is the best place to fetch data, because
fetching data on the server will likely yield speedier results than fetching it on the client. So
all of this can be handled by co-locating our data fetch with our component.

## Passing data as props

We've gotten to see how all of our components by default in an XJS application are server
components. This means that they're going to be server rendered as HTML, and then anything that is
going to be interactive, we need to handle that in a slightly different way. So we're going to make
some adjustments here specifically to our hotels component. And this component is going to start to
introduce us to some of these different concepts. Now, the first thing I want to do here is I want
to create an asynchronous function. This time we need to load some data for this page. So we're
going to grab it like we did before. So we'll say const res equals await fetch. So our response is
going to fetch from https, snowtooth-hotel-api.fly.dev. Okay, and once we do that, we can return
response.json. So make the request and return it as JSON. Just like we did before, we are to use a
$100,000 word, co-locating our request with our component. So we are going to grab it here, getData.
And then we're going to, within our component, we have a squiggly red underline, so let's fix that.
We need to make that an asynchronous function in order to use a weight within that context. So once
we do that, we can make use of some of this data. This burned me last time, so I'm going to learn
from my mistake, and I'm going to say JSON.stringify. And we'll just stringify that data. So this
will allow me to take a look at it and know what I want to parse out of that data inside of that
response. So let's make sure that we are in the right context here. We're going to look in
06_02b/next-react-project. Then we're going to run npn run dev. We'll take a look at the hotels. And
this is going to show me all of this data. So, so far so good. We're seeing the data in context. Now
we need to create some components based on that. So let's create a function called HotelBlock. And
HotelBlock is going to be this dynamic component that we're going to render here within our page. So
we're still want to keep the div, but we're going to place something in there that's a little bit
more dynamic. So let's take a look at what that might look like. Our HotelBlock, we're going to say
return, a div, and then inside of that div, we'll use an h2 that displays the name. Now, the name is
going to be a property that we pass down from the data. So let's make sure to destructure that.
Think of this as being kind of like a presentation component. It's just presenting some UI. And the
page is actually going to be responsible for passing that data down. So we know how to pass data
down to a child component, right? We need to pass it down as props. So if we take another look at
our HotelBlock, remember, we're going to render this inside of this div so we can map over the data
that's returned. And for each hotel, we're going to use HotelBlock. And HotelBlock is going to give
us the values that we need. So we'll say key equals hotel.id. Then we're also going to need the
name. So name equals hotel.name. Nice. So now if we go back to our hotels, we see these little
components being rendered. We're just grabbing the name for each. And if we wanted to make this more
complex, we could. There's always a way to make it more complex, right? So inside of the HotelBlock,
we're going to parse another one of those variables, parse another one of these properties. So we'll
say capacity. Then I can also grab capacity from properties, and of course, I need to pass it down
here, capacity equals hotel.capacity. All right, so that's starting to show up. Maybe I should use
JSX and not just type that in. There we go. So now we see the capacity for that hotel. Pretty cool.
So again, we can parse out any of those values. One other thing that I want to fix that is upset
with us is this property class thing. So that's just saying that I'm trying to use, and I did this
when I was trying to add those tailwind classes, I used class like HTML, but we really should be
using className. And you'll notice that I did it two different ways. So that's obviously not going
to fly. It's a warning, not an error, but it still is nice to clean up. Okay, so we cleaned that up.
We created our HotelBlock. We're passing down data to a child component. In the next video, we'll
take a look at how we can display some client details using a client component.

## Creating a client component

Currently, we are rendering a few different components that are loading dynamic data from that REST
API. So within this component, we fetch some data, we populate it into the page, and then that
passes data down to each one of these HotelBlocks. Now something that is an important thing to know
about working with Next.js is the concept of client components. We know that we fetch data most
often in a server component, and that all of our components are by default server components when
we're working in a Next application. But sometimes when we have interactivity when we're working
with browser APIs or we're trying to reach out to some local files, we might need to use client
rendering in order for these sites to work. Let me show you an example. Here in our HotelBlock, what
would happen if we wanted to use an image? We have here inside of the public folder a few different
images for these different hotels. Each one of these has an id.jpeg, so we can parse that from the
data. So we want to create these image URLs so that we can load them from the public folder. So a
couple changes that I need to make here. We're going to first add an id here to our name, and then
we're going to import Image from "next/image"; Then inside of the HotelBlock, we're going to add
this image component. We'll specify the source here, so id, and then we'll add a pixel width and a
pixel height. Okay, so from here, what we can do is we're going to create a loader function that's
going to create the URL for wherever these images live based on their ID. So we can create here on
line 11 an imageLoader. That's going to take a look at whatever the source is, the source value, and
then we'll return './hotels/${src}.jpeg' J-P-E-G. So this is going to create that URL for us. What
we can do with the image component with Next.js is we can say, hey, I actually want to use a loader
called imageLoader to load that file. Then what I can do is I can scroll down a little bit and make
sure that my page component has an id. Great. So now this can be consumed in the HotelBlock. Let's
see how we're doing. So over here it says, functions cannot be passed to client components, blah,
blah, blah, and what this error actually means is that we need to make an adjustment so that our
HotelBlock is a client component. Our HotelBlock is trying to look in this folder, but because it's
being server rendered, it doesn't know how to do that. So we need to create inside of hotels a new
file. We're going to call it HotelBlock.js, and we're going to take any of the code that makes this
work and add it to this file. We also need to grab the image from next/image. We want to export
default this function so that the HotelBlock can be consumed over here inside of this page. So we'll
say import HotelBlock from "./HotelBlock"; Okay, so we can also delete that image because we're not
using it there. We're using it in the different file. Okay, so we're still getting this error, and
that's because I need to take one final step. I alluded to this before. At the very top of this
file, we're going to say "use client"; use client will always use if we're adding interactivity,
button clicks, if we are trying to look in this public directory, if we are trying to work with any
browser APIs like local storage or geolocation, things like that. So if I go back now, we should see
all of these images are loading. So this is our first example of creating a client component in
Next.js. We know it's a client component because we're trying to deal with some local files, and we
say that it's a client component with use client.

## Building a form

Currently we have this Contact Us page that's really just an H1 that's being displayed when I go to
the /contact route. So we want to create a contact form. Typically, the best way to get started
creating that form is to create the containers where the data will go, and then we'll wire
everything up so that it works appropriately. So within this main element, we're going to add a
form. Within that, we're going to add a div that has a label. The label will be right next to an
input. We're going to create another div that has a label again, and then it will also have a text
area. There are a few attributes that we want to add. So we're going to say HTML for email. Then
we're going to, in the input, create an ID of email, a type of email, a name email, and then we'll
say that it's required. From here, we can add a label for message. So the form is just going to be
an email address and some sort of a contact message. This one is going to have another HTML for
attribute, HTML for message. Our text area will say ID message required name. And then there's a
property called rows that we can set to say basically, how high do we want this text area to be. In
this case, we want it to be four rows. So outside of the div, but inside of the form, let's add a
button. What would a form be without a button? Let's add some text to that button. So we'll say send
message. Then inside of the button opening tag, we're going to say type submit. And then we'll give
that a save. So over here we're starting to see this thing take shape. It looks like I forgot the
label for the email. So let's add that here on line seven, and then we should see all of that there.
So that's starting to take somewhat of a shape here. Let's add a few tailwind classes to make this
look a little bit nicer. What I'll do inside of the main is let's say class name, Max-w-md. And then
we'll add some padding on all sides. We'll give it a background of white, a shadow of md, rounded
md, so that centers it, that's starting to come together. Let's add a style for our H1. Remember, we
always use class name instead of class and react unless we forget and then we have to go back and do
it. But still, we should be doing that from the start. We'll say font bold, we'll center the text,
and then we'll add MB-6, which will set the margin on the bottom. Okay, starting to take shape here.
We probably want to add a few styles to the form elements as well. So let's, inside of the form add
a class name space Y-4. Okay. And then we will also on the label. So here we want to set this as
block text, small font, medium text, gray 700. And we'll copy this class for the other label as
well. Okay. Then let's go ahead and add to our input. So the input is here. Line 20 will say class
name. Let's just say focus ring two. So that'll add a little bit of style to that. We'll also say
focus ring blue 500. And then we'll say focus border blue 500. And let's add a style for just what
it looks like regularly, even when it's not clicked on. We'll say border gray 300. Okay, and then
we'll add that border attribute so we see it in both places. Now I can just copy this entire class
name to our other input for the sake of making it easy on ourselves. So that's that. And then for
good measure, let's add a little bit of style to our button, shall we? Let's add, we'll say class
name and then we'll set that equal to text white, background blue, 600, rounded md. Okay, that looks
pretty cool. And let's add some padding to it just so that there's a little bit of space around that
button. There we go. Nice. So that's looking good. This is another benefit of working with Next.js.
We can configure it to work with Tailwind so that we can just jump in and we can start to add those
classes to our elements. Know that working with Tailwind is totally optional. You don't have to do
it, but just know that this is available to you if you do choose to configure your project with it.

## Creating a server action

We've created a little container for our form so we have an email and a message. Now we need to
handle these form elements when they are submitted. So I want to show you how you can work with
server actions. Server actions are functions that can be executed on the server. These can be called
from server components or client components to do things like handling form submissions and other
types of data mutations. So we're going to put all of this directly into the same file so that we
can see how everything works together. The first thing I'll do is here in contact page.js, we're
going to create an asynchronous function called submitForm. You can call this function whatever you
want to call it, something descriptive. Obviously the more descriptive the better. Then we're going
to use formData. So whenever I'm dealing with form elements, our browser is going to process those
form elements so we can just work with those elements natively instead of having to create different
state variables for each one of these fields or for the entire form. So we're going to use a
directive called useServer here. UseServer is going to mark that this is a server side function that
can be called from client side code. So this is some client side code here. We're calling this
function. Then we're going to adjust this. So we need to process these form fields. So let's create
a variable called formFields, and we'll set the email to be formData.get ("email") and then we'll
say message: formdata.get ("message"), so processing both of our form elements here. Then we're
going to console log this. So we'll add a couple console logs, so we'll say formFields. So here,
then we'll say console.log TODO: Send these form field values to a backend. So depending on which
backend you're using, the rules will be slightly different, but this is the moment that you would do
that. And then we'll just return these form fields on line 12. The way that we're going to wire up
this function to our form is to scroll down to where we start our form. We're going to add an action
to the form, so when it's submitted, we want to call this function. So we need to grab submitForm,
pass it in here, and we'll click save. Alright, so couple things here. I'll plug in my email address
and I'll say, have we successfully built a server action? And then we're going to click send. So
sometimes when we're dealing with Next.js code, a quick hint here, we will look in the browser
console for our contact us message, and we don't see it. So the place we actually want to look for
this, because this is a server action. This terminal is going to show us, because our application is
running here, it's not a client message, it's a server message. So this is telling me that my form
fields are returned, and now I can go through the process of building our form field values into our
backend. So regardless of where you want to send this, if it's an API, if it's some sort of database
that you want to send it to, all of that would be handled right here. Alright, so we've done it.
We've created a server action here. It's called submitForm, and we've wired it up to the form
element by adding it as an attribute. So whatever types of mutations or form submissions that you
want to create, if you're working with Next.js in React, it's a good idea to take advantage of these
server actions.

## Designing performant apps with React Server Components

Well, we did it. We built a couple different React applications first using Vites and second using
Next.js to introduce the fundamentals of the React library. Now from here, there are a couple
different directions that you can go in, depending on what you most enjoy and what your work
requirements are. So I would definitely check out more on Next.js data storage options. So this
would probably involve another tool on top of Next.js, but learning to store data in databases, REST
APIs, even GraphQL APIs, all are different next steps that you can take as you're starting to build
out your apps. I would also recommend learning more about React server components. There are a lot
of cool things coming out in this world of React, not necessarily just for Next.js, but all over the
place for really thinking about how to make our applications as performant as possible. And speaking
of that, we can talk about Next.js caching in more detail as well. The Next.js documentation is a
great place to keep your eye out for all of these things. If you're looking for guides on testing,
authentication, deployment, this can all be found here in the Next.js documentation. I want to thank
you so much for being part of this journey with me. The world of React has changed a ton since it
was introduced in 2013, but the future is very bright for continuing to build all of our
applications using the library and using all of the tools that are part of the ecosystem.

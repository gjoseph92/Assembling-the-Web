<style>
@import url(http://fonts.googleapis.com/css?family=PT+Sans:400,400italic);
@import url(http://fonts.googleapis.com/css?family=PT+Serif:400,400italic);
@import url(http://fonts.googleapis.com/css?family=PT+Mono);
body {
    margin-left: 12%;
    width: 75%;
    color: rgb(76,76,76);
    font-family: 'PT Serif', serif;
}
h1, h2, h3, h4 {
    color: rgb(46,46,46);
    font-family: 'PT Sans', sans-serif;
    margin-left: -7%;
}
h1, h2, h3, h4:after {
    content: 
}
h1 {
    font-size: 5em;
}
h2 {
    font-size: 2.8em;
}
code {
    font-family: 'PT Mono', monospace;
}
body > p {
    font-size: 1.2em;
    line-height: 165%;
}
img {
    max-width: 100%;
}
a {
    text-decoration: none;
    border-bottom: 1px dotted rgb(112, 40, 40);
    color: rgb(112, 40, 40);
    transition: color 0.15s;
}
a:hover {
    text-decoration: none;
    border-bottom: 1px dotted rgb(182, 65, 65);
    color: rgb(182, 65, 65);
}
/* Don't link-style images */
a[href$=jpg], a[href$=jpeg], a[href$=jpe], a[href$=png], a[href$=gif] {
    text-decoration: none;
    border: 0 none;
}

.caption > p {
    font-size: 0.8em;
    line-height: 130%;
    font-style: italic;
    text-align: center;
}
.caption.rfloat {
    float: right;
    max-width: 60%;
    margin-left: 15px;
}
</style>
<script>
function captionImgsWithAlt() {
    var imgs = document.getElementsByTagName('img');
    for (var i = 0; i < imgs.length; i++) {
        var img = imgs[i];
        var wrap = document.createElement("div");
        wrap.setAttribute('class', img.getAttribute('class') + ' caption');
        img.setAttribute('class', '');
        img.parentNode.insertBefore(wrap, img);

        var link = document.createElement("a");
        link.setAttribute('href', img.getAttribute('src'));
        link.setAttribute('target', '_blank');
        link.appendChild(img);

        wrap.appendChild(link);

        var alt = img.getAttribute('alt');
        var caption = document.createElement('p');
        caption.innerHTML = alt;
        wrap.appendChild(caption);
        console.log(wrap);
    }
}
window.onload = captionImgsWithAlt;
</script>

# Assembling the Web
**[Gabe Joseph](mailto:gabriel.joseph@tufts.edu)** // Dec 18, 2013 // Tufts University: [COMP150 IDS](http://www.cs.tufts.edu/comp/150IDS/)

-------------------------------------

No disks, no downloads, no installation: just a full 3D game engine running smoothly. Inside a browser. In four days, Mozilla and Epic engineers compiled the [full Unreal Game Engine](https://blog.mozilla.org/blog/2013/03/27/mozilla-is-unlocking-the-power-of-the-web-as-a-platform-for-gaming/) to a highly-optimized subset of JavaScript, playable with near-native performance. As this demo swept the Internet, it solidified excitement in an emerging idea about the future of the Web: ["Big web app? Compile it!"](http://kripken.github.io/mloc_emscripten_talk/#/) What if existing code in any language could compile down to JavaScript, the Web's universal language&mdash;and by doing so, run faster than pure JavaScript itself?

Generating JavaScript from other languages is [nothing new](https://github.com/jashkenas/coffee-script/wiki/List-of-languages-that-compile-to-JS). Some new languages just add syntactic sugar to plain JavaScript (like [CoffeeScript](http://coffeescript.org/)), or the high-level structure of existing languages is translated into JavaScript's corresponding syntax (as for Python with [Pyjamas](http://pyjs.org/)). Only now is true compilation beginning, with projects like Mozilla's [Emscripten](http://emscripten.org/) (the magic behind the Unreal Engine port) *compiling* C, C++, and more not to the low-level instructions of machine code, but JavaScript that emulates its behavior.

## Emscripten Description
<img alt="The modular LLVM compiler." class="rfloat" src="http://www.aosabook.org/images/llvm/RetargetableCompiler.png"></img>
Emscripten exists to make JavaScript a compiler target, much like x86, ARM, or any other architecture. It is based around [LLVM](http://llvm.org/), a compiler&mdash;or rather, the modular core of one&mdash;that connects to different front-ends and back-ends to parse a variety of languages, perform common optimizations, and output executables to different machine languages, one of which is now JavaScript that acts out similar operations to what the machine code would. Native code manages its own memory, so the assembled JavaScript just allocates a giant array to hold a stack and heap and stores most data there, using indicies as pointers. By pre-computing memory offsets into typed structures instead of using costly JavaScript objects, reducing garbage collection, and benefiting from decades of work on optimization strategies for static compilation, this code circumvents the typical slowdowns of dynamic JavaScript.

Now nearly any existing program can be turned into JavaScript, but Mozilla realized performance would be even better if the JavaScript engine could optimize for this specific style of code. Thus [asm.js](http://asmjs.org/) was created, standardizing this subset of JavaScript that only allows numbers and uses explicit typing, and letting the interpreter pre-compile ruthlessly efficient code that wastes no time type-checking or needlessly garbage-collecting. By this point, the code looks nothing like typical JavaScript, decorated with type casts (`|0` for `int`, `+` for `double`), strange numeric constants, bitshifts, and array accesses, but devoid of the quintessential dynamic objects or strings. It's hardly human-readable, but not intended to be: JavaScript has indeed become the [assembly language of the Web](http://www.hanselman.com/blog/JavaScriptIsAssemblyLanguageForTheWebSematicMarkupIsDeadCleanVsMachinecodedHTML.aspx).

<img alt="Unreal Engine, asm.js edition." src="http://acko.net/files/asmjs/code.jpg"></img>

<img alt="Emscripten + asm.js performance." class="rfloat" src="http://kripken.github.io/mloc_emscripten_talk/macro4b.png"></img>
By the numbers, the assembled JavaScript future is bright&mdash;blazingly so. In browsers that know about the asm.js pattern, the performance is staggering: just 1-2x slower than native code. This gives the potential for many existing programs to run in the browser (like Unreal Engine has), not as hacks or demos, but fully usable applications, right now.

<img alt='Box2D physics engine, browser-ified many different ways. (Axis is time slower than native C.) The Chrome 30 and Firefox 22 entries are making use of optimization for asm.js, whereas Chrome 27 and IE 10 use the same Emscripten-produced code without explicitly optimizing for it. Note both are still faster than the "typically" ported Box2dWeb' class="rfloat" src="http://kripken.github.io/mloc_emscripten_talk/box2d_graph_updated2.png"></img>
Even more exciting are browsers that *don't* know about asm.js&mdash;they too see performance gains with no internal modification at all, just the benefits of assembly by Emscripten.[^1] That is the beauty of assembling to JavaScript: already a standardized, universal language, assembled code can run in any existing (modern) browser on any machine, and quite quickly.

This (theoretically) opens all the benefits of being "on the Web" to a world of existing code: cross-platform support, zero installation, automatic updates, and greater convenience for users. Vendors embracing this model get the economic advantages of Web applications, such as easier protection of intellectual property or subscription-based pricing, while reducing hosting costs, as more work can be done by the high-performance client instead of the backend. Users can enjoy the same familiar experience, but with the security of their data backed by the cloud. Most of all, the Web can benefit from an ever-increasing number of resources to connect with each other.

With a fast, universal, machine-independent assembly language, it seems there's nothing not to like. 

## Taking the "JavaScript" out of JavaScript
By [Atwood's Law](http://www.codinghorror.com/blog/2007/07/the-principle-of-least-power.html), "any application that *can* be written in JavaScript, *will* eventually be written in JavaScript." With Emscripten and assembled JavaScript, the set of "can" and "will" have just exploded&mdash;but calling the results "written in JavaScript" is a stretch.

Because of its structure, [fundamental parts of program flow](https://web.cs.umass.edu/publication/docs/2013/UM-CS-2013-026.pdf) can't be expressed in JavaScript. The language is inherently single-threaded, with blocking, asynchronous-only events. Combined, these constructs make it impossible to simulate multi-threaded or synchronous behavior, which is an essential part of the design of many programs. The [ECMA standard](http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf) states, "ECMAScript as defined here is not intended to be computationally self-sufficient"&mdash;yet the languages being assembled to it are. A truly arbitrary solution requires executing this assembled JavaScript in a quasi-operating-system made in JavaScript that recreates a scheduler and call stack (as created by [Vilk et al](https://web.cs.umass.edu/publication/docs/2013/UM-CS-2013-026.pdf)). This is starting to sound much like [IP over HTTP](http://www.isi.edu/in-notes/rfc3093.txt): it's not reinventing the wheel, it's building the wheel out of lots of small wheels.

Linux already can [run in the browser](http://bellard.org/jslinux/). [`llvm.js`](http://kripken.github.io/llvm.js/demo.html) can compile itself in the browser, compiling C++ (or any other language) to JavaScript by executing JavaScript that was assembled from C++. Why not just compile all of Chrome to JavaScript and run the [browser in the browser](http://badassjs.com/post/20294238453/webkit-js-yes-it-has-finally-happened-browser)? With [Kelly Johnson's principle](http://en.wikipedia.org/wiki/KISS_principle) in mind, cramming an already-functional program on top of a tower of abstractions not designed to support it is not keeping it simple stupid, especially when the only simplicity gain is from not having to download and save an executable in order to run it. Where is line drawn for things that have no place running in the browser?

Though impressive, these assembled JavaScript demos are not a mark of victory for JavaScript. Just the opposite: they exist only by using as little of JavaScript as possible. The whole premise of assembled JavaScript is based on taking advantage of abstraction leaks from the interpreters: 64-bit double handling, optimized [special treatment](http://wingolog.org/archives/2011/05/18/value-representation-in-javascript-implementations) for number types, and JavaScript VMs' detection of code that acts statically typed, among other things. The point is to optimize for how the interpreters optimize for the machine, not the machine itself. asm.js is an elegant acknowledgment of the fact that a full JavaScript engine is overkill when it will just be acting as a glorified virtual machine. Yet the concerns of assembled versus typical dynamic JavaScript seem far too separated to be named by the same language. After all, there is a reason C and assembly are different entities. The purpose of a scripting language, as defined by the [ECMA standard](http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf), is to "manipulate, customize, and automate the facilities of an existing system"&mdash;not to recreate them. Doing otherwise feels like building the antithesis of JavaScript into JavaScript.

<img alt="No objects here: memory to an assembled JavaScript program." class="rfloat" src="http://mbebenita.github.io/LLJS/images/arrays.png"></img>
Indeed, assembled JavaScript deliberately avoids the core features for which the language was created. JavaScript is object-oriented, interpreted, high-level, and garbage-collected. Assembled JavaScript ignores this for weakly-typed C-structs it re-implements in its virtual "heap", which it manages itself. Thus, even though the language was created for the purpose of interacting with elements in a Web page, assembled JavaScript currently has no interaction with the DOM, because the source languages have no idea they are running in a browser. The whole optimization of assembled JavaScript would break if they did: the DOM API exposes dynamically-typed objects, while assembled JavaScript gets much of its speed boost from static typing and memory-offset-based structures. An `HTMLElement` object can't be stored on the `UInt8Array` "heap".[^3] Therefore, without being able to access the Web browser's core features, it's hard for assembled JavaScript to be a part of the Web.

## The browser &ne; the web
Clearly, it's possible for JavaScript, and the browser environment in which it runs, to be the compilation target for a desktop application. That was evident the moment a 3D game engine flickered to life inside a program originally designed for parsing structured text documents. However, the Web is a larger idea than the user agent accessing it: its value lies in the accessibility and interaction of all its subcomponents. For making new classes of applications accessible through the Web, assembled JavaScript is a boon; for making them interact with each other, it deliberately ties their hands.

Linkability is already a [well-discussed concern](http://www.w3.org/2001/tag/doc/IdentifyingApplicationState) in web applications. It's easy for current web applications to become walled gardens when they don't need a URI scheme for the server to render them a new view of each resource. However, conventions and APIs (such as hash-refs and `history.pushState()`) are developing so that in traditional web applications, ["sharable, reproducible states can be identified by URIs"](http://www.w3.org/2001/tag/doc/IdentifyingApplicationState#UseURIsforStates). These depend on accessing the browser's APIs, though, which assembled JavaScript cannot do. The bigger challenge is structural: what does a link to an application's state mean? When a resource can only be accessed through the application (say a file in a [Linux operating system](http://bellard.org/jslinux/) running in a browser), an HTTP `GET` of it no longer returns a representation of the resource, but a representation of the (highly obfuscated) process needed to generate the representation of the resource.

Such linking challenges, plus the fact that most assembled JavaScript applications will not generate DOMs, makes them a pain for search engines to index. Searchability is likely the most valuable feature of the Web, and making a native program runnable on the Web is hardly helpful if nobody can find it. As a side effect of conforming its resources to broad but universal formats, the Web has made them universally searchable. Yet if resources created by a DOM-less application are only linkable through its own interface, returned as a blob of JavaScript code, search engines won't know how to interpret them.

Neither, in fact, will people. A huge factor in the Web's growth was its openness: just click 'View Source' to learn from how somebody created that page. With assembled JavaScript, nobody created the page: like other assembly, it's only supposed to be machine-readable. The "source" shown is not actually the source that created the application. (Minification already has this problem, but assembled code takes obfuscation to a new level.) So where is that original source, and how was it transformed? In this case, the LLVM compiler&mdash;which as [Steven Wittens mentions](http://acko.net/blog/on-asmjs/), has no relationship to Web architecture. ["Following your nose"](http://www.ietf.org/rfc/rfc3986.txt) can't explain how and why it works, or give any standards for it&mdash;because it is not a new component, but a very elegant way of hacking an existing one. What this will mean in the long term is unknown, but by making learning seem less accessible, it can only widen the producer-consumer gap between developers and users that the Web had, at one point, done such a good job of equalizing.

## Flashback
Whether it becomes the future of the Web or the catalyst for a different system, the ripples from assembled JavaScript's big splash will not fade quickly. With asm.js in particular, it is redefining what the language is expected to do, for better or worse. Evolution is a constant presence in the Web, though, so this is hardly the first debate between new interaction and current integration.

<img alt="A model website implementing Flash's stringent human interface guidelines." class="rfloat" src="http://i1-win.softpedia-static.com/screenshots/Easy-Templates-Flash-Website-Kit_2.png"></img>
Without offense against the developers of Emscripten and asm.js, there are some surprising parallels between these projects and Flash's origins a decade ago. Flash was created to enable interactions HTML couldn't provide, like animation, higher-performance graphics, and complex games. (A familiar-sounding billing.) This led to an uncountable number of sites using Flash, whether they needed it or not. However, this [came at the cost](http://antezeta.com/news/flash-problems) of less natural integration with the Web in which Flash lived. Breaking the separation of model and view, Flash content was for a long time unsearchable, linkability needed to be programmed manually, back buttons were often meaningless, and screen readers may as well have not existed. Picture the quintessential example: a photography gallery with 10-second-long animated musical intros and overzealous animations for every click, all while HTML offered a highly usable `<img>` tag.

Thanks to HTML5 and JavaScript, Flash is now becoming less and less prevalent (to [few complaints](http://isflashdeadyet.com/)). Yet it hints at an important addendum to the [Rule of Least Power](http://www.w3.org/2001/tag/doc/leastPower.html): "Use the least powerful language suitable for expressing information, constraints or programs on the World Wide Web," because people are guaranteed to use the most powerful language they can get their hands on, *regardless of whether they really need it*. The consequences of this today are aged-looking websites with mostly-unlinkable content, or at worst, an empty page with a "plugin not installed" alert. At least this is contained to a plugin. In building a next generation of web applications on the current nuances of JavaScript interpreters, future VMs may become tied to these esoteric abstraction leaks and be unable to change for risk of losing backwards compatibility, and therefore market share.

Flash was a step in the right direction, just with the wrong foot. Assembled JavaScript seems to be no different.

## What's next?
The biggest problem is not even with JavaScript, but making whatever source language it compiles from an equally first-class citizen of the Web. This is most apparent when an entire application is assembled to JavaScript: a program that never had any connection to the Web can't be expected to grow one just by replacing its memory and CPU with a JavaScript VM. Ripping away the many layers of abstraction of JavaScript is clearly necessary to get at performance improvements, but they must be re-added in some way so that these high-performing applications can be useful in *context*.[^2]

A logical approach is more targeted optimization. Rather than inserting a whole native application into a browser, just its computation-intensive components (graphics engines, image processing, linear algebra, etc.) could be compiled into modular libraries, and interaction rebuilt on top of them using existing standards (HTML and vanilla JavaScript). This would be similar to the optimized APIs offered by HTML5 ([`<video>`](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Using_HTML5_audio_and_video), [`<canvas>`](http://www.w3schools.com/html/html5_canvas.asp), [Web Audio](https://developer.mozilla.org/en-US/docs/Web_Audio_API), and more), but would let developers create (and share) new ones without needing to rebuild a browser. The goal: performance of a native (or asm.js) plugin, with the simplicity of loading any other JavaScript library.

Some projects are working towards this, particularly [LLJS](http://mbebenita.github.io/LLJS). The "bastard child of JavaScript and C," it allows static typing and manual memory management when needed within normal JavaScript. Though it currently compiles to (quite human-readable) JavaScript, perhaps future interpreters could optimize for the uncompiled source. The innards of demanding modules could be written this optimizable way (or probably translated from existing C code), but natively interact with dynamic JavaScript within the same code. This would make for high-performing, highly-usable, still-human-readable modules that could easily be shared across projects, equally accessible to developers just using the functionality through its high-level interface as for those tweaking or learning from its implementation.

Despite its many pitfalls, highly-optimizable, lower-level JavaScript has huge potential to bring about the next step in the Web's evolution. After all, the Web was designed with the express purpose of being universal, allowing as many different components as possible to connect, and to share the benefits of those connections. Resources should not be excluded based on performance concerns, so reducing them only increases the diversity&mdash;and therefore value&mdash;of the Web.

One way or another, the future of the Web is bound to involve a more powerful execution environment. The question is how&mdash;and whether&mdash;it will be assembled.

[^1]: The type coercion syntax of asm.js is simply a codification of an existing syntactical trick that already acts as an optimization hint to most existing interpreters, and is valid JavaScript, but has no actual meaning. At best, it will speed up any browser that sees it, at worst, it will be ignored, but it won't break anything. (asm.js is a *strict* subset of JavaScript, after all.)

[^2]: "Ripping away layers of abstraction" is an admittedly funny way to describe building a low-level assembly language on top of a high-level interpreted one. But this is a rare case where adding a few more layers of abstraction (cros-compilation) has created the illusion that many more have been removed.

[^3]: This will [hopefully change](http://ejohn.org/blog/asmjs-javascript-compile-target) with [typed objects](http://wiki.ecmascript.org/doku.php?id=harmony:binary_data) in the ECMAScript 6 specification. In general, the new spec could change much of the assembled JavaScript movement, depending on how much of asm.js or other strong typing schemes like LLJS are included natively.


# Bibliography
Berners-Lee, T. and Mendelsohn, N. ["The Rule of Least Power"](http://www.w3.org/2001/tag/doc/leastPower.html). W3C TAG Finding 23 February 2006. Accessed 17 Dec 2013.

Carlos, Sean. ["8 reasons to avoid Flash (or Silverlight) like the plague when designing a website"](http://antezeta.com/news/flash-problems). *Antezeta*. 10 Mar 2007. Accessed 16 Dec 2013.

ECMA International. ["ECMAScript Language Specification"](http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf). Ed 5.1. Geneva: June 2011. Accessed 15 Dec 2013.

Hanselman, Scott. ["JavaScript is Assembly Language for the Web: Sematic Markup is Dead! Clean vs. Machine-coded HTML"](http://www.hanselman.com/blog/JavaScriptIsAssemblyLanguageForTheWebSematicMarkupIsDeadCleanVsMachinecodedHTML.aspx). 6 July 2011. Accessed 14 Dec 2013.

Herman, D., Wagner, L., Zakai, A. ["asm.js Working Draft"](http://asmjs.org/spec/latest/). 12 Dec 2013. Accessed 16 Dec 2013.

Mozilla. ["LLJS : Low-Level JavaScript"](http://mbebenita.github.io/LLJS/). Accessed 16 Dec 2013.

Mozilla. ["Mozilla is Unlocking the Power of the Web as a Platform for Gaming."](https://blog.mozilla.org/blog/2013/03/27/mozilla-is-unlocking-the-power-of-the-web-as-a-platform-for-gaming/) *The Mozilla Blog*. 27 Mar 2013. Accessed 13 Dec 2013.

Raman, T.V. and Malhotra, A. ["Identifying Application State"](http://www.w3.org/2001/tag/doc/IdentifyingApplicationState). W3C TAG Finding 1 December 2011. Accessed 17 Dec 2013.

Resig, John. ["Asm.js: The JavaScript Compile Target"](http://ejohn.org/blog/asmjs-javascript-compile-target/). 3 April 2013. Accessed 14 Dec 2013.

Schiller, Scott. ["Is Flash Dead Yet?"](http://isflashdeadyet.com/). 2010. Accessed 16 Dec 2013.

Vilk et al. ["DOPPIO: Breaking the Browser Language Barrier"](https://web.cs.umass.edu/publication/docs/2013/UM-CS-2013-026.pdf). *University of Massachusetts, Amherst*. UM-CS-2013-026. Accessed 15 Dec 2013.

Wittens, Steven. ["On Asm.js: Ending The Ice Age of JavaScript"](http://acko.net/blog/on-asmjs/). 27 Nov 2013. Accessed 13 Dec 2013.

Zakai, Alon (Mozilla). [*emscripten*](https://github.com/kripken/emscripten/commit/1a007b1631509b9d72499a8f4402294017ee04dc). GitHub repository. 7 Dec 2013. Accessed 17 Dec 2013.

Zakai, Alon. ["Big Web App? Compile It!"](http://kripken.github.io/mloc_emscripten_talk/#/) Accessed 14 Dec 2013.


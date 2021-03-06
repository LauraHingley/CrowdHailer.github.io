---
layout: post
title: Opening Clojure
description: Impression from my first day with clojure
date: '2014-03-13T10:01:00+00:00'
tags:
- Clojure
- functional programming
author: Peter Saxton
tumblr_url: http://crowdhailer.tumblr.com/post/79448251181/opening-clojure
---
<p>Today was my first day playing with Clojure. I have heard snippets about it from so many different sources, most recently in a talk at makers from <a href="https://twitter.com/fgeorge52" title="Fred George on Twitter" target="_blank">Fred George.</a> </p>
<p><img alt="image" src="https://31.media.tumblr.com/516e5be0cc307adbdc22f811eeed91e4/tumblr_inline_n2dgj4yDHX1s4ay8u.png"/></p>

<p><!-- more --></p>
<p><strong>Intro</strong></p>
<p>For an incredibly succinct and accessible demonstration of clojure <a href="http://tryclj.com/" title="Try Clojure" target="_blank">try Clojure</a>. I was directed to this website and had to run through it. I was hooked immediately. The thing that most blew me away, in five minutes of clojure, was how it treated fractions. It remembers fractions not nasty inaccurate decimals.</p>
<blockquote>
<p>user=&gt; (/ 9 4)<br/>9/4<br/>user=&gt; (type (/ 9 4))<br/>clojure.lang.Ratio</p>
</blockquote>
<p>So the sample above shows that this memory is Ratio type. It also quite clearly shows how far this is from ruby and python. The odd structure &ldquo;(/ 9 4)&rdquo; is division in <a href="http://en.wikipedia.org/wiki/Polish_notation" title="Polish_notation wiki" target="_blank">polish prefix notation</a>. </p>
<p><strong>Installing</strong></p>
<p>Getting clojure installed on my machine was the easiest thing ever. The recommended method is to install using <a href="http://leiningen.org/" title="Leiningen" target="_blank">Leiningen</a> which is a &rsquo;<em>build automation and dependency management tool&rsquo;</em> for clojure. As Leiningen is already available in the apt core libraries installing it was simply to run the following command</p>
<blockquote>
<p>sudo apt-get install leiningen</p>
</blockquote>
<p>Done</p>
<p><strong>Final notes</strong></p>
<p>I can now run the <a href="http://clojure.org/repl_and_main" title="repl_and_main" target="_blank">Clojure REPL</a> (read-eval-print-loop) which is the interactive environment. This is started with the command &lsquo;clojure&rsquo;. Exiting is more tricky and is using.</p>
<blockquote>
<p>user=&gt;(System/exit 0)</p>
</blockquote>
<p>Or to crash out CTRL + D</p>
<p>So I am sure I will be writing more about clojure here, hopefully not too soon as it will mean I have been distracted from the important business of my final project.</p>
<p>Addition</p>
<p>I have just started <a href="http://www.4clojure.com/" title="4clojure - home" target="_blank">4clojure</a> to learn the language</p>

---
layout: post
title: 'Fred George talk and Microservice excitement'
description: My introduction to the microservice architecture
date: '2014-03-26T13:08:00+00:00'
tags:
- microservices
- programming
- object oriented programming
tumblr_url: http://crowdhailer.tumblr.com/post/80774246592/fred-george-talk-and-microservice-excitement
---
<p><strong>Recently we had an excellent talk from <a href="https://twitter.com/fgeorge52" title="@fgeorge52" target="_blank">Fred George</a></strong>. I very much enjoyed the unconventional start to the talk. As it  was unplanned he started with a question to the audience which was &lsquo;what would you like me to talk about&rsquo;. My favourite most meta question was  from one of the cohort below ours. 'What is the answer to the question I don&rsquo;t yet know how to ask?&rsquo;. It turns out the answer to that question was clojure. The other topics covered everything from advice for people starting out to what are you top three languages. Unfortunately I can&rsquo;t remember the answers but I think <a href="http://en.wikipedia.org/wiki/Smalltalk" title="Smalltalk - Wikipedia" target="_blank">Smalltalk</a> was included.</p>
<blockquote>
<p>When certain developers talk about small-talk it reminds me of oceans 11 or 13. 'You should know better you shook Sinatra&rsquo;s hand&rsquo;. There is an understanding among Smalltalk developers</p>
</blockquote>
<p>As the title suggest microservices most caught my attention.</p>
<p><strong>What is a microservice?</strong></p>
<p><strong><!-- more --></strong></p>
<p>It is an extension of the idea of separating code that starts with <a href="http://en.wikipedia.org/wiki/Object-oriented_programming" title="Object Oriented Programming" target="_blank">OOP</a> and <a href="http://en.wikipedia.org/wiki/Model-view-controller" title="Model View Controller" target="_blank">MVC</a>. With cheap accessible servers available through providers like <a href="https://www.heroku.com/" title="Heoku | Cloud application platform" target="_blank">Heroku</a> code can be disassembled into separately <a href="http://davidmorgantini.blogspot.co.uk/2013/08/micro-services-introduction.html" title="Micro-services - Introduction" target="_blank">deployable apps</a>. Depending on context these apps can though <a href="http://yobriefca.se/blog/2013/04/29/micro-service-architecture/" title="Micro Service Architecture" target="_blank">various api&rsquo;s</a>. Micro-services are something that can be half followed and this is a good article explains all the features that make a system a true <a href="http://12factor.net/" title="The Twelve-Factor App" target="_blank">micro-service</a>.</p>
<p><strong>what are the benefits?</strong></p>
<ol><li><span>Each service can be a different technology stack and the most appropriate for that task can be chosen</span></li>
<li><span>Services can more easily be migrated to new technologies as only one part at a time need to be upgraded</span></li>
<li>Small services perform one task only and perform it well and are simpler to understand</li>
</ol><p><strong>what are the costs?</strong></p>
<ol><li><span>Requires low network </span>latency</li>
<li><a href="http://en.wikipedia.org/wiki/Fallacies_of_Distributed_Computing" title="Fallacies of distributed computing" target="_blank"><span>Others I have yet to discover</span></a></li>
</ol><p><strong>Initially is seamed to me that there would always need to be some level of controller.</strong> Next day Fred was back in the office, probably to work, so thanks for more of his time which he took to answer all the questions I had.</p>
<p>It is possible to eliminate the controller entirely. For example to break a webpage to microservices separate Javascript files can import content for different services and display them in designated areas of the page. At no point does knowledge need to be shared amongst the code. </p>
<p>For our final project I have developed a microservice to serve a users CV from github. It provides the Javascript it needs and can be used by incorporating one target element and one script tag</p>

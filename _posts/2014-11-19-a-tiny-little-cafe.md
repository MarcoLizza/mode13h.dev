---
title: A Tiny Little Cafe
layout: post
permalink: /a-tiny-little-cafe/
categories:
  - languages
tags:
  - javascript
  - scripting
  - tinyjs
---
If you're in the search for a JavaScript interpreter for Your-Amazing-Application, I think this can be a valuable option.

The good [Gordon Williams][1] implemented this tiny little interpreter some time ago.

I really appreciate the spirit of the projects itself. Not aiming to be a full-fledged Javascript behemoth with a complex API and heavy memory footprint. But being a mean and lean implementation that, while lacking many the advanced Javascript features and not being byte(pre)compiled (i.e. not so blazing fast), it is small enough (consisting in a single `.h` and `.cpp` pair) to be incorporated with almost no hassle.

I took the liberty to fork the original (no longer maintained?) project on [GitHub][2] and make some minor modifications to suit my needs.

I will certainly use it anytime I need some basic but versatile scripting facility in my applications without weighting them too much.

*( for any advanced and mission critical uses I will go and use Lua, instead )*

 [1]: http://www.pur3.co.uk/
 [2]: http://github.com/mlizza/tiny-js
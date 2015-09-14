---
layout: post
title: Regarding Spaces...
date: 2015-09-14 12:00
author: Adam Presley
tags: development
comments: true
---
When asked about spaces in a repository URL, my response was **Spaces are the DEVIL!**. The more reasonable response would be...

<!-- excerpt -->

> Spaces are difficult because they serve a dual-purpose, both as a literal character and as a delimiter. Because
> of that ambiguity the usage of space for anything that might be used in an automation tool, or as an identifier,
> is discouraged.

Yes, one *can* use spaces in a URL, or folder name, or source file. However when the time comes for tooling around these resources the issue of encoding and escaping comes into play. If this is a headache you wish to persue, then name your source code file ```process that shopping cart.php``` and enjoy.

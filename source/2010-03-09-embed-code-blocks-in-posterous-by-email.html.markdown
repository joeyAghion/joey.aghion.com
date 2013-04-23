---
title: Embed code blocks in posterous posts by email
date: 2010-03-09 7:43
tags: 
---

This took a little figuring out, so hopefully this post will be useful for others. To embed monospaced, syntax-highlighted code in a posterous post by email, surround it in square-bracketed "code" and "/code" tags. You can even specify a language, e.g.: "code lang='ruby'"

Just make sure to compose your email in plain text, or the code will be packed with unwanted div tags.

    [code lang='ruby']
    switch_to(:plain_text) if current_format == :rich
    between("code", "/code") do
      yourHelloWorld || whatever
    end
    [/code]

Enjoy

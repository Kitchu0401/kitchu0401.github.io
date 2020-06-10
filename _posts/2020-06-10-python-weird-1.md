---
layout: post
title: "파이썬에 대한 단상 #1"
date: 2020-06-10 19:19:19 +0900
published: true
comments: true
categories: [python]
tags: [python, code-clip]
---

가끔 생각하지만 파이썬은 참 요상한 언어다..

``` py
list_a = list()
list_b = list()

word_a = 'some'  # should go b
word_b = 'text'  # should go b
word_c = 'here'
word_d = 'hayo!'  # should go a

d = {
    word_a: [(list_a, .6), (list_b, .75)],
    word_b: [(list_b, .3)],
    word_c: [],
    word_d: [(list_a, .9)]
}

[sorted(l, key=lambda x: x[1], reverse=True)[0][0].append(w) for w, l in d.items() if l]

print('list_a', list_a)
print('list_b', list_b)
print('none_list', [w for w, l in d.items() if not l])

# list_a ['hayo!']
# list_b ['some', 'text']
# none_list ['here']
```

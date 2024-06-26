---
layout: article
title: My First Post
aside:
  toc: true
show: false
pageview: true
cover: https://raw.githubusercontent.com/z3f1r0/z3f1r0.github.io/master/img/markdown.png
tag: Blog
---

# Welcome to my blog!
> #### *Such as West wind sometimes I am like a gentle breeze and sometimes I am like an heavy storm!*

Hi everyone! This is my first post on a blog built with [jekyll TeXt theme](https://github.com/kitian616/jekyll-TeXt-theme){:target="_blank"} by [kitian616](https://github.com/kitian616){:target="_blank"}. A very nice, powerful and customizable theme to build personal sites and blogs! I suggest to web passionate to try it.

I would like to start writing about my passion in information technology, especially in cybersecurity. Also introducing my personal opinion. 
I think I will publish different projects, procedures, interesting sheets and walkthroughts about the cyber world. 
I hope this blog could be help for someone.

## Where can I start?
I have to make practice with markdown syntax. 
I found a [very useful guide](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#links){:target="_blank"} and I should  write a lot in order to memorize every special format.
In this moment I am writing on [https://stackedit.io](https://stackedit.io){:target="_blank"}, a wonderful online editor for markdown.

![Alt text](https://raw.githubusercontent.com/z3f1r0/z3f1r0.github.io/master/img/stackedit.png)

I want to make practice with markdown in order to became faster in writing posts.

## Basic Markdown Syntax
<html>
<table>
  <thead>
    <tr>
      <th>Element</th>
      <th>Markdown Syntax</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Heading</b></td>
      <td><code># H1<br>
          ## H2<br>
          ### H3</code></td>
    </tr>
    <tr>
      <td><b>Bold</b></td>
      <td><code>**bold text**</code></td>
    </tr>
    <tr>
      <td><b>Italic</b></td>
      <td><code>*italicized text*</code></td>
    </tr>
    <tr>
      <td><b>Blockquote</b></td>
      <td><code>&gt; blockquote</code></td>
    </tr>
    <tr>
      <td><b>Ordered List</b></td>
      <td><code>
        1. First item<br />
        2. Second item<br />
        3. Third item<br /></code>
      </td>
    </tr>
    <tr>
      <td><b>Unordered List</b></td>
      <td>
        <code>
          - First item<br />
          - Second item<br />
          - Third item<br /></code>
      </td>
    </tr>
    <tr>
      <td><b>Code</b></td>
      <td><code>`code`</code></td>
    </tr>
    <tr>
      <td><b>Horizontal Rule</b></td>
      <td><code>---</code></td>
    </tr>
    <tr>
      <td><b>Link</b></td>
      <td><code>[title](https://www.example.com)</code></td>
    </tr>
    <tr>
      <td><b>Image</b></td>
      <td><code>![alt text](image.jpg)</code></td>
    </tr>
  </tbody>
</table> 
</html>

## Some examples

### Code
```javascript
var s = "JavaScript syntax highlighting";
alert(s);
```
You have to insert `code` inside "backticks", like this:
```
```javascript
var s = "JavaScript syntax highlighting";
alert(s);
```

### Youtube video (I like this)

<div>{%- include extensions/youtube.html id='x5wkIewzyNg' -%}</div>

To upload YouTube videos I used a [TeXt theme extension](https://kitian616.github.io/jekyll-TeXt-theme/docs/en/extensions#video){:target="_blank"} and I had to write something like this:
```
<div>{%- include extensions/youtube.html id='x5wkIewzyNg' -%}</div>
```

### Mermaid Flowchart

```mermaid
graph TB;
    A[Do you have a problem in your life?]
    B[Then don't worry]
    C[Can you do something about it?]
    A--no-->B;
    A--yes-->C;
    C--no-->B;
    C--yes-->B;
```

```
```mermaid
graph TB;
    A[Do you have a problem in your life?]
    B[Then don't worry]
    C[Can you do something about it?]
    A--no-->B;
    A--yes-->C;
    C--no-->B;
    C--yes-->B;
```

### Useful links
You can find lots of tips to write in markdown at the following links:
- [https://kitian616.github.io/jekyll-TeXt-theme/docs/en/additional-styles](https://kitian616.github.io/jekyll-TeXt-theme/docs/en/additional-styles){:target="_blank"}
- [https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#links](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#links){:target="_blank"}

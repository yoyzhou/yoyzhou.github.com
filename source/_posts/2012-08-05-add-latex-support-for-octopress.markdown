---
layout: post
title: "使用kramdown和MathJax给博客添加Latex支持"
date: 2012-08-05 15:24
comments: true
categories: [Octopress, Latex]
---

{% blockquote @kramdown http://kramdown.rubyforge.org/index.html %}
kramdown (sic, not Kramdown or KramDown, just kramdown) is a free GPL-licensed Ruby library for parsing and converting a superset of Markdown.  It is completely written in Ruby, supports standard Markdown.
{% endblockquote %}

##Install kramdown
First of all make sure you have ruby installed on your box:

{% codeblock %}
$ ruby --version # or
$ which ruby 
{%endcodeblock%}
If command "which ruby" returns nothing, then you should install it first.

{% codeblock %}
$ sudo apt-cache search ruby # to chech out the available ruby package
$ sudo apt-get install ruby[*.*] # to install *.* version of ruby
{%endcodeblock%}
Also you can use [RVM](https://rvm.io/), aka Ruby Version Manager, to install/manage your ruby entries, but actually I am not able to install ruby 1.9.2 approperately following the instruction from Octopress site on my PC which is *ubuntu 12.04*.

{% codeblock Install kramdown lang:sh %}
$ sudo gem install kramdown 
{% endcodeblock %}

## Change _config.yml and Gemfile to include kramdown
Change markdown engine to kramdown

{% codeblock Change markdown engine to kramdown _config.yml %}
[...]
markdown: kramdown
[...]
{% endcodeblock %}
<!-- more -->
Add kramdown to Gemfile

{% codeblock Add kramdown to Gemfile %}
[...]
gem 'pygments.rb', '~> 0.2.12'
gem 'kramdown', '~> 0.13'
gem 'RedCloth', '~> 4.2.9'
[...]
{% endcodeblock %}

Next, install gem dependencies

{% codeblock  lang:sh %}
gem install bundler
bundle install
{% endcodeblock %}

## Add MathJax.js to head.html
So far, we have installed kramdown, and attach the dependency of kramdown to project, the last thing we have to do is including MathJax.js to your html head, since by default MathJax.js is not included in html head.

{% codeblock Add MathJax lang:sh %}
$ vim [repo-root]/source/_includes/head.html #where repo-root is the repository root, optional. 
$ #add "<script src="http://kramdown.rubyforge.org/MathJax/MathJax.js" type="text/javascript"></script>" (without quote) to head.html 
{% endcodeblock %}
## Show Cases
$$
\begin{align}
E & = mc^2 
\end{align}
$$

$$
\begin{align}
F & = ma 
\end{align}
$$

$$
\begin{align}
e^{i \pi} & = -1
\end{align}
$$

And,

$$
\begin{align*}
  & \phi(x,y) = \phi \left(\sum_{i=1}^n x_ie_i, \sum_{j=1}^n y_je_j \right)
  = \sum_{i=1}^n \sum_{j=1}^n x_i y_j \phi(e_i, e_j) = \\
  & (x_1, \ldots, x_n) \left( \begin{array}{ccc}
      \phi(e_1, e_1) & \cdots & \phi(e_1, e_n) \\
      \vdots & \ddots & \vdots \\
      \phi(e_n, e_1) & \cdots & \phi(e_n, e_n)
    \end{array} \right)
  \left( \begin{array}{c}
      y_1 \\
      \vdots \\
      y_n
    \end{array} \right)
\end{align*}
$$







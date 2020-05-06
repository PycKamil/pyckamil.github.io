---
layout: post
title:  "Lesson learned (again)"
date:   2020-05-06 21:30:15 +0200
categories: programming
---
Even if you do proffesional programming for 10+ years you can think that easy `while` loop in bash is something simple. But it took 2 hours for me to figure out it's opposite. 

{% highlight shell %}

array=()

while element
do
array+= element
done

{% endhighlight %}

In every case `array` was always empty. What? 🤨
Turnes out while loop is executed in different subshell and don't have access to variable outside of it scope.

Solution: **Switch to for loop**.

---
layout: post
title:  "My experience contributing to open source software - pandas"
date:   2018-03-13
excerpt: ""
tag:
- blogpost
---
Open-source software brings together many altruistic programmers that freely distribute their code to the world. As a programmer that leverages open-source software to make a living I have always wanted to make a contribution back to the open source community. This past weekend I achieved this goal during the global pandas documentation sprint.

I know that I didn’t have to wait for this global event to make an open source contribution. In fact, I frequently creep the issue boards of open-source software that I frequently use. However, this being my first time contributing to an open-source project, it really helped having the support from the community along with detailed instructions. I was also given a very specific task, which also helped.

The pydata TO group hosted an event at the hacklab space in Toronto. The organizer assigned me the pandas.DataFrame.all documentation but before I got started I had to setup my environment. The pandas webpage has excellent instructions for doing this found [here.](http://pandas.pydata.org/pandas-docs/stable/contributing.html#creating-a-python-environment-pip)

I prefer to manage my virtual environments with venv and although there were a lot of warnings, everything worked pretty well. I could now start writing the documentation, which started as the following:

![jekyll Image]({{ site.url }}/assets/img/pandas_contrib/pandas_one.png)

I ran the following line of code to produce the doc string output.

{% highlight py %}
python ../scripts/validate_docstrings.py pandas.DataFrame.all
{% endhighlight %}

![jekyll Image]({{ site.url }}/assets/img/pandas_contrib/pandas_two.png)

Underneath the first set of hash-tags is what the doc-string currently looks like and underneath the second set is the current error list. The first error is:

{% highlight py %}
Docstring text (summary) should start in the line immediately after the opening quotes (not in the same line, or leaving a blank line in between)
Use only one blank line to separate sections or paragraphs
{% endhighlight %}


![jekyll Image]({{ site.url }}/assets/img/pandas_contrib/pandas_three.png)

I was given a line associated with this doc-string and was confronted with confusion when I read the code.


I was expecting something a little closer to what the actual docs looked like. After some investigation I realized that I was missing the magic within the decorators. The appender decorator appends the content found in the _bool_doc variable. The _bool_doc variable itself has references to variables that are interpolated from the arguments given in the substitution decorator.

When I tracked down the _bool_doc variable, I found the source of the first error starring at me.

![jekyll Image]({{ site.url }}/assets/img/pandas_contrib/pandas_four.png)

The %(desc)s variable placeholder has a blank line after the open quotes. This is not allowed. So I removed this blank line and voila! I had debugged my first error.

![jekyll Image]({{ site.url }}/assets/img/pandas_contrib/pandas_five.png)

I kept working away at these errors and was then presented with a challenge. I was completing the documentation for DataFrame.all but it actually shares documentation with the DataFrame.any method. So when I needed to put in a see also and examples section, I had to add arguments to the instantiation function of these docs. This would allow these two related methods share the template for their documentation but have differences where needed.

I added the examples and see_also arguments to the _make_logical_function and passed them in as variables.

![jekyll Image]({{ site.url }}/assets/img/pandas_contrib/pandas_six.png)

![jekyll Image]({{ site.url }}/assets/img/pandas_contrib/pandas_seven.png)

Below is an example of a variable that was passed to the argument that generated the docs.Notice how I added the _all_doc, _all_examples and _all_see_also variables tot he cls.all object.

![jekyll Image]({{ site.url }}/assets/img/pandas_contrib/pandas_eight.png)

These changes resulted in the new documentation page looking like the following

{% highlight py %}
python make.py --single pandas.DataFrame.all
{% endhighlight %}

![jekyll Image]({{ site.url }}/assets/img/pandas_contrib/pandas_nine.png)

![jekyll Image]({{ site.url }}/assets/img/pandas_contrib/pandas_ten.png)

An improvement for sure.

The next step was to get this branch ready to be merged. Including making sure that it compiled and passed the pep8 standards.

{% highlight py %}
git diff upstream/master -u -- "*.py" | flake8 --diff
{% endhighlight %}

I submitted a pull request and had comments within minutes. The comments ranged from stylistic changes to a request to add information about the all method being applicable to series data structures. Three separate core pandas developers commented on the branch. Although I frequently undergo this process at work it was really cool to do this with people that I’ve never met before across the world.

Finally at 10:42am on March 11th my branch was merged and I officially became a contributor to the pandas project in the 0.23.0 release. Merge can be found [here.](https://github.com/pandas-dev/pandas/pull/20216)

![jekyll Image]({{ site.url }}/assets/img/pandas_contrib/pandas_eleven.png)


I hope to make more contributions to open-source software and will hopefully contribute to implementation details as well as more documentation.

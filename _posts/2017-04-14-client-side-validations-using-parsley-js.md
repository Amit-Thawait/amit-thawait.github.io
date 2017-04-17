---
title: Super simple client side validations using Parsley JS
layout: post
published: true
category: programming
tags: [js]
comments: true
---

No html form in a web page is complete without client side validations.

The easiest way to do it is using [Parsley JS](http://parsleyjs.org/){:target="_blank"}.

Lets take a look step by step on how to add parsley js to a form.

Lets say you have simple form like this :

<script src="https://gist.github.com/Amit-Thawait/a54579e482d23b5495dc1668eb3abafe.js"></script>

It will look like :

<p align="middle">
	<img src="../assets/images/parsley_js/form_without_validation.png" alt="simple form">
	<figcaption align="middle">Form without validation</figcaption>
</p>

Now to add [Parsley JS](http://parsleyjs.org/){:target="_blank"} to this form you need to include :

1. jQuery (dependency of parsley js)
2. Parsley JS file
3. Parsley CSS file (for some styling) (Download it from [Parsley JS Bundle](https://github.com/guillaumepotier/Parsley.js/releases/tag/2.7.0))

To make a form to use parsley, you need to add `data-parsley-validate` attribute to the form tag and add `required` attribute to all the elements which you want to make mandatory for the end user.

There are many other built-in validators as well, like you can check a field for only number inputs or check for min length, max length, check for a specific patterns etc. For a detailed set of available options refer : [Build-in Validators](http://parsleyjs.org/doc/index.html#validators-list){:target="_blank"} 

Next, you need to call the function `parsley()` in the form object like this :

	$('#demo-form').parsley();

After adding all the above mentioned stuff, the form will look like this :

<script src="https://gist.github.com/Amit-Thawait/48a8b81afb67246a60335ffc0533d656.js"></script>

After you click on validate, it will look something like this :

<p align="middle">
	<img src="../assets/images/parsley_js/form_after_validation.png" alt="validation">
	<figcaption align="middle">Form after validation</figcaption>
</p>

There are some parsley functions that you need to keep in mind which I found very helpful :

1. Let's say at any given time if you want to validate a form either on click of a button or something else, then just call the `validate()` function on that form.

		$('#demo-form').validate();

2. Let's say at any given time if you want to check if a form is valid or not, then just check it by using `isValid()` function

		$('#demo-form').isValid()

`isValid()` will return `true` or `false` depending on user's input entered in the form.

There are lots of examples in their example section, do check it out for more at [Parsley JS Examples](http://parsleyjs.org/doc/examples.html){:target="_blank"} page.

For their detailed documentation refer [Parsley JS Documentation](http://parsleyjs.org/doc/index.html){:target="_blank"}.

That's how simple it is to integrate parsley js to a html form. I hope you found it useful.
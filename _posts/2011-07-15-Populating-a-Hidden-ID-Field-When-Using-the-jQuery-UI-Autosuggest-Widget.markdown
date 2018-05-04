---
layout: post
title:  "Populating a Hidden ID Field When Using the jQuery UI Autosuggest Widget"
date:   2011-07-15 13:00:00 -0400
categories: JavaScript
---

I've used [Prototype](http://www.prototypejs.org/) for a very long time as the JavaScript framework for many of our Web applicaitons, but we've recently been moving towards using [jQuery](http://jquery.com) as it has a much more vibrant developer ecosystem. In a recent project, I wanted to provide customers with the ability to type someone's name into a text field, and use the jQuery UI autosuggst widget to return search results rather than having customers search through a list of 30,000+ customer names. The [jQuery UI documentation](http://jqueryui.com/demos/autocomplete/) made it easy to get the autosuggest widget set up. What I needed to do, however, was figure out how to pass the customerID value for the selected customer from the autosuggest widget to my form because when the form was submitted, I wanted to work with the customerID value rather than a string composed of a first name and last name.

The service that is called by the jQuery autosuggest widget returns a simple JSON object which contains an array of "rows" with a first name, last name, and customer ID value. The key was adding the customerID value to the autosuggest widget in the success event handler that's part of the autosuggest widget setup:

{% highlight javascript %}
$j("input#customerName").autocomplete({
	minLength: 2,
	source: function( request, response ) {
		$j.ajax({
			url: "myService.cfc",
			cache: false,
			dataType: "json",
			data: {
				searchString: request.term
			},
			success: function( data ) {
				response( $j.map( data, function( item ) {
					return {
						label: item.lastName + ", " + item.firstName,
						value: item.lastName + ", " + item.firstName,
						customerID: item.customerID
					}
				}));
			}
		});
	}
});
{% endhighlight %}

The autosuggest success event handler takes the JSON returned by my service and loops through it, returning a JavaScript object which contains three values: the label that's displayed in the autosuggest widget, the value for the text input when something is selected, and &mdash; this isn't in the core jQuery UI docs &mdash; I added the customerID value so that I can use it later. This all gets passed to the autosuggest widget so that when a particular name is chosen, the correct value is displayed in the text field *and* the correct customerID is also utilized.

>One quick side note: the autosuggest control uses the value "term" as the name of the field it's going to pass to the URL defined in the setup. If your service or server-side handler expects a variable named something other than "term," you have to make this change in the data object in the autosuggest setup. In my example, my service expects a string with the name "searchString." So in the data object in the setup, I'm telling the autosuggest control to pass the "term" value to my service with the name "searchString," instead of "term."

When an item is selected from the possible choices in the autosuggest widget, the "select" event is called. That's where you can grab the customerID of the person who was selected. It's really simple to do this:

{% highlight javascript %}
select: function( event, ui ) {
	$j("#customerID").val(ui.item.customerID);
	console.log("CustomerID " + ui.item.customerID + " was selected.");
}
{% endhighlight %}

The ui.item object contains all of the data that you mapped out in the success event handler of the autosuggest control. I'm simply grabbing the ui.item.customerID value and assigning that to a hidden form field in the form called customerID. When the form is submitted, the customerID value is used for processing, rather than the string value of the selected person's name in the text input.

I wasn't able to find this particular (and, I think, common) use case in the jQuery UI docs, so hopefully it will be of help to someone!
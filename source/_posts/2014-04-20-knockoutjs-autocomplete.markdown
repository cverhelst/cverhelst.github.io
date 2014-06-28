---
layout: post
title: "KnockoutJS - AutoComplete"
date: 2014-04-20 15:44
comments: true
categories: sharepoint, knockoutjs, jquery
---

I've been looking into Knockout JS recently and wanted to see how it could be integrated with JQuery (and JQuery.UI) to have an autocomplete field.

Some of the examples I found were doing what I wanted, but too complicated for me to understand with my limited JavaScript experience or were just not very generic at all.

I also wanted to still have the original object supplying the label after a selection was made. This can be helpful to supply values to other fields afterwards, when you simply can't regenerate it from the label alone.

I did find one example that I managed to change to be quite "simple" and generic.

[JSFiddle: Full code & working example](http://jsfiddle.net/5PRMe/)

I'll run you through the sourcecode step by step:

## Original Data

We have a datasource that will supply the option list that will be used for the autocomplete functionality:

	// Array with original data
	var remoteData = [{
	    name: 'Ernie',
	    id: 1
	}, {
	    name: 'Bert',
	    id: 2
	}, {
	    name: 'Germaine',
	    id: 3
	}, {
	    name: 'Sally',
	    id: 4
	}, {
	    name: 'Daisy',
	    id: 5
	}, {
	    name: 'Peaches',
	    id: 6
	}];

Currently it's just an array containing objects with a `name`and an `id` property. 

## JQuery.UI Autocomplete widget

The original data array itself can be of any structure, but JQuery.UI's [autocomplete widget](http://api.jqueryui.com/autocomplete/#option-source) expects an array of strings as a minimum, they will be used for the label & value both, or you can supply an array of objects that have a `label` and a value `property`. Since we want a different value for the option's `label` and `value` we will use this object array. The `label` and `value` properties are mandatory, but we are free to add our own properties, which we will do using the following function to convert our initial data array to a proper JQuery.UI autocomplete widget's source array:

	function (element) {
        // JQuery.UI.AutoComplete expects label & value properties, but we can add our own
        return {
            label: element.name,
            value: element.id,
            // This way we still have acess to the original object
            object: element
        };
    };

As you can see, we've added a `source` property to hold our original object.

## ViewModel

As you may know, KnockoutJS is a MVVM framework, and here is our ViewModel that will be used for the autocomplete widget: 

	function ViewModel() {
	    var self = this;
	    
	    self.users = remoteData;
	    
	    self.selectedOption = ko.observable('');
	    self.options = self.users.map(function (element) {
	        // JQuery.UI.AutoComplete expects label & value properties, but we can add our own
	        return {
	            label: element.name,
	            value: element.id,
	            // This way we still have acess to the original object
	            object: element
	        };
	    });
	}

As you can see, it uses our previous function to convert the original data array to an options list. It also uses the **KnockoutJS Observable** to hold the selected value. We use the observable because we may want to know if it updates.

KnockoutJS's observables are implementations from the Observable pattern and it will automatically let any instances that depend on the Observable know if it's value is updated.

We use the `self` property for a few reasons, it's best explained in [this stackoverflow answer](http://stackoverflow.com/a/17163358). In short: it allows use to access the ViewModel from inside function scopes where `this` would refer to the function being implemented instead of the parent object(the ViewModel).


## KnockoutJS - Custom binding

To allow us to generically pass in the data for the Autocomplete Widget in the correct KnockoutJS manner, we will implement a [custom binding](http://knockoutjs.com/documentation/custom-bindings.html).

### View binding

I think it's easier to understand it's functionality when you see how it's being used:
	
	<input type="text" data-bind="autoComplete: { selected: selectedOption, options: options }" />

	<!-- Debugging -->
	<p data-bind="text: selectedOption().object.name"></p>

The input textbox is will be converted in the AutoComplete Widget by JavaScript code later.

The `data-bind` tag is KnockoutJS's declarative way of binding ViewModel to  the View (being HTML tags).

In the `data-bind` we can specify a **binding handler** which in this case is the `autocomplete` binding handler. Built-in handlers are for example `text`, which will just put the ViewModel's value inside the bound HTML tag as text.

Our custom binding handler will be a bit more complex. It takes a parameter that is an object with 2 properties: `selected` and `options`. The `selected` property must be a KnockoutJS Observable that will be updated with the option that was selected. The `options` property will be the JQuery.UI AutoComplete's source array with the options `labels` and `values`.

The properties passed are properties on the ViewModel, being `selectedOption` and `options`.

### The binding handler

	ko.bindingHandlers.autoComplete = {
	    // Only using init event because the 
		// Jquery.UI.AutoComplete widget will take care of the update callbacks
	    init: function (element, valueAccessor, allBindings, viewModel, bindingContext) {
	        // valueAccessor = { selected: mySelectedOptionObservable, options: myArrayOfLabelValuePairs }
	        var settings = valueAccessor();
	
	        var selectedOption = settings.selected;
	        var options = settings.options;
	
	        var updateElementValueWithLabel = function (event, ui) {
	            // Stop the default behavior
	            event.preventDefault();
	
	            // Update the value of the html element with the label 
	            // of the activated option in the list (ui.item)
	            $(element).val(ui.item.label);
	
	            // Update our SelectedOption observable
	            if(typeof ui.item !== "undefined") {
	                // ui.item - id|label|...
	                selectedOption(ui.item);
	            }
	        };
	
	        $(element).autocomplete({
	            source: options,
	            select: function (event, ui) {
	                updateElementValueWithLabel(event, ui);
	            },
	            focus: function (event, ui) {
	                updateElementValueWithLabel(event, ui);
	            },
	            change: function (event, ui) {
	                updateElementValueWithLabel(event, ui);
	            }
	        });
	    }
	};

This is a lot to take in at once. You should focus on the following:

- valueAccessor
	- Represents the passed in argument, being out object containing the observable for the selected options and the options array
- updateElementValueWithLabel
	- This is a function we call on each JQuery.UI Autocomplete widget's events to set the selectedOption observable and to set the textbox to contain the label. It has the [same parameters as the original widget events](http://api.jqueryui.com/autocomplete/#event-select)
- $(element).autoComplete(...)
	- This is how we convert the textbox to the JQuery.UI Autocomplete widget.
	- We override the default functionality in case an option is selected, the focus in the options list changes or the textbox value is changed.
	- Default functionlity is to place the value of the option list in the textbox (which is strange that it doesn't update it with the label).

#### Value accessor

	// valueAccessor = { selected: mySelectedOptionObservable, options: myArrayOfLabelValuePairs }
	var settings = valueAccessor();

The `valueAccessor` parameter of the binding deserves some explanation. I think typically it's not a complex object like in my case. So far I've seen people use [multiple bindings](http://stackoverflow.com/a/11378286) to pass extra values to their binding handler. I don't think it's very clean so I just pass one object, which has multiple properties for representing all the parameters. Nothing is static typed so using this approach or the multiple binding's is practically the same, in my opinion.

So now that we have our input, we read our individual parameters from it.

	var selectedOption = settings.selected;
	var options = settings.options;

#### Autocomplete widget

    $(element).autocomplete({
        source: options,
        select: function (event, ui) {
            updateElementValueWithLabel(event, ui);
        },
        focus: function (event, ui) {
            updateElementValueWithLabel(event, ui);
        },
        change: function (event, ui) {
            updateElementValueWithLabel(event, ui);
        }
    });

This is pretty straigthforward, `source` property takes the list of options and then we override the events on the widget.

#### Update Element Value With Label

    var updateElementValueWithLabel = function (event, ui) {
        // Stop the default behavior
        event.preventDefault();

        // Update the value of the html element with the label 
        // of the activated option in the list (ui.item)
        $(element).val(ui.item.label);

        // Update our SelectedOption observable
        if(typeof ui.item !== "undefined") {
            // ui.item - label|value|...
            selectedOption(ui.item);
        }
    };

This function will stop the default behavior of updating the textbox with the option's `value`, on the `ui.item` object, we want to use its `label` instead.

Finally, we update the selectedOption Observable with the whole item from the option array, containing the mandatory `label` & `value` properties, as well as our own `object` property containing the original data item.

## KnockoutJS

Don't forget the mandatory KnockoutJS initialization code:

	$(function () {
	    ko.applyBindings(new ViewModel());
	});
	
## Full Code
	
### HTML 

	<input type="text" data-bind="autoComplete: { selected: selectedOption, options: options }" />

	<!-- Debugging -->
	<p data-bind="text: selectedOption().object.name"></p>

### JavaScript

	ko.bindingHandlers.autoComplete = {
	    // Only using init event because the Jquery.UI.AutoComplete widget will take care of the update callbacks
	    init: function (element, valueAccessor, allBindings, viewModel, bindingContext) {
	        // { selected: mySelectedOptionObservable, options: myArrayOfLabelValuePairs }
	        var settings = valueAccessor();
	
	        var selectedOption = settings.selected;
	        var options = settings.options;
	
	        var updateElementValueWithLabel = function (event, ui) {
	            // Stop the default behavior
	            event.preventDefault();
	
	            // Update the value of the html element with the label 
	            // of the activated option in the list (ui.item)
	            $(element).val(ui.item.label);
	
	            // Update our SelectedOption observable
	            if(typeof ui.item !== "undefined") {
	                // ui.item - label|value|...
	                selectedOption(ui.item);
	            }
	        };
	
	        $(element).autocomplete({
	            source: options,
	            select: function (event, ui) {
	                updateElementValueWithLabel(event, ui);
	            },
	            focus: function (event, ui) {
	                updateElementValueWithLabel(event, ui);
	            },
	            change: function (event, ui) {
	                updateElementValueWithLabel(event, ui);
	            }
	        });
	    }
	};
	
	// Array with original data
	var remoteData = [{
	    name: 'Ernie',
	    id: 1
	}, {
	    name: 'Bert',
	    id: 2
	}, {
	    name: 'Germaine',
	    id: 3
	}, {
	    name: 'Sally',
	    id: 4
	}, {
	    name: 'Daisy',
	    id: 5
	}, {
	    name: 'Peaches',
	    id: 6
	}];
	
	function ViewModel() {
	    var self = this;
	    
	    self.users = remoteData;
	    
	    self.selectedOption = ko.observable('');
	    self.options = self.users.map(function (element) {
	        // JQuery.UI.AutoComplete expects label & value properties, but we can add our own
	        return {
	            label: element.name,
	            value: element.id,
	            // This way we still have acess to the original object
	            object: element
	        };
	    });
	}
	
	$(function () {
	    ko.applyBindings(new ViewModel());
	}); 
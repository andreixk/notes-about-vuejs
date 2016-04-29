## Notes About VueJS
Some notes and nuances I discovered while using VueJS

I recently discovered a new front end framework: [VueJS](http://vuejs.org/). So far it meets my expectations and then some! But as any tool it has some nuances of which you must be mindful. Here are a few (for version **1.0.16**).

### Nuance 1 - mixins
A [mixin](http://vuejs.org/guide/mixins.html) is **not** a child of an element - it **is** the element. As such, doing things like `$dispatch` and `$broadcast` from within a mixin will not trigger events on the element itself. To trigger an event on the element you must use `$emit`.

### Nuance 2 - filters, bitwise operators
[Filters](http://vuejs.org/api/#Filters) are awesome, but if you need to do a bitwise OR operation, you're out of luck: either use fancy tricks or create a function.
example:

```javascript
//setting bit #2 on this.flag:
<a @click="flag=flag|4">link</a>
//will fail since VueJS is trying to find a filter '4'
```
To set bit #2 for example, you could:
```javascript
flag = (flag & ~4) ^4
//this is equivalent to resetting bit 2 to zero first
// then setting it to 1 through XOR
```
-OR-
```html
<!--html:-->
<a @click.prevent="setFlagBit(4)">link</a>
```
```javascript
//js

//...inside the component itself
methods:{
	setFlagBit:function(bitNum){
		this.flag = this.flag | bitNum;
	}
	//...other methods
}
```
Bonus tip: if you just want to toggle a bitwise flag:
```html
<a @click.prevent="flag^=4">toggler</a>
```

### <a name="nuance3"></a>Nuance 3 - checkboxes

If you wish to run a method when clicking a checkbox, note this:
_to get the **OLD** value of the checkbox (i.e. before click) you can use `@click` method:_
```html
<input type="checkbox" v-model="myCb" @click="cbClicked">
```
_To get the **NEW** value, you need to use getter/setter:_
```javascript
//...
data:{
	cbValue:false
},
computed:{
  myCb:{
	  get:function(){
	  return this.cbValue;
	},
	set:function(checked){
	  this.cbResultSet = checked;
	  this.cbValue=checked;
	}
  }
}
//...
```
[jsfiddle](https://jsfiddle.net/ecq21wjq/)

### Nuance 4 - working with jinja (flask)

By default VueJS uses "{{" "}}" delimeters for parsing its variables. So does Jinja template engine used by Flask.

If you **NEVER** parse templates (conaining VueJS code) from Flask, and never plan to. You can stop reading.

In order to get them working together you need to either (1) change the Jinja delimeters, or (2) change VueJS delimeters.

**-1-**
```python
#normally you'd do this: app = Flask(...)
#here's how to override the default delimeters
class CustomFlask(Flask):
	""" this is required to customize the template field delimeters """
	jinja_options = Flask.jinja_options.copy()
	jinja_options.update(dict(
		block_start_string='<%',
		block_end_string='%>',
		variable_start_string='%%',
		variable_end_string='%%',
		comment_start_string='<#',
		comment_end_string='#>'
	))


app = CustomFlask(__name__)
# now instead of {{ varname }} you need to use:
# %% varname %%
# and including templates is done like this:
# <% include 'path/to/template/file' %>
```
[Read](http://jinja.pocoo.org/docs/dev/templates/) more about jinja templates


**-2-** as described [here](http://vuejs.org/api/#delimiters)
```javascript
Vue.config.delimiters = ['(*', '*)']
```

### Nuance 5 - global event listeners (e.g. from window.addEventListener)

When creating a global event listener (such as 'hashchange' or 'keyup') make sure you remove them if the component is removed, otherwise you'll end up with functions that are trying to access undefined properties.

e.g.:
```javascript
{
	//...
	methods:{
		checkHash:function (e) {
			console.log(e.newURL);
			this.someProperty=e.newURL;
		}
	}
	,ready:function () {
		window.addEventListener('hashchange',this.checkHash);
	}
}
```
This will work great when the component is active and well. However, even when it's destroyed, `checkHash` will still get called (since it's now in window's event listeners), and you'll see the new hash in the console log! **BUT**, `this.someProperty` inside `checkHash` will be _undefined_ (since the component was destroyed). And you'll get some odd behaviour.

So do this:

```javascript
{
	//...inside a component (or mixin) definition
	methods:{
		checkHash:function (e) {
			console.log(e.newURL);
			this.someProperty=e.newURL;
		}
	}
	,ready:function () {
		window.addEventListener('hashchange',this.checkHash);
	}
	//NB: this will ensure your listener is gone from the DOM tree.
	,beforeDestroy:function () {
		window.removeEventListener('hashchange',this.checkHash);
	}
}
```

### Nuance 6 - property/data naming

I don't believe the [documentation](http://vuejs.org/guide) mentions this explicitly, but component data fields cannot start with an underscore.

The following code won't work properly. You'll get a warning that you're trying to use a non-existing model `_someField`:

```javascript
//...
data:function () {
	return {
		_someField:''
	}
}
//...
```
```html
<input type="text" v-model="_someField"></input>
```

Same goes for computed properties. Apprently Vue reserves underscored names for internal purposes.


### _A Finding_

Turns out VueJS has an undocumented `Vue.util.extend` function very similar to jQuery's to create a deep copy of an object:

```javascript
var myObj = {id:25};
var func = function(someObject){
	tmpObj = someObject;
	tmpObj.id=15; //this will reset myObj.id to 15
	tmpObj = Vue.util.extend({},someObject);
	tmpObj.id=15; //myObj.id will stay intact
}
func(myObj);
```

### Nuance 7 - Counting/accessing filtered results

_NB: tested in Vue 1.0.20_

When using filterBy in a v-for loop, occasionally you may want to display some message if there's nothing found or do something else with filtered results. The easiest way I found (as Evan [described](https://github.com/vuejs/Discussion/issues/387#issuecomment-139350747)) is to use a computed property:

```javascript
// in your component:
data:{
		items:[{id:1,name:'abc'},{id:2,name:'abd'},{id:3,name:'cde'}],
		searchText:''
},
computed:{
	filteredResults:function() {
		var filter = Vue.filter('filterBy');
		return filter(this.items,this.searchText,'name');
	}
}
// now you can do things like this.filteredResults.length in your methods
// or iterate through filteredResults as a regular array
```

```html
<!-- in your html: -->
<input type="text" v-model="searchText">
<div v-for="item in filteredResults">
	{{item.name}}
</div>
<div v-if="!filteredResults.length">
	Nothing found, sorry!
</div>
```
[jsfiddle](https://jsfiddle.net/8gukrnbe/)

_NB: you can also use custom filters: `v-for="item in items | filterBy searchText in 'name' | customAfterFilter "` and do something like Vue.$set() inside it, but that's a hackier way IMHO_

### Nuance 8 (addition to [Nuance 3](#nuance3))

A way to get the BEFORE and AFTER value of a checkbox (or radio button) is to watch that property (which is much shorter in code anyway).

```javascript
watch:{
  	myCb:function(valueAfter, valueBefore){
      	this.cbResultClick = valueBefore;
      	this.cbResultSet = valueAfter;
  }
}
```
[updated jsfiddle](https://jsfiddle.net/ecq21wjq/1/)

### Nuance 9 - Order of declaration

If you've simply added `<script scr="vue.js">` tag and started hacking away at the code, note that you must create components BEFORE running your `new Vue(...)` code, otherwise you'll get an error about some unknown component.
```html
<!--suppose this is your HTML-->
<div id="app">
	<mycomp></mycomp>
</div>
<template id="mycomp-component">
	eeee
</template>
```

```javascript
// this code works
;(function(){
	Vue.component('mycomp',{
		template:'#mycomp-component'
	});

	new Vue({
		el: '#app',
		data: {
			message: 'Hello Vue.js!'
		}
	});
})();
```

```javascript
// this code doesn't
;(function(){
	new Vue({
		el: '#app',
		data: {
			message: 'Hello Vue.js!'
		}
	});
	Vue.component('mycomp',{
		template:'#mycomp-component'
	});
})();
```

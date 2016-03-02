### Notes About VueJS
Some notes and nuances I discovered while using VueJS

I recently discovered a new front end framework: [VueJS](http://vuejs.org/). So far it meets my expectations and then some! But as any tool it has some nuances of which you must be mindful. Here are a few (for version **1.0.16**).

#### Nuance 1 - mixins
A [mixin](http://vuejs.org/guide/mixins.html) is **not** a child of an element - it **is** the element. As such, doing things like `$dispatch` and `$broadcast` from within a mixin will not trigger events on the element itself. To trigger an event on the element you must use `$event`.

#### Nuance 2 - filters, bitwise operators
[Filters](http://vuejs.org/api/#Filters) are awesome, but if you need to do bitwise operations, you're out of luck: either use fancy tricks or create a function.
example:

```javascript
//setting bit #2 on this.flag:
<a @click="flag=flag|4">link</a>
//will fail since VueJS is trying to find a filter '4'
```
To set bit #2 for example, you could:
```javascript
flag = (flag & ~4) ^4 //equivalent to resetting bit 2 to zero first, then setting it to 1 through XOR
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

### Nuance 3 - checkboxes

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

**1:**
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


**2:** as described [here](http://vuejs.org/api/#delimiters)
```javascript
Vue.config.delimiters = ['(*', '*)']
```

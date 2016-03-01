### Notes About VueJS
Some notes and nuances I discovered while using VueJS

I recently discovered a new front end framework: [VueJS](http://vuejs.org/). So far it meets my expectations and then some! But as any tool it has some nuances of which you must be mindful. Here are a few (for version **1.0.16**).

#### Nuance 1 - mixins
A [mixin](http://vuejs.org/guide/mixins.html) is **not** a child of an element - it **is** the element. As such, doing things like `$dispatch` and `$broadcast` from within a mixin will not trigger events on the element itself. To trigger an event on the element you must use `$event`.

#### Nuance 2 - filters
<a href="" target="_blank">Filters</a> are awesome, but if you need to do bitwise operations, you're out of luck: either use fancy tricks or create a function.
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
<a @click="setFlagBit(4)">link</a>
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

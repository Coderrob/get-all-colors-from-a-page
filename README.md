# Get ALL Colors From A Page

I had a friend ask how they'd get all the colors used in a running webpage. I have never really thought about how to do that before. Honestly, first thought was about scraping style files, page level style blocks, and even checking inline styles.

Turns out there was a much more novel approach using the `window` object to get the computed styles.

Let's take a look at the solution I came up with to accomplish the goal:

```javascript
    (() => {
      const allColors = [];
      document
        .querySelectorAll('*')
        .forEach(element => {
          const styles = window.getComputedStyle(element, null);
          Object
            .keys(styles)
            .filter(property => !isNaN(+property) && styles[property]?.indexOf('color') > -1)
            .map(property => styles.getPropertyValue(styles[property]))
            .forEach(value => {
              if (value && allColors.indexOf(value) < 0) {
                allColors.push(value);
              }
            });
        })
      console.log('allColors', allColors);
    })();
```

First thing to do is to select _all_ the elements on the page.

We can do this by using the `document` to query all the elements:

```javascript
document.querySelectorAll('*');
```

The `querySelectorAll` function will return back a list of elements, of type `NodeListOf<Element>`, that supports an iterator. That allows us to use `forEach` to iterate over the collection instead of writing any for loops with ordinal based lookups.

Next, we can use the `window` object to give us the computed styles for each element. If you've used a browser's dev tools to inspect an element on a page you've likely seen the "computed" properties for an element.

We can request the values for those properties using the `window.getComputedStyle` function.

In the code we iterate over the elements and request the computed style for each element:

```javascript
      document
        .querySelectorAll('*')
        .forEach(element => {
          const styles = window.getComputedStyle(element, null);
```

The `getComputedStyle` function returns back an `CSSStyleDeclaration` object. This `styles` object now contains _both_ numeric property names with kebab cased CSS property name values, and camel cased property names with the computed values. 

Sounds weird but here is an example of what the object looks like:

```json
{
    "0": "accent-color",
    "1": "align-content",
    "2": "align-items",
    "3": "align-self",
    "4": "alignment-baseline",
    "5": "animation-delay",
    "6": "animation-direction",
    "7": "animation-duration",

    . . .

    "accentColor": "auto",
    "additiveSymbols": "",
    "alignContent": "normal",
    "alignItems": "normal",
    "alignSelf": "auto",
    "alignmentBaseline": "auto",
    "all": "",
    "animation": "none 0s ease 0s 1 normal none running",
    "animationDelay": "0s",
    "animationDirection": "normal",
    "animationDuration": "0s",

    . . . 
}
```

From the `styles` object we will get all the object's property names using `Object.keys`.

```javascript
Object
    .keys(styles)
```

The numeric property names map to the standard kebab cased property names. We need those property names to lookup their computed values for the element.

Using the unary `+` operator we can attempt to convert the string safely to a number.

If the property name is not a number we ignore it by using `isNaN` then we look at its key value to see if the style property name contains `color`.

```javascript
.filter(property => !isNaN(+property) && styles[property]?.indexOf('color') > -1)
```

From the filtered list of properties we can use the `CSSStyleDeclaration` object's `getPropertyValue` function to get the computed style value associated with each of the properties.

Using `map` we create a new array of the computed property style values from our color properties:

```javascript
.map(property => styles.getPropertyValue(property))
```

Next we iterate the array of property values. Because so many styles are likely duplicated we need to make sure only color styles that have defined non-empty string values and the color style value haven't already been added to `allColors`.

```javascript
.forEach(value => {
  if (value && allColors.indexOf(value) < 0) {
    allColors.push(value);
  }
});
```

After the elements have all been iterated we will have the unique list of all color style values used in the page.

To use this script on any site you can minify and compress the script into a single line. For convenience I've pasted the minified version below:

```javascript
(()=>{const e=[];document.querySelectorAll("*").forEach(o=>{const l=window.getComputedStyle(o,null);Object.keys(l).filter(e=>!isNaN(+e)&&l[e].indexOf("color")>-1).map(e=>l.getPropertyValue(l[e])).forEach(o=>{o&&e.indexOf(o)<0&&e.push(o)})}),console.log("allColors",e)})();
```

With this script you can inspect any site by pasting the script into the browser's console window, hit enter, and you'll get the full list of colors used on the page.

For example from Linkedin.com login page I was able to grab the list of unique colors by pasting the script into the console, hitting enter, and copying the console object.

```javascript
[
  "auto",
  "rgba(0, 0, 0, 0)",
  "rgb(0, 0, 0)",
  "srgb",
  "linearrgb",
  "rgb(255, 255, 255)",
  "economy",
  "rgba(0, 0, 0, 0.18)",
  "rgba(0, 0, 0, 0.9)",
  "rgb(10, 102, 194)",
  "rgba(0, 0, 0, 0.6)",
  "rgb(143, 88, 73)",
  "rgb(209, 17, 36)",
  "rgb(232, 240, 254)",
  "rgb(41, 119, 201)",
  "rgb(0, 115, 177)",
  "rgba(0, 0, 0, 0.08)",
  "rgb(178, 64, 32)",
  "rgba(0, 0, 0, 0.15)",
  "rgba(0, 0, 0, 0.16)",
  "rgba(0, 0, 0, 0.75)"
]
```

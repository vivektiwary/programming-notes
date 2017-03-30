##React Fundamentals

**Rendering elements**

Elements are the smallest building blocks of React apps.

An element describes what you want to see on the screen:
```jsx
const element = <h1>Hello, world</h1>;
```

Unlike browser DOM elements, React elements are plain objects, and are cheap to create. React DOM takes care of updating the DOM to match the React elements.

> **Note**:
>
>One might confuse `elements` with a more widely known concept of "components". `Elements` are what `components` are "made of".

**Rendering an Element into the DOM**

Let's say there is a <div> somewhere in your HTML file:

```html
<div id="root"></div>
```

We call this a "root" DOM node because everything inside it will be managed by `React DOM`.

Applications built with `just React` usually have a `single root DOM node`. If you are integrating React into an existing app, you may have as many `isolated root DOM nodes` as you like.

To render a React element into a root DOM node, pass both to `ReactDOM.render()`:

```jsx
const element = <h1>Hello, world</h1>;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```
It displays "Hello World" on the page.

**Updating the Rendered Element**

React elements are `immutable`. Once you create an element, you `can't change its children or attributes`. An `element` is like a `single frame` in a movie: it represents the UI at a `certain point in time`.

With our knowledge so far, the only way to update the UI is to create a new `element`, and pass it to `ReactDOM.render()`.

Consider this ticking clock example:
```jsx
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```
It calls `ReactDOM.render()` every second from a `setInterval()` callback.

>**Note**:
>
>In practice, most React apps only call `ReactDOM.render()` once.

**React Only Updates What's Necessary**

React DOM `compares` the element and its children to the `previous one`, and only applies the DOM updates necessary to bring the DOM to the desired state.

You can verify by inspecting the `last example` with the browser tools.

Even though we create an element describing the whole UI tree on every tick, only the text node whose contents has changed gets updated by React DOM.

In our experience, thinking about how the UI should look at any given moment rather than how to change it over time eliminates a whole class of bugs.


## Components and Props

Components let you split the UI into independent, reusable pieces, and think about each piece in isolation.

Conceptually, components are like `JavaScript functions`. They accept `arbitrary inputs` (called "props") and return `React elements` describing what should appear on the screen.

**Functional and Class Components**

The simplest way to define a component is to write a JavaScript function:
```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

This function is a valid `React component` because it accepts a single "props" object argument with data and returns a `React element`. We call such components `"functional"` because they are literally `JavaScript functions`.

You can also use an `ES6` class to define a component:
```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```
The above two components are equivalent from React's point of view.
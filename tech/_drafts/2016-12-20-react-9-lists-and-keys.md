---
layout: post
title: React 9 - 列表及其主键（Lists and Keys）
tags: 原创 技术 翻译 React
---

[原文](https://facebook.github.io/react/docs/lists-and-keys.html)

First, let's review how you transform lists in JavaScript.

在本章开始之前，让我们首先回顾一下在JavaScript中的数组转换。

Given the code below, we use the [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) function to take an array of `numbers` and double their values. We assign the new array returned by `map()` to the variable `doubled` and log it:

如下面的代码所示，使用数组的[`map()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map)方法将`numbers`数组转换为其两倍值组成的新数组。将该结果赋值给`doubled`并且输出日志：

```javascript
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((number) => number * 2);
console.log(doubled);
```

This code logs `[2, 4, 6, 8, 10]` to the console.

上述代码在控制台中输出日志`[2, 4, 6, 8, 10]`。

In React, transforming arrays into lists of [elements](/tech/2016/12/14/react-4-rendering-elements.html) is nearly identical.

在React中，将数组转换为[元素](/tech/2016/12/14/react-4-rendering-elements.html)列表的方式和上述方法基本相同。

### 多组件渲染（Rendering Multiple Components）

You can build collections of elements and [include them in JSX](/tech/2016/12/14/react-3-introducing-jsx.html#embedding-expressions-in-jsx) using curly braces `{}`.

可以创建元素集合，并用一对大括号`{}`[在JSX中直接将其引用](/tech/2016/12/14/react-3-introducing-jsx.html#embedding-expressions-in-jsx)即可。

Below, we loop through the `numbers` array using the Javascript [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) function. We return an `<li>` element for each item. Finally, we assign the resulting array of elements to `listItems`:

下面的例子中，用JavaScript的[`map()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map)方法将`numbers`数组循环处理，将其中每个数字转换为`<li>`元素，并将其构成的结果数组赋值给`listItems`：

```javascript
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);
```

We include the entire `listItems` array inside a `<ul>` element, and [render it to the DOM](/tech/2016/12/14/react-4-rendering-elements.html#rendering-an-element-into-the-dom):

将整个`listItems`数组包裹在`<ul>`元素中，并将其[渲染到DOM](/tech/2016/12/14/react-4-rendering-elements.html#rendering-an-element-into-the-dom)中：

```javascript
ReactDOM.render(
  <ul>{listItems}</ul>,
  document.getElementById('root')
);
```

[Try it on CodePen.](https://codepen.io/gaearon/pen/GjPyQr?editors=0011)

[打开CodePen试一试](https://codepen.io/gaearon/pen/GjPyQr?editors=0011)

This code displays a bullet list of numbers between 1 and 5.

代码渲染出1到5构成的项目列表。

### Basic List Component

Usually you would render lists inside a [component](/react/docs/components-and-props.html).

We can refactor the previous example into a component that accepts an array of `numbers` and outputs an unordered list of elements.

```javascript{3-5,7,13}
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li>{number}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

When you run this code, you'll be given a warning that a key should be provided for list items. A "key" is a special string attribute you need to include when creating lists of elements. We'll discuss why it's important in the next section.

Let's assign a `key` to our list items inside `numbers.map()` and fix the missing key issue.

```javascript{4}
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

[Try it on CodePen.](https://codepen.io/gaearon/pen/jrXYRR?editors=0011)

[打开CodePen试一试](https://codepen.io/gaearon/pen/jrXYRR?editors=0011)

## Keys

Keys help React identify which items have changed, are added, or are removed. Keys should be given to the elements inside the array to give the elements a stable identity:

```js{3}
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```

The best way to pick a key is to use a string that uniquely identifies a list item among its siblings. Most often you would use IDs from your data as keys:

```js{2}
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```

When you don't have stable IDs for rendered items, you may use the item index as a key as a last resort:

```js{2,3}
const todoItems = todos.map((todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
);
```

We don't recommend using indexes for keys if the items can reorder, as that would be slow. You may read an [in-depth explanation about why keys are necessary](/react/docs/reconciliation.html#recursing-on-children) if you're interested.

### Extracting Components with Keys

Keys only make sense in the context of the surrounding array.

For example, if you [extract](/react/docs/components-and-props.html#extracting-components) a `ListItem` component, you should keep the key on the `<ListItem />` elements in the array rather than on the root `<li>` element in the `ListItem` itself.

**Example: Incorrect Key Usage**

```javascript{4,5,14,15}
function ListItem(props) {
  const value = props.value;
  return (
    // Wrong! There is no need to specify the key here:
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Wrong! The key should have been specified here:
    <ListItem value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

**Example: Correct Key Usage**

```javascript{2,3,9,10}
function ListItem(props) {
  // Correct! There is no need to specify the key here:
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Correct! Key should be specified inside the array.
    <ListItem key={number.toString()}
              value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

[Try it on CodePen.](https://codepen.io/rthor/pen/QKzJKG?editors=0010)

[打开CodePen试一试](https://codepen.io/rthor/pen/QKzJKG?editors=0010)

A good rule of thumb is that elements inside the `map()` call need keys.

### Keys Must Only Be Unique Among Siblings

Keys used within arrays should be unique among their siblings. However they don't need to be globally unique. We can use the same keys when we produce two different arrays:

```js{2,5,11,12,19,21}
function Blog(props) {
  const sidebar = (
    <ul>
      {props.posts.map((post) =>
        <li key={post.id}>
          {post.title}
        </li>
      )}
    </ul>
  );
  const content = props.posts.map((post) =>
    <div key={post.id}>
      <h3>{post.title}</h3>
      <p>{post.content}</p>
    </div>
  );
  return (
    <div>
      {sidebar}
      <hr />
      {content}
    </div>
  );
}

const posts = [
  {id: 1, title: 'Hello World', content: 'Welcome to learning React!'},
  {id: 2, title: 'Installation', content: 'You can install React from npm.'}
];
ReactDOM.render(
  <Blog posts={posts} />,
  document.getElementById('root')
);
```

[Try it on CodePen.](https://codepen.io/gaearon/pen/NRZYGN?editors=0010)

[打开CodePen试一试](https://codepen.io/gaearon/pen/NRZYGN?editors=0010)

Keys serve as a hint to React but they don't get passed to your components. If you need the same value in your component, pass it explicitly as a prop with a different name:

```js{3,4}
const content = posts.map((post) =>
  <Post
    key={post.id}
    id={post.id}
    title={post.title} />
);
```

With the example above, the `Post` component can read `props.id`, but not `props.key`.

### Embedding map() in JSX

In the examples above we declared a separate `listItems` variable and included it in JSX:

```js{3-6}
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <ListItem key={number.toString()}
              value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

JSX allows [embedding any expressions](/tech/2016/12/14/react-3-introducing-jsx.html#embedding-expressions-in-jsx) in curly braces so we could inline the `map()` result:

```js{5-8}
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map((number) =>
        <ListItem key={number.toString()}
                  value={number} />
      )}
    </ul>
  );
}
```

[Try it on CodePen.](https://codepen.io/gaearon/pen/BLvYrB?editors=0010)

[打开CodePen试一试](https://codepen.io/gaearon/pen/BLvYrB?editors=0010)

Sometimes this results in clearer code, but this style can also be abused. Like in JavaScript, it is up to you to decide whether it is worth extracting a variable for readability. Keep in mind that if the `map()` body is too nested, it might be a good time to [extract a component](/react/docs/components-and-props.html#extracting-components).
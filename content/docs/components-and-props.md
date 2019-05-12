---
id: components-và-props
title: Components và Props
permalink: docs/components-and-props.html
redirect_from:
  - "docs/reusable-components.html"
  - "docs/reusable-components-zh-CN.html"
  - "docs/transferring-props.html"
  - "docs/transferring-props-it-IT.html"
  - "docs/transferring-props-ja-JP.html"
  - "docs/transferring-props-ko-KR.html"
  - "docs/transferring-props-zh-CN.html"
  - "tips/props-in-getInitialState-as-anti-pattern.html"
  - "tips/communicate-between-components.html"
prev: rendering-elements.html
next: state-and-lifecycle.html
---

Các component cho phép chia nhỏ UI thành các phần độc lập, có thể tái sử dụng và tư duy từng phần riêng lẻ. Trang này giới thiệu về ý tưởng của component. Bạn có thể tìm [tài liệu tham khảo component API tại đây](/docs/react-component.html).

Về khái niệm, component như các function trong Javascript. Nó chấp nhận các đầu vào tuỳ ý (gọi là props) và trả về các phần tử React mô tả những gì hiển thị ra màn hình.

## Function và Class Components {#function-and-class-components}

Cách đơn giản nhất để định nghĩa một component là viết Javascript function:

```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

Function này hợp lệ với React component bởi vì nó chấp nhận một "props" với dữ liệu được trả về một phần tử React. Chúng tôi gọi các component như vậy là "function components", vì chúng là các Javascript function theo đúng nghĩa đen.

Bạn cũng có thể dùng [ES6 class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes) để định nghĩa các component:

```js
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

Hai component trên tương đương với quan điểm trong React : "React's point of view"

Các class có một số tình năng bổ sung mà chúng ta sẽ thảo luận trong  [phần tiếp theo](/docs/state-and-lifecycle.html). Cho đến lúc đó, chúng ta sẽ sử dụng các function component cho thống nhất.

## Render một Component {#rendering-a-component}

Trước đây, chúng ta chỉ gặp các phần tử React đại diện cho DOM

```js
const element = <div />;
```

Tuy nhiên, các phần tử này cũng có thể đại diện cho các component do người dùng tự định nghĩa:

```js
const element = <Welcome name="Sara" />;
```

Khi React thấy một phần tử đại diện cho một component do người dùng định nghĩa, nó chuyển các thuộc tính JSX cho component này dưới dạng một single object. Chúng ta gọi đối tượng này là "props".

For example, this code renders "Hello, Sara":
Ví dụ: code dưới đây in ra màn hình chữ "Hello, Sara"

```js{1,5}
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

[](codepen://components-and-props/rendering-a-component)

Hãy xe điều gì sảy ra ở những dòng code trên:

1. Chúng ta gọi `ReactDOM.render()` bằng phần tử `<Welcome name = "Sara" />`.
2. React gọi component `Welcome` với `{name: 'Sara'}` là props.
3. Component `Welcome` trả về kết quả là một thẻ `<h1>Hello, Sara</h1>`.
4. React DOM tính toán, cập nhật lại để kết quả DOM khớp với`<h1>Hello, Sara</h1>`.

>**Chú ý:** Luôn bắt đầu tên của component bằng chữ in hoa.
>
>React xử lý các component bắt đầu bằng chữ thường dưới dạng thẻ DOM. Ví dụ: `<div />` thẻ div trong HTML, nhưng `Welcome` đại diện cho component và yêu cầu `Welcome` trong phạm vi scope.
>
>To learn more about the reasoning behind this convention, please read
>Để tìm hiểu về lý do sử dụng quy ước này, vui lòng đọc [JSX In Depth](/docs/jsx-in-depth.html#user-defined-components-must-be-capitalized).

## Composing Components {#composing-components}

Components can refer to other components in their output. This lets us use the same component abstraction for any level of detail. A button, a form, a dialog, a screen: in React apps, all those are commonly expressed as components.

For example, we can create an `App` component that renders `Welcome` many times:

```js{8-10}
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

[](codepen://components-and-props/composing-components)

Typically, new React apps have a single `App` component at the very top. However, if you integrate React into an existing app, you might start bottom-up with a small component like `Button` and gradually work your way to the top of the view hierarchy.

## Extracting Components {#extracting-components}

Don't be afraid to split components into smaller components.

For example, consider this `Comment` component:

```js
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

[](codepen://components-and-props/extracting-components)

It accepts `author` (an object), `text` (a string), and `date` (a date) as props, and describes a comment on a social media website.

This component can be tricky to change because of all the nesting, and it is also hard to reuse individual parts of it. Let's extract a few components from it.

First, we will extract `Avatar`:

```js{3-6}
function Avatar(props) {
  return (
    <img className="Avatar"
      src={props.user.avatarUrl}
      alt={props.user.name}
    />
  );
}
```

The `Avatar` doesn't need to know that it is being rendered inside a `Comment`. This is why we have given its prop a more generic name: `user` rather than `author`.

We recommend naming props from the component's own point of view rather than the context in which it is being used.

We can now simplify `Comment` a tiny bit:

```js{5}
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <Avatar user={props.author} />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

Next, we will extract a `UserInfo` component that renders an `Avatar` next to the user's name:

```js{3-8}
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">
        {props.user.name}
      </div>
    </div>
  );
}
```

This lets us simplify `Comment` even further:

```js{4}
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

[](codepen://components-and-props/extracting-components-continued)

Extracting components might seem like grunt work at first, but having a palette of reusable components pays off in larger apps. A good rule of thumb is that if a part of your UI is used several times (`Button`, `Panel`, `Avatar`), or is complex enough on its own (`App`, `FeedStory`, `Comment`), it is a good candidate to be a reusable component.

## Props are Read-Only {#props-are-read-only}

Whether you declare a component [as a function or a class](#function-and-class-components), it must never modify its own props. Consider this `sum` function:

```js
function sum(a, b) {
  return a + b;
}
```

Such functions are called ["pure"](https://en.wikipedia.org/wiki/Pure_function) because they do not attempt to change their inputs, and always return the same result for the same inputs.

In contrast, this function is impure because it changes its own input:

```js
function withdraw(account, amount) {
  account.total -= amount;
}
```

React is pretty flexible but it has a single strict rule:

**All React components must act like pure functions with respect to their props.**

Of course, application UIs are dynamic and change over time. In the [next section](/docs/state-and-lifecycle.html), we will introduce a new concept of "state". State allows React components to change their output over time in response to user actions, network responses, and anything else, without violating this rule.

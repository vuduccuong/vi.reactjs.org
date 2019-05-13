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

Ví dụ: code dưới đây in ra màn hình chữ "Hello, Sara":

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

Hãy xem điều gì sảy ra ở những dòng code trên:

1. Chúng ta gọi `ReactDOM.render()` bằng phần tử `<Welcome name = "Sara" />`.
2. React gọi component `Welcome` với `{name: 'Sara'}` là props.
3. Component `Welcome` trả về kết quả là một thẻ `<h1>Hello, Sara</h1>`.
4. React DOM tính toán, cập nhật lại để kết quả DOM khớp với`<h1>Hello, Sara</h1>`.

>**Chú ý:** Luôn bắt đầu tên của các component bằng chữ in hoa.
>
>React xử lý các component bắt đầu bằng chữ thường dưới dạng thẻ DOM. Ví dụ: `<div />` thẻ div trong HTML, nhưng `Welcome` đại diện cho component và yêu cầu `Welcome` trong phạm vi scope.
>
>Để tìm hiểu về lý do sử dụng quy ước này, vui lòng đọc [JSX In Depth](/docs/jsx-in-depth.html#user-defined-components-must-be-capitalized).

## Composing Components {#composing-components}

Các component có thể tham khảo(refer) các component khác trong out put của nó. Điều này cho phép chúng ta sử dụng abstraction cho mọi mức độ chi tiết nào. Button, Form, dialog: Trong các ứng dụng React, tát cả các thứ đó đều có thể được thể hiện bởi component.

Ví dụ: chúng ta có thể tạo component `App` và render ra nhiều `Welcome` một lúc:

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

Thông thường, các ứng dụng React mới sẽ có một `App` component ở trên cùng. Tuy nhiên, nếu bạn đang đang làm ứng dụng React, bạn hoàn toàn có thể bắt đầu từ những thứ nhỏ nhất ví dụ như `Button` component và dần dần thêm các component lớn hơn.

## Extracting Components {#extracting-components}

Đừng ngại việc chia nhỏ các component.

Ví dụ xem `Comment` component dưới đây:

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

Nó nhận `author`(kiểu object), `text`(kiểu string) và `date` (kiểu date) là props và mô tả comment trên một trang mạng xã hội.

Component này có thể rất khó thay đổi, vì tất cả đều lồng vào nhau, và cũng khó để tái sử dụng các phần nhở trong nó. Bây giờ, thử xem một vài đề xuất này:

Đầu tiên, chúng ta xử lý với `Avatar`:

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

`Avatar` không cần quan tâm rằng nó sẽ hiển thị trong `Comment`. Đó là lý do tại sao chúng ta đặt props cho nó một tên chung chung: `user` thay vì dùng `author`.

Chúng tôi khuyên bạn nên đặt tên props theo component chứ không phải là bối cảnh sử dụng.

Chúng ta đã thấy, `Comment` đã đơn giản hơn, nhỏ hơn một chút:

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

Tiếp theo là xử lý `UserInfo` component để render ra `Avatar` bên cạnh tên người dùng:

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

Bây giờ `Comment` component sẽ trông như thế này:

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

Việc xử lý những component ban đầu trông giống những việc đơn giản, nhưng việc có một danh sách các component có thể tái sử dụng trong các dự án lớn là một điều cần phải nghiên cứu kĩ. Một nguyên tắc nhỏ là nếu một phần UI đơn giản được sử dụng nhiều lần (`Button`, `Panel`, `Avatar`), hoặc phức tạp như (`App`, `FeedStory`, `Comment`), thì việc biến chúng làm component tái sử dụng là một điều nên làm.

## Props are Read-Only {#props-are-read-only}

Cho dù bạn khai báo một component là [một function hay một class](#function-and-class-components), thì nó không bao giờ phải sửa đổi props của nó. Hãy xem `function sum` sau đây:

```js
function sum(a, b) {
  return a + b;
}
```

Các function như thế này được gọi là  ["pure"(nguyên chất)](https://en.wikipedia.org/wiki/Pure_function) bởi vì chúng không có sự thay đổi đầu vào và luôn trả về cùng một kết quả cho cùng một đầu vào.

Ngược lại, function này không "nguyên chất" vì chúng thay đổi đầu vào của chính nó:

```js
function withdraw(account, amount) {
  account.total -= amount;
}
```

React rất linh hoạt, nhưng nó có một quy tắc:

**Tất cả các React component phải hoạt động như các function "nguyên chất" đối với các props của nó** 

tất nhiên, ứng dụng UI rất linh hoạt, và thay đổi theo thời gian, Trong [phần tiếp theo](/docs/state-and-lifecycle.html), chúng tôi giới thiệu một khái niệm mới về "state". State cho phép các React component thay đổi đầu ra của chúng trong thời gian để trả lời hành động của người dùng, phản hồi của network và bất cứ điều gì khác mà không vi phạm quy tắc trên

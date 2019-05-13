---
id: forms
title: Forms
permalink: docs/forms.html
prev: lists-and-keys.html
next: lifting-state-up.html
redirect_from:
  - "tips/controlled-input-null-value.html"
  - "docs/forms-zh-CN.html"
---

Trong React, các phần tử HTML form hoạt động hơi khác một chút so với các phần tử DOM khác. Vì các form element(phần tử) sẽ giữ một số internal state bên trong nó. Ví dụ: Form này trong HTML đơn giản chỉ là chấp nhận một trường `name`

```html
<form>
  <label>
    Name:
    <input type="text" name="name" />
  </label>
  <input type="submit" value="Submit" />
</form>
```

Form này sẽ mặc định đưa trình duyệt đến một trang mới khi người dùng click vào button submit. Nếu bạn muốn hành động này trong React, nó sẽ hoạt động. Nhưng hầy hết các trường hợp, nó rất tiện lợi khi có một function Javascript xử lý việc Form có quyền truy cập vào dữ liệu mà người dùng đã nhập vào form. Cách chuẩn nhất để làm điều này là dùng một kỹ thuật được gọi là "controlled components".

## Controlled Components {#controlled-components}

Trong HTML, các form element như `<input>`, `<textarea>` và `<select>` thường duy trì state của chúng và được cập nhật dựa vào dữ liệu mà người dùng nhập vào. Trong React, trạng tháicó thể thay đổi được lưu trong state của component, và chỉ cập nhật với [`setState()`](/docs/react-component.html#setstate).

Chúng ta có thể kết hợp cả hai cách làm cho React state trở thành là một thứ duy nhất "single source of truth". Sau đó, React component render một formvaf kiểm soát những gì người dùng nhập vào. Một input form element có giá trị được điều khiển bởi React được gọi là "controlled component".

Ví dụ: nếu chúng ta muốn ví dụ trước ghi lại name khi submit form, chúng ta viết form dưới dạng một controlled conponent:

```javascript{4,10-12,24}
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/VmmPgp?editors=0010)

Vì thuộc tính `value` được đặt trên form element, giá trị được hiển thị sẽ luôn là `this.state.value`, làm cho React state trở thành "source of truth". Vì `handleChange` chạy mỗi lần nhập giá trị vào `input` và nó cập nhật React state, giá trị được hiển thị sẽ cập nhật giống như những gì người dùng nhập.

Đối với controlled component, mọi sự thay đổi state đều có các function xử lý (handler) liên quan. Điều này làm cho nó đơn giản để sửa đổi hoặc xác nhận giá trị người dùng nhập. Ví dụ: nếu chúng ta muốn biến tất các các chữ cái nhập vào thành chữ in hoa, thì chúng ta có thể viết `handleChange` như sau:

```javascript{2}
handleChange(event) {
  this.setState({value: event.target.value.toUpperCase()});
}
```

## Thẻ textarea {#the-textarea-tag}

Trong HTML thẻ `<textarea>` xác định text của nó theo phần tử con:

```html
<textarea>
  Xin chào...tóc đẹp không ạ?
</textarea>
```

Trong React, thẻ `<textarea>` sử dụng thuộc tính `value` để thay thế. Theo cách này, một form sử dụng `<textarea>` có thể được viết rất giống với form sử dụng single-line input:

```javascript{4-6,12-14,26}
class EssayForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: 'Please write an essay about your favorite DOM element.'
    };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('An essay was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Essay:
          <textarea value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

Lưu ý rằng, `this.state.value` được khởi tạo trong contructoer, để textarea bắt đầu với một số text ở trong nó.

## Thẻ select {#the-select-tag}

Trong HTML, thẻ `<select>` tạo một drop-down list. Ví dụ: HTML dưới đây tạo một danh sách trái cây sử dụng drop-down:

```html
<select>
  <option value="grapefruit">Grapefruit</option>
  <option value="lime">Lime</option>
  <option selected value="coconut">Coconut</option>
  <option value="mango">Mango</option>
</select>
```

Lưu ý rằng, ban đầu giá trị được hiển thị sẽ là Coconut, bởi vì nó được khai báo ban đầu. React, thay vì sử dụng `selected`, nó sẽ sử dụng biến `value` trong thẻ `select`. Điều này thuận tiện hơn trong một controlled component vì bạn chỉ cần cập nhật nó ở một nơi. Ví dụ:

```javascript{4,10-12,24}
class FlavorForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: 'coconut'};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('Your favorite flavor is: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Pick your favorite flavor:
          <select value={this.state.value} onChange={this.handleChange}>
            <option value="grapefruit">Grapefruit</option>
            <option value="lime">Lime</option>
            <option value="coconut">Coconut</option>
            <option value="mango">Mango</option>
          </select>
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/JbbEzX?editors=0010)

Nhìn chung, điều đó làm cho `<input type="text">`, `<textarea>`, và `<select>` đều hoạt động giống nhau - tất cả đều nhận thuộc tính `value` mà bạn có thể dùng để triển khai controlled component.

> Chú thích
>
>Bạn có thể chuyển một array vào thuộc tính `value`, cho phép bạn chọn nhiều tuỳ chọn trong một thẻ `<select>`:
>
>```js
><select multiple={true} value={['B', 'C']}>
>```

## Thẻ input file {#the-file-input-tag}

Trong HTML, một thẻ `<input type="file">` cho phép người dùng chọn một hoặc nhiều tệp từ bộ nhớ để tải lên server hoặc được Javascript sử lý thông qua [File API](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications).

```html
<input type="file" />
```

Bởi vì nó là giá trị "read-only", nên nó là **uncontrolled** component trong React. Nó sẽ được nói đến trong [phần cuối](/docs/uncontrolled-components.html#the-file-input-tag).

## Xử lý nhiều Inputs {#handling-multiple-inputs}

Khi bạn cần xử lý nhiều thành phần controlled `input`, bạn có thể thêm thuộc tính `name` cho từng component và để handle function chọn việc cần làm dựa trên giá trị của `event.target.name`.

Ví dụ:

```javascript{15,18,28,37}
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }

  render() {
    return (
      <form>
        <label>
          Is going:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange} />
        </label>
        <br />
        <label>
          Number of guests:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```

[**Try it on CodePen**](https://codepen.io/gaearon/pen/wgedvV?editors=0010)

Lưu ý cách chúng ta sử dụng [cú pháp ES6](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names) để cập nhật state tương đương với tên ban đầu đã cho:

```js{2}
this.setState({
  [name]: value
});
```

Nó tương đương với code ES5 này:

```js{2}
var partialState = {};
partialState[name] = value;
this.setState(partialState);
```

Ngoài ra, vì `setState()` tự động [hợp nhất một phần state vào state hiện tại](/docs/state-and-lifecycle.html#state-updates-are-merged), chúng ta chỉ cần gọi nó với các phần đã thay đổi.

## Giá trị Null trong Controlled Input {#controlled-input-null-value}

Việc chỉ định giá trị prop trên controlled component ngăn người dùng thay đổi đầu vào trừ khi bạn muốn như vậy. Nếu bạn đã chỉ định `value` nhưng đầu vào vẫn có thể chỉnh sửa, bạn có thể đã vô tình đặt `value` thành `undefined` hoặc `null`. 

Đoạn code dưới đây chứng minh điều này (input bị khoá lúc đầu nhưng có thể chỉnh sửa sau một thời gian delay ngắn.)

```javascript
ReactDOM.render(<input value="hi" />, mountNode);

setTimeout(function() {
  ReactDOM.render(<input value={null} />, mountNode);
}, 1000);

```

## Alternatives to Controlled Components {#alternatives-to-controlled-components}

It can sometimes be tedious to use controlled components, because you need to write an event handler for every way your data can change and pipe all of the input state through a React component. This can become particularly annoying when you are converting a preexisting codebase to React, or integrating a React application with a non-React library. In these situations, you might want to check out [uncontrolled components](/docs/uncontrolled-components.html), an alternative technique for implementing input forms.

## Fully-Fledged Solutions {#fully-fledged-solutions}

If you're looking for a complete solution including validation, keeping track of the visited fields, and handling form submission, [Formik](https://jaredpalmer.com/formik) is one of the popular choices. However, it is built on the same principles of controlled components and managing state — so don't neglect to learn them.

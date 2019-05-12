---
id: hooks-state
title: Using the Effect Hook
permalink: docs/hooks-effect.html
next: hooks-rules.html
prev: hooks-state.html
---

*Hooks* là một tính năng mới được thêm vào từ phiên bản 16.8. Cho phép sử dụng state và các tính năng React mà không cần viết class

*Effect Hook* cho phép bạn xử lý các side effects trong function components:

```js{1,6-10}
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

Đoạn mã trên được lấy từ [ví dụ của phần trước](/docs/hooks-state.html), nhưng đã được thêm tính năng: chúng tôi đặt lại tiêu đề trang một tin nhắn khi click vào button.

Lấy dữ liệu và hiển thị, thiết lập đăng ký, và thay đổi DOM trong React components là tất cả các ví dụ về thực thi "side effects" (hoặc đơn giản là "effects"), cho dù bạn có hay không sử dụng để gọi các hoạt động này trong component hay không.

>Mẹo nhỏ
>
>Nếu bạn quen thuộc với React class lifecycle methods, bạn có thể nghĩ đến `useEffect` Hook kết hơp `componentDidMount`, `componentDidUpdate` và `componentWillUnmount`.

Có hai loại side effect phổ biến trong React components: Những loại không có yêu cầu dọn dẹp, và những loại yêu cầu. Hãy cùng xem chi tiết sự khác biệt này nào...

## Effects không dọn dẹp {#effects-without-cleanup}

Đôi khi chúng ta muốn **chạy một số mã bổ sung sau khi React cập nhật lại DOM**  khi có request, các DOM sẽ thay đổi và ghi lại log trong Network là ví dụ phổ biến về effrcts không yêu cầu dọn dẹp. Chúng ta có thể nói rằng có thể chạy chúng mà không cần quan tâm chúng. Và giờ hãy so sánh các class và Hooks thể hiện điều này.
### Ví dụ sử dụng Classes {#example-using-classes}

Trong React class components, phương thức `render` không gây ra side effects. Nó quá sớm để thực hiện, mà chúng tôi muốn thực hiện các effect *sau khi* React đã cập nhật lại DOM.

Đó là lý do tại sao trong các React class, chúng tôi lại đặt side effect vào trong `componentDidMount` và `componentDidUpdate`. Quay trở lại ví dụ, đây là React counter class component cập nhật lại title ngay sau khi React thay đổi DOM:

```js{9-15}
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

Lưu ý **chúng ta tạo hai dòng lệnh giống nhau giữa hai phương thức lifecycle methods trong class**

Đó là do trong nhiều trường hợp, chúng ta muốn thực hiện nhũng side effect giống nhau bất kể các components được gắn hoặc cập nhật. Về mặt khái niệm, chúng tôi muốn nó thực hiện sau mỗi lần `render` - nhưng React class components không có các phương thức như thế. Chúng ta có thể tạo ra 2 phương thức riêng biệt nhưng sẽ vẫn gọi nó ở 2 nơi.

Bây giờ hãy xem chúng ta có hãy xem `useEffect` Hook có thể làm được gì nào.

### Ví dụ sử dụng Hooks {#example-using-hooks}

Chúng ta đã thấy ví dụ này trên đầu trang, nhưng hãy phân tích lại nó:

```js{1,6-8}
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

**Tác dụng của `useEffect` là gì?** Bằng cách sử dụng Hook, chúng ta nói với React rằng component của chúng ta cần làm gì sau khi `render`. React sẽ ghi nhớ những lần mà function của chúng ta đã vượt qua(chúng tôi gọi nó là "effect"), gọi nó sau khi thực hiện cập nhật lại DOM. Trong effect này, chúng ta gán lại giá trị cho document.title, nhưng cũng có thể thực hiện nạp dữ liệu hoặc gọi một số API.

**Tại sao ``useEffect` được gọi bên trong một component** Đặt `useEffect` bên trong component cho phép chúng ta truy cập vào biến `count`(hoặc bất kỳ props nào) ngay từ effect. Chúng ta không cần một API đặc biệt nào để đọc nó, vì nó đã có trong function scope. Hook bao quanh Javascript và tránh đem vào những API React đặc biệt bởi vì Javascript đã cung cấp những giải pháp rồi.

**`useEffect` có chạy sau mỗi lần render hay không?**  Có, và nó mặc chạy sau lần render đầu tiên, và sau mỗi lần cập nhật. (Chúng ta sẽ nói sau về [cách cấu hình lại chúng](#tip-optimizing-performance-by-skipping-effects).) Thay vì suy nghĩ về các khía cạnh của việc "mounting" và "updating", bạn có thể dễ dàng hơn khi nghĩ rằng các effect sảy ra "sau khi render". React đảm bảo rằng DOM đã được cập nhật vào thời điểm nó chạy các effect 

### Chi tiết {#detailed-explanation}

Bây giờ chúng ta đã hiểu hơn về effect và những dòng code này: 

```js
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
```

Chúng ta khai báo biến `count` và nói với React rằng cần sử dụng một effect. Chúng ta chuyển chức năng đó cho `useEfect` Hook. function này chứa effect. Bên trong effect, chúng ta gán giá trị cho document title bằng API của trình duyệt `document.title`. Chúng ta có thể đọc biến `count` mới nhất trong effect bởi nó trong phạm vi scope của function. Khi React renders các component của chúng ta, nó sẽ ghi nhớ các effect mà chúng ta sử dụng, và sau đó chạy những effect này sau khi cập nhật lại DOM. Điều này diễn ra ở mọi lần render, ngay cả ở lần render đầu tiên

Những developer Javascript có kinh nghiệm có thể thấy rằng function truyền cho `useEffect` sẽ khác nhau sau mỗi lần render. Điều này là cố ý. Trong thực tế, đây là những gì cho phép chúng ta đọc giá trị biến `count` từ trong effect mà không lo nó không cập nhật lại giá trị mỗi khi chúng ta render lại, chúng tôi ghi lại cho mỗi effect khác nhau, thay thế chúng cho các effect trước đó. Nói cách khác, điều này làm cho các effect hoạt động giống như một phần của kết quả render - mỗi effect thuộc về một render cụ thể. Chúng ta sẽ thấy rõ hơn tại sao điều này hữu ích sau khi đọc đến [phần này](#explanation-why-effects-run-on-each-update).

>Mẹo nhỏ
>
>Không giống như `componentDidMount` hoặc `componentDidUpdate`, effect sẽ lên kế hoặc với `useEffect` không chặn trình duyệt khi màn hình đang cập nhật. Điều này làm cho cảm giác chương trình chạy nhanh hơn. Phần lớn, các effect không cần chạy đồng bộ. Trong một số trường hợp(chẳng hạn như đo bố cục), có một `useLayoutEffect` Hook đặc biệt với API giống hệt với `useEffect`.

## Effects yêu cầu làm sạch {#effects-with-cleanup}

Trước đó, chúng ta đã tìm hiểu effect không yêu cầu làm sạch, Tuy nhiên một số effect lại yêu cầu. Ví dụ, **chúng ta muốn thiết lập đăng ký** một số nguồn dữ liệu bên ngoài. Trong trường hợp đó, điều quan trọng là phải làm sạch để không bị tràn bộ nhớ, Hãy so sánh các cách chúng ta có thể làm với class và Hooks.


### Sử dụng Classes {#example-using-classes-1}

Trong React class, thông thường, bạn sẽ thiết lập đăng ký trong `componentDidMount` và làm sạch nó trong `componentWillUnmount`.  Ví dụ, chúng ta có module ChatAPI lấy trạng thái bạn bè online. Ở đây, chúng ta có thể thiết lập, và hiển thị trạng thái online bằng class:

```js{8-26}
class FriendStatus extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }

  render() {
    if (this.state.isOnline === null) {
      return 'Loading...';
    }
    return this.state.isOnline ? 'Online' : 'Offline';
  }
}
```

Lưu ý `componentDidMount` và `componentWillUnmount` cần đối ngược nhau. Các phương thức lifecycle buộc chúng ta phải phân tách logic này, mặc dù nội dung trong cả hai đều là một effect. 

>Chú ý
>
>Bạn đọc tinh ý có thể nhận thấy rằng ví dụ chính xác cũng cần thêm `componentDidUpdate`. Chúng tôi đã loại bỏ nó, nhưng sẽ quay lại nó trong [phần sau](#explanation-why-effects-run-on-each-update)

### Sử dụng Hooks {#example-using-hooks-1}

Hãy xem chúng ta có thể làm được gì với Hooks.

Bạn có nghĩ rằng chúng ta cần thiết lập một effect riêng để thực hiện việc làm sạch. Nhưng code để thêm và xoá đăng ký phải liên quan chặt chẽ đvì thế `useEffect` được thết kế để thực hiện chúng cùng nhau. Nếu effect của bạn trả về một function, React sẽ chạy nó khi đến lúc dọn dẹp:

```js{6-16}
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

**Tại sao lại return function trong effect?** Đây là cơ chế làm sạch cho các effect. Mỗi effect có thể trả về một chức năng dọn dẹp sau nó. Điều này cho phép chúng ta giữ logic để thêm và xoá đăng ký gần nhau. Nó là nhóm các effect tương tự nhau

**Chính xác thì khi nào React dọn dẹp một effect?** React thực hiện việc làm sạch khi component ngắt kết nối. Tuy nhiên, như chúng ta đã tìm hiểu trước đó, các effect chạy trong mọi lần render. Đây là lý do tại sao React cũng làm sạch các effect từ lần render trước đó khi chạy effect vào lần tiếp theo. Chúng ta sẽ thảo luận về lý do [tại sao điều này tránh gặp bugs](#explanation-why-effects-run-on-each-update) và [làm thế nào để tăng hiệu suất](#tip-optimizing-performance-by-skipping-effects) ngay sau đây.

>Chú thích
>
>Chúng ta không phải return tên của một function từ effect. Chúng tôi đã gọi nó là `dọn dẹp` để làm rõ mục đích của nó, nhưng bạn có thể return một arrow function hoặc gọi nó bằng một cái tên khác.

## Tóm tắt {#recap}

Chúng ta đã tìm hiểu được rằng `useEffect` cho phép chúng ta thể hiện các effect khác nhau sau mỗi lần render. Một số effect có thể yêu cầu làm sạch để chúng return một function:

```js
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
```

Một số effect khác có thể không có giai đoạn dọn dẹp và không cần return gì.

```js
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
```

Trong 2 trường hợp Effect Hook sử dụng với một API.  

-------------

**Nếu bạn cảm thấy mình hiểu được cách thức hoạt động của Effect Hook, bạn có thể chuyể sang [trang tiếp theo, nói về các quy tắc trong Hooks](/docs/hooks-rules.html) ngay bây giờ**

-------------

## Một số mẹo cho việc sử dụng các Effect {#tips-for-using-effects}

Chúng ta sẽ tìm hiểu tiếp với một cái nhìn sâu hơn về một số khía cạnh của `useEffect` mà một số người đã có kinh nghiệm quan tâm. Bạn không cần phải bắt buộc tìm hiểu sâu về chúng ngay bây giờ, mà có thể quay lại trang này để tìm hiểu sâu hơn về Effect Hook vào bất cứ lúc nào.

### Mẹo: Sử dụng nhiều Effect riêng biệt {#tip-use-multiple-effects-to-separate-concerns}

Một trong những vấn đề chúng ta đã [trình bày](/docs/hooks-intro.html#complex-components-become-hard-to-understand) trong Hooks là các lifecycle của class thường chứa logic không liên quan, nhưng logic liên quan bị chia thành nhiều phương thức. Đây là một component kết hợp đếm và lấy trạng thái bạn bè online từ các ví dụ trước:

```js
class FriendStatusWithCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0, isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }
  // ...
```

Lưu ý, thời điểm để gán `document.title` là khoảng thời gian nằm giữa `componentDidMount` và `componentDidUpdate`. logic đăng ký cũng được nằm trong khoảng thời gian giữa `componentDidMount` và `componentWillUnmount`. Và `componentDidMount` sảy ra ở cả 2 nhiệm vụ.

Vậy, bằng cách nào mà Hooks có thể giải quyết vấn đề này? Giống như việc [bạn sử dụng State Hook nhiều lần](/docs/hooks-state.html#tip-using-multiple-state-variables), bạn cũng có thể sử dụng một số effect. Điều này cho phép chúng ta tách những thứ không liên quan đến nhau, thành những effect khác nhau:

```js{3,8}
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
  // ...
}
```

**Hooks cho phép chúng ta tách code theo chức năng của nó** thay vì đặt theo tên lifecycle method. React sẽ áp dụng mọi effect được sử dụng bởi component theo một thứ tự được định sẵn

### Giải thích: Tại sao Effect chạy sau mỗi lần cập nhật {#explanation-why-effects-run-on-each-update}

Nếu bạn sử dụng class, bạn có thể tự hỏi tại sao giai đoạn dọn dẹp effect lại sảy ra sau mỗi lần render lại, và không chi một lần trong khi unmounting. Hãy cùng xem một ví dụ thực tế để xem tại sao thiết kế này giúp chúng ta tạo ra các component ít lỗi hơn.

[Trước đó, ở trang này](#example-using-classes-1), chúng ta đã được biết về ví dụ `FriendStatus component` để hiển thị danh sách bạn bè có online hay không. Class của chúng ta đọc `friend.id` từ `this.props`, đăng ký trạng thái online của bạn bè sau khi component mounts, và huỷ đăng ký trong khi unmounting:

```js
  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
```

**Nhưng điều gì sảy ra nếu `friend` props thay đổi** trong khi component còn trên màn hình? Component của chúng ta sẽ tiếp tục hiển thị trạng thái trực tuyến của một người bạn khác. Đây là một bug. Chúng ta sẽ gây ra tràn bộ nhớ hoặc gặp lỗi khi unmounting vì huỷ đăng ký sẽ sử dụng sai friend ID.

Trong class component, chúng ta sẽ cần thêm `componentDidUpdate` để xử lý trường hợp này:

```js{8-19}
  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate(prevProps) {
    // Unsubscribe from the previous friend.id
    ChatAPI.unsubscribeFromFriendStatus(
      prevProps.friend.id,
      this.handleStatusChange
    );
    // Subscribe to the next friend.id
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
```

Quên sử lý `componentDidUpdate` đúng cách sẽ phát sinh lỗi trong các ứng dụng React.

Nào, bây giờ hãy xem phiên bản component sử dụng Hooks:

```js
function FriendStatus(props) {
  // ...
  useEffect(() => {
    // ...
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
```

Nó không phải chịu lỗi.(Nhưng chúng tôi đã không thực hiện bất kì thay đổi nào với nó.)

Không có code đặc biệt để xử lý các lần cập nhật, bởi vì `useEffect` xử lý chúng *theo mặc định*. Nó dọn dẹp các effect trước khi áp dụng các effect tiếp theo. Để minh hoạ cho điều này, đây là một chuỗi các lần đăng ký và huỷ đăng ký mà component này có thể tạo ra theo thời gian:

```js
// Mount with { friend: { id: 100 } } props
ChatAPI.subscribeToFriendStatus(100, handleStatusChange);     // Run first effect

// Update with { friend: { id: 200 } } props
ChatAPI.unsubscribeFromFriendStatus(100, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(200, handleStatusChange);     // Run next effect

// Update with { friend: { id: 300 } } props
ChatAPI.unsubscribeFromFriendStatus(200, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(300, handleStatusChange);     // Run next effect

// Unmount
ChatAPI.unsubscribeFromFriendStatus(300, handleStatusChange); // Clean up last effect
```

This behavior ensures consistency by default and prevents bugs that are common in class components due to missing update logic.
Hành vi này đảm bảo tính nhất quán theo mặc định và ngăn ngừa các lỗi phổ biến trong các class component do thiếu logic cập nhật.

### Mẹo: Tối ưu hoá hiệu suất bằng cách bỏ qua Effects {#tip-optimizing-performance-by-skipping-effects}

Trong một số trường hợp, việc dọn dẹp hoặc áp dụng effect sau mỗi lần render có thể tạo ra các vấn đề về hiệu năng. Trong các class component, chúng ta có thể giải quyết việc này bằng cách viết bổ sung `prevProps` hoặc `prevState` bên trong `componentDidUpdate`:

```js
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

Yêu cầu này đủ phổ biến để được tích hợp vào `useEffect` Hook API. Bạn có thể yêu cầu React bỏ qua việc áp dụng effect nếu một số giá trị nhất định đã thay đổi giữa những lần render lại. Để làm như vậy, truyền vào một mảng dưới dạng đối số là lựa chọn thứ hai cho `useEffect`:

```js{3}
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // Only re-run the effect if count changes
```

Trong ví dụ trên, chúng ta đặt `[count]` làm đối số thứ hai. Điều đó có nghĩa là gì? Nếu `count` là `5`, và sau đó, component sẽ render lại với `count` vẫn bằng `5`, React sẽ so sánh `[5]` từ lần render trước đó và `[5]` từ lần render tiếp theo. Bởi vì tất các các item trong mảng đều giống nhau(`5 === 5`), React sẽ bỏ qua effect đó. Đó là tối ưu hoá `useEffect`.

Khi chúng ta render với `count` đã được cập nhật là `6`, React sẽ so sánh các item trong mảng (`[5]`) từ lần render trước đó với các item trong mảng (`[6]`) từ lần render tiếp theo. Lần này, React sẽ áp dụng  lại effect bởi vì `5 !== 6`. Nếu có nhiều item trong mảng, React sẽ chạy lại effect ngay cả khi chỉ một số chúng là khác nhau.

Điều này cũng hoạt động cho các effect yêu cầu dọn dẹp:

```js{10}
useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
}, [props.friend.id]); // Only re-subscribe if props.friend.id changes
```

Trong tương lai, đối số thứ hai có thể được thêm tự động bằng cách chuyển đổi build-time.

>Chú thích
>
>Nếu bạn sử dụng tối ưu hoá này, hãy đảm bảo mảng gồm **tất cả các giá trị từ component scope (chẳng hạn như props và state) thay đổi theo thời gian và được sử dụng vởi effect**. Nếu không code của bạn sẽ tham chiếu các giá trị cũ từ các lần render trước đó. Tìm hiểu thêm về [cách xử lý các function](/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) và [phải làm gì khi mảng thay đổi quá thường xuyên](/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often).
>
>Nếu bạn muốn chạy một effect và dọn dẹp nó chỉ một lần (trên mount và unmount), bạn có thể chuyển một mảng rỗng (`[]`) làm đối số thứ hai. Điều này cho React biết rằng, effect của bạn không phụ thuộc vào bất kỳ giá trị nào từ props hoặc state, do đó mà không bao giờ cần phải chạy lại. Điều này được xử lý như một trường hợp đặc biệt - nó diễn ra trực tiếp từ cách mảng đầu vào luôn hoạt động.
>
>Nếu bạn muốn sử dụng một mảng rỗng (`[]`), props và state trong effect sẽ luôn có các giá trị ban đầu của chúng. Trong khi chuyển `[]` làm đối số thứ hai gần hơn với mô hình quen thuộc `componentDidMount` và `componentWillUnmount` thường có các [giải pháp ](/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often) [tốt hơn](/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) để tránh các effect chạy lại quá thường xuyên. Ngoài ra, đừng quên rằng React trì hoãn việc sử dụng `useEffect` cho đến khi trình duyệt đã vẽ xong, việc làm thêm sẽ gặp ít vấn đề hơn.
>
>Chúng tôi khuyên bạn nên sử dụng quy tắc [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) như một phần package[`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation). Nó cảnh báo khi các phụ thuộc được chỉ định không chính xác và đề nghị sửa chữa.

## Bước tiếp theo {#next-steps}

Xin chúc mừng, đây là một trang khá dài, nhưng hy vọng các câu hỏi của bạn về Effect Hook đã được trả lời. Bạn đã học được cả State Hook và Effect Hook, và có thể làm được rất nhiều thứ với cả hai. Chúng bao gồm hầu hết các trường hợp sử dụng class -- mà ở đó chúng ta không thể làm, bạn có thể tìm [thêm các Hooks hữu ích khác](/docs/hooks-reference.html).

Chúng ta cũng bắt đầu thấy cách Hook giải quyết các vấn đề được nêu trong [Motivation](/docs/hooks-intro.html#motivation). Chúng ta có thể thấy cách dọn dẹp effect tránh sự trùng lặp trong `componentDidUpdate` và `componentWillUnmount`, khiến code gọn gàng hơn và giúp tránh lỗi. Chúng ta đã thấy cách mà chúng tôi có thể phân tách các effect theo mục đích của chúng, đó là điều mà không thể làm được với khi sử dụng class.

Tại thời điểm này, bạn có thể đặt câu hỏi về cách thức hoạt động của Hook. Làm thế nào React có thể gọi chính xác `useState` nào tương ứng với biến state giữa mỗi lần render lại? Làm thế nào để React lựa chọn effect trước và sau khi update? **Trong trang tiếp theo, chúng ta sẽ học về [Quy tắc trong Hook](/docs/hooks-rules.html) -- Nó rất cần thiết để Hook có thể hoạt động.**

---
id: hooks-custom
title: Building Your Own Hooks
permalink: docs/hooks-custom.html
next: hooks-reference.html
prev: hooks-rules.html
---

*Hooks* là một tính năng mới được thêm vào từ phiên bản 16.8. Cho phép sử dụng state và các tính năng React mà không cần viết class

Xây dựng Hook riêng của bạn, cho phép bạn trích xuất các component có thể tái sử dụng.

Khi chúng ta tìm hiểu cách [sử dụng Effect Hook](/docs/hooks-effect.html#example-using-hooks-1), chúng ta đã thấy component này từ một ứng dụng chat hiển thị danh sách bạn bè onlie hay offline:

```js{4-15}
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
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

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

Bây giờ, có thể nói rằng ứng dụng chat của chúng ta đã có một danh sách liên lạc và chúng ta muốn hiển thị tên người dùng trực tuyến với màu xanh lục. Chúng ta có thể sao chép và dán code ở trên vào component `FriendListItem` nhưng nó không phải là một ý tưởng tốt:

```js{4-15}
import React, { useState, useEffect } from 'react';

function FriendListItem(props) {
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

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

Thay vào đó, chúng tôi muốn sử dụng logic này qua lại giữa `FriendStatus` và `FriendListItem`.

Trong React, chúng ta có hai cách phổ biến để chia sẻ logic stateful giữa các component: [render props](/docs/render-props.html) và [higher-order components](/docs/higher-order-components.html). Chúng ta sẽ xem xét cách mà Hooks giải quyết nhiều vấn đề tương tự mà không buộc bạn phải thêm nhiều component vào dự án.

## Trích xuất một Hook tuỳ chỉnh{#extracting-a-custom-hook}

Khi chúng ta muốn chia sẻ logic giữa hai Function trong Javascript, chúng ta trích xuất nó sang một function thứ ba, cả component và Hook đều là function, vì vậy điều này cũng hoạt động với chúng.

**Một Hook tuỳ chình là một function Javascript có tên bắt đầu bằng "`use`" và có thể gọi các Hook khác.** Ví dụ: `useFriendStatus` bên dưới là một Hook tuỳ chỉnh của chúng ta:

```js{3}
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

Không có gì mới bên trong nó - logic được coppy từ các component trên. Giống như trong một component, đảm bảo chỉ gọi các Hook khác vô điều kiện ở top level của Hook tuỳ chỉnh.

Không giống như một React component, một Hook tuỳ chỉnh không cần phải có một ký hiệu cụ thể. Chúng ta có thể quyết định những gì nó cần là takes as arguments, and what, if anything, it should return. Nói cách khác, nó giống như môt function bình thường. Tên của nó luôn phải bắt đầu bằng `use` để bạn có thể biết trong nháy mắt rằng [các quy tắc của Hook](/docs/hooks-rules.html) áp dụng cho nó.

Mục đích của `useFriendStatus` Hook là khai báo status của danh sách bạn bè. Đây là lý do tại sao nó lấy `friendID` làm đối số và return trạng thái người bạn này có online hay không:

```js
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  // ...

  return isOnline;
}
```

Bây giờ, hãy cho chúng tôi xem cách bạn tuỳ chỉnh Hook của mình nào.

## Cách sử dụng một Hook tuỳ chỉnh {#using-a-custom-hook}

Ban đầu, mục tiêu đã nêu của chúng tôi là loại bỏ logic trùng lặp trong `FriendStatus` và `FriendListItem` component. Cả hai đều muốn biết liệu một người bạn có online hay không.

Bây giờ chúng ta đã trích xuất logic này vào `useFriendStatus` Hook, chúng ta có thể *sử dụng nó:*

```js{2}
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

```js{2}
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

**Code này có  tương đương với những ví dụ ban đầu không?** Có, nó hoạt động giống như nhau. Nếu bạn nhìn kỹ, bạn sẽ nhận thấy chúng ta đã thực hiện bất kỳ thay đổi nào đối với hành vi. Tất cả những gì chúng ta làm là trích xuất một số code dùng chung giữa hai function thành một function riêng biệt. **Hook tuỳ chỉnh là một quy ước chứ không phải là một tính năng của React**

**Tôi có phải đặt tên cho các Hook tuỳ chỉnh của mình bắt đầu bằng "`use`"?** Hãy làm điều đó, quy ước này rất quan trọng. Nếu không có nó, chúng ta sẽ có thể tự động kiểm tra các vi phạm [những quy tắc trong Hook](/docs/hooks-rules.html) vì chúng ta không thể biết nếu một function có chứa các cuộc gọi đến Hook bên trong nó


**Làm thế nào để một Hook tuỳ chỉnh có state cô lập?** Mỗi *cuộc gọi* đến một Hook đều là state cô lập. Bởi vì chúng ta gọi `useFriendStatus` trực tiếp, từ khái niệm của React, component chỉ gọi `useState` và `useEffect`. Và như chúng ta đã [được học](/docs/hooks-state.html#tip-using-multiple-state-variables) [trước đó](/docs/hooks-effect.html#tip-use-multiple-effects-to-separate-concerns), chúng ta có thể gọi `useState` và `useEffect` nhiều lần trong một component, và chúng hoàn toàn cô lập.

### Mẹo: Tông tin giữa các Hook {#tip-pass-information-between-hooks}

Vì Hooks là các function, chúng ta có thể truyền thông tin giữa chúng.

Để minh hoạ điều này, chúng ta sẽ sử dụng component khác từ ví dụ chat của mình. Đây là tin nhắn hiển thị cho dù người bạn đang chat có online hay không: 

```js{8-9,13}
const friendList = [
  { id: 1, name: 'Phoebe' },
  { id: 2, name: 'Rachel' },
  { id: 3, name: 'Ross' },
];

function ChatRecipientPicker() {
  const [recipientID, setRecipientID] = useState(1);
  const isRecipientOnline = useFriendStatus(recipientID);

  return (
    <>
      <Circle color={isRecipientOnline ? 'green' : 'red'} />
      <select
        value={recipientID}
        onChange={e => setRecipientID(Number(e.target.value))}
      >
        {friendList.map(friend => (
          <option key={friend.id} value={friend.id}>
            {friend.name}
          </option>
        ))}
      </select>
    </>
  );
}
```

Chúng ta gán friendID bằng biến state `recipientID`, và câp nhật nó nếu người dùng chọn một người bạn trong thẻ `<select>`.

Bởi vì lệnh gọi `useState` Hook cung cấp cho chúng ta giá trị mới nhất của biến `recipientID`, chúng ta có thể chuyển nó tới `useFriendStatus` Hook tuỳ chỉnh làm đối số:

```js
  const [recipientID, setRecipientID] = useState(1);
  const isRecipientOnline = useFriendStatus(recipientID);
```

Điều này cho chings ta biết rằng, liệu người bạn đang chọn có online hay không. Nếu chúng ta chọn một người bạn khác và cập nhật biến state `recipientID`, `useFriendStatus` Hook sẽ huỷ đăng ký khỏi người bạn đã chọn trước đó và đăng ký trạng thái cho người mới được chọn.

## `useYourImagination()` {#useyourimagination}

Hooks tuỳ chỉnh cung cấp tính linh hoạt của việc chia sẻ logic mà trước đó có thể có trong các thành phần React. Bạn có thể viết các Hook của riêng bạn bao gồm nhiều trường hợp sử dụng như xử lý form, animation, declarative subscriptions, timers và nhiều hơn nữa. Hơn nữa, bạn có thể xây dựng các Hook dễ sử dụng như các tính năng tích hợp của React.

Cố gắng đừng thêm abstraction quá sớm. Bây giờ các function component có thể làm được nhiều hơn, nó có khả năng là average function component khiến code của bạn dài hơn. Điều nà là bình thường - đừng có cảm giác bạn phải chia nhỏ nó trong Hooks. Nhưng chúng tôi khuyến khích bạn bắt đầu phát hiện các trường hợp trong đó Hook tuỳ biến có thể ẩn logic phức tạp đằng sau một giao diện đơn giản hoặc gỡ rối một thành phần lộn xộn.

Ví dụ: có thể bạn có một component phức tạp chứa nhiều local state đươc quản lý môt cách đặc biệt. `useState` không tập trung cho logic cập nhật dễ dàng hơn vì vậy bạn có thể thích viết nó như [Redux](https://redux.js.org/) reducer:

```js
function todosReducer(state, action) {
  switch (action.type) {
    case 'add':
      return [...state, {
        text: action.text,
        completed: false
      }];
    // ... other actions ...
    default:
      return state;
  }
}
```

Reducers rất thuận tiện để kiểm tra độc lập và chia tỉ lệ để thể hiện logic cập nhật phức tạp. Bạn có thể chia chúng trong các reducers nhỏ hơn nếu cần thiết. Tuy nhiên bạn cũng có thể tận hưởng những lợi ích của việc sử dụng React local state hoặc có thể không muốn cài đặt một thư viện khác

Vậy điều gì sảy ra nếu chúng ta viết `useReducer` Hook cho phép quản lý local state của component bằng reducer? Một phiên bản đơn giản hoá của nó có thể trong như thế này:

```js
function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    const nextState = reducer(state, action);
    setState(nextState);
  }

  return [state, dispatch];
}
```

Bây giờ chúng ta có thể sử dụng nó trong component để reducer điều khiển quản lý state của nó:

```js{2}
function Todos() {
  const [todos, dispatch] = useReducer(todosReducer, []);

  function handleAddClick(text) {
    dispatch({ type: 'add', text });
  }

  // ...
}
```

Nhu cầu quản lý local state với reducer trong một component phức tạp là đủ phổ biến để chúng ta xây dựng `useReducer` Hook trong React. Bạn sẽ tìm thấy nó cùng với các Hook được tích hợp trong các [Hooks API reference](/docs/hooks-reference.html).

---
id: hooks-rules
title: Những quy tắc trong Hook
permalink: docs/hooks-rules.html
next: hooks-custom.html
prev: hooks-effect.html
---

*Hooks* là một tính năng mới được thêm vào từ phiên bản 16.8. Cho phép sử dụng state và các tính năng React mà không cần viết class

Bản chất của Hooks là các Javascript function, nhưng bạn cần tuân theo quy tắc khi sử dụng chúng. Chúng tôi cung cấp một [linter plugin](https://www.npmjs.com/package/eslint-plugin-react-hooks) để thực thi các quy tắc này một cách tự động:

### Chỉ gọi Hooks ở Top Level {#only-call-hooks-at-the-top-level}

Không được gọi Hook bên trong vòng lặp, biểu thức điều kiện hoặc các function lồng nhau. Thay vào đó, luôn sử dụng Hooks ở top level trong React function của bạn. Bằng cách tuân theo các quy tắc, bạn đảm bảo rằng các Hook đưcoj gọi theo cùng một thứ tự mỗi khi component render. Đó là những gì cho phép React bảo toàn chính xác state của Hook giữa nhiều lệnh goi `useState` và `useEffect`. (Nếu bạn tò mò, chúng tôi sẽ giải thích điều này sâu hơi ở [bên dưới](#explanation).) )

### Chỉ gọi Hooks từ React Function{#only-call-hooks-from-react-functions}

**Không được gọi Hooks từ các function Javascrip thông thường.** Thay vào đó, bạn có thể:

* ✅ Gọi Hooks từ React function components.
* ✅ Gọi Hooks từ các Hook tuỳ chỉnh (Chúng ta sẽ học chúng trong [phần tiếp theo](/docs/hooks-custom.html)).

Bằng cách tuân thủ các quy tắc, bạn đảm bảo rằng tất cả các stateful logic trong một component được hiển thị rõ ràng từ source code của nó.

## ESLint Plugin {#eslint-plugin}

Chúng tôi phát hành một plugin có tên [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks) để thực thi hai quy tắc này. Bạn có thể thêm plugin này vào project của mình nếu như bạn muốn thử:

```bash
npm install eslint-plugin-react-hooks --save-dev
```

```js
// Your ESLint configuration
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // Checks rules of Hooks
    "react-hooks/exhaustive-deps": "warn" // Checks effect dependencies
  }
}
```

Trong tương lai, chúng tôi dự định sẽ bao gồm plugin này vào Create React App và các bộ công cụ tương tự.

**Bạn có thể chuyển sang trang tiếp theo giải thích [làm thế nào bạn viết Hooks](/docs/hooks-custom.html) ngay bây giờ.** Trong trang này, chúng tôi sẽ tiếp tục bằng cách giải thích lý do những quy tắc này. 

## Giải thích {#explanation}

Như chúng ta đã [học trước đó](/docs/hooks-state.html#tip-using-multiple-state-variables), chúng ta có thể sử dụng nhiều State hoặc Effect Hooks trong một component:

```js
function Form() {
  // 1. Use the name state variable
  const [name, setName] = useState('Mary');

  // 2. Use an effect for persisting the form
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });

  // 3. Use the surname state variable
  const [surname, setSurname] = useState('Poppins');

  // 4. Use an effect for updating the title
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
```

Vậy, làm thế nào để React biết state nào tương ứng với lệnh gọi `useState` nào? Câu trả lời là **React dựa vào thứ tự mà Hook được gọi**. Ví dụ của chúng tôi hoạt động vì thứ tự gọi các lệnh gọi Hook giống nhau trên mỗi lần render:

```js
// ------------
// First render
// ------------
useState('Mary')           // 1. Initialize the name state variable with 'Mary'
useEffect(persistForm)     // 2. Add an effect for persisting the form
useState('Poppins')        // 3. Initialize the surname state variable with 'Poppins'
useEffect(updateTitle)     // 4. Add an effect for updating the title

// -------------
// Second render
// -------------
useState('Mary')           // 1. Read the name state variable (argument is ignored)
useEffect(persistForm)     // 2. Replace the effect for persisting the form
useState('Poppins')        // 3. Read the surname state variable (argument is ignored)
useEffect(updateTitle)     // 4. Replace the effect for updating the title

// ...
```

Miễn là thứ tự của các lệnh gọi Hooks giống nhau giữa các lần render, React có thể liên kết một số local state với từng trạng thái của chúng. Nhưng điều gì sảy ra nếu chúng ta đặt vào Hook  một lời gọi biểu thức điều kiện (ví dụ effect `persistForm`)?

```js
  // 🔴 We're breaking the first rule by using a Hook in a condition
  if (name !== '') {
    useEffect(function persistForm() {
      localStorage.setItem('formData', name);
    });
  }
```

Điều kiện `name !== ''` là đúng trong lần đầu render đầu tiên, vì vậy chúng tôi chạy Hook này. Tuy nhiên, trong lần render tiếp theo, người dùng có thể xoá form, khiến biểu thức điều kiện sai. Khiến Hook bị bỏ qua trong lần render tiếp theo, thứ tự cảu các lệnh gọi Hook trở nên khác nhau:

```js
useState('Mary')           // 1. Read the name state variable (argument is ignored)
// useEffect(persistForm)  // 🔴 This Hook was skipped!
useState('Poppins')        // 🔴 2 (but was 3). Fail to read the surname state variable
useEffect(updateTitle)     // 🔴 3 (but was 4). Fail to replace the effect
```

React không biết cái gì được return cho lần gọi `useState` thứ hai. React dự kiến rằng lần gọi Hook thứ hai trong component này tương ứng với hiệu ứng `persistForm`, giống nhau trong lần render trước đó, nhưng nó không còn nữa. Từ thời điểm đó, mỗi lần gọi Hook tiếp theo, chúng tôi bỏ qua, dẫn đến lỗi.

**Đây là lý do tai sao Hooks phải được đặt ở top level trong component.**  Nếu chúng ta muốn chạy một effect có điều kiện, chúng ta có thể đặt điều kiện đó trong Hook của mình:

```js
  useEffect(function persistForm() {
    // 👍 We're not breaking the first rule anymore
    if (name !== '') {
      localStorage.setItem('formData', name);
    }
  });
```

**Lưu ý rằng bạn không cần phải lo lắng về vấn đề này nếu bạn sử dụng [quy tắc lint](https://www.npmjs.com/package/eslint-plugin-react-hooks).** Nhưng bây giờ, bạn cũng đã biết tại sao Hook hoạt động theo cách này mà vấn đề quy tắc đang ngăn chặn.

## Phần tiếp theo {#next-steps}

Cuối cùng, chúng ta sẽ tìm hiểu [cách viết Hooks](/docs/hooks-custom.html)riêng của bạn! Hook tuỳ biến cho phép bạn kết hợp Hook được cung cấp bởi React vào abstraction, và sử dụng lại logic stateful giữa các component khác nhau.

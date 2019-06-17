---
id: react-dom
title: ReactDOM
layout: docs
category: Reference
permalink: docs/react-dom.html
---

Nếu bạn load React từ một thẻ `<script>`, ở những API cấp cao nhất sẽ có sẵn trên `ReactDOM` global. Nếu bạn dùng ES6 và npm, bạn có thể viết `import ReactDOM from 'react-dom'`. Nếu bạn dùng ES5 và npm, thì bạn có thể khai báo `var ReactDOM = require('react-dom')`.

## Giới thiệu chung {#overview}

Gói `react-dom` cung cấp các phương thức dành riêng cho DOM rằng có thể sử dụng ở cấp cao nhất trong ứng dụng của bạn và như là một lối thoát để thoát ra khỏi mô hình React nếu bạn cần đến. Hầu hết các "thành phần" (component) của bạn không cần phải sử dụng module này.

- [`render()`](#render)
- [`hydrate()`](#hydrate)
- [`unmountComponentAtNode()`](#unmountcomponentatnode)
- [`findDOMNode()`](#finddomnode)
- [`createPortal()`](#createportal)

### Trình duyệt hỗ trợ {#browser-support}

React hỗ trợ hầu hết các trình duyệt phổ biến, bao gồm Internet Explorer 9 trở lên, mặc dù [một số polyfill là bắt buộc](/docs/javascript-environment-requirements.html) cho những trình duyệt cũ hơn như IE 9 và IE 10.

> Lưu ý
>
> Chúng tôi không hỗ trợ các trình duyệt cũ không hỗ trợ các phương thức ES5, nhưng bạn có thể thấy rằng các ứng dụng của bạn vẫn hoạt động trong các trình duyệt cũ nếu các polyfill như [es5-shim và es5-sham](https://github.com/es-shims/es5-shim) được đưa vào trang. Bạn có thể tự mình độc lập nếu chọn theo con đường này.

* * *

## Tài liệu tham khảo {#reference}

### `render()` {#render}

```javascript
ReactDOM.render(element, container[, callback])
```

"Kết xuất" (render) một "thành phần" (element) React vào DOM trong `container` được cung cấp và trả về một ["quy chiếu" (reference)](/docs/more-about-refs.html) tới "thành phần" (component) (hoặc trả về `null` cho [các "thành phần" (component) "phi trạng thái" (stateless)](/docs/components-and-props.html#functional-and-class-components)).

Nếu element React trước đây đã từng được đưa vào `container`, điều này sẽ thực hiện việc cập nhật trên nó và chỉ làm thay đổi DOM khi cần thiết để phản ảnh element React mới nhất.

Nếu tuỳ chọn hàm callback được cung cấp, nó sẽ được thực thi sau khi "thành phần" được "kết xuất" (rendered) hoặc cập nhật.

> Lưu ý:
>
> `ReactDOM.render()` kiểm soát nội dung của container node mà bạn truyền vào. Mọi "phần tử" (element) DOM hiện có bên trong được thay thế khi được gọi lần đầu tiên. Các lần gọi sau này sẽ sử dụng thuật toán khuếch tán React DOM để cập nhật hiệu quả.
>
> `ReactDOM.render()` không sửa đổi container node (chỉ thay đổi các phần tử con của container). Nó có thể chèn một "thành phần" (component) vào DOM node hiện tại mà không cần phải ghi đè lên các phần tử con hiện có.
>
> `ReactDOM.render()` hiện trả về một "tham chiếu" (reference) tới thực thể `ReactComponent` gốc. Tuy nhiên, sử dụng giá trị trả về này là di sản
> và nên tránh vì các phiên bản sau của React có thể khiến các "thành phần" component không đồng bộ trong một số trường hợp. Nếu bạn cần một tham chiếu tới thực thể `ReactComponent` gốc, giải pháp được ưa thích là đính kèm một 
> ["tham chiếu gọi lại" (callback ref)](/docs/more-about-refs.html#the-ref-callback-attribute) cho phần tử gốc.
>
> Sử dụng `ReactDOM.render()` để hydrat hoá một container kết xuất máy chủ sẽ không được dùng nữa và sẽ bị xoá ở phiên bản React 17. Thay vào đó, hãy sử dụng [`hydrate()`](#hydrate).

* * *

### `hydrate()` {#hydrate}

```javascript
ReactDOM.hydrate(element, container[, callback])
```

Giống như hàm [`render()`](#render), nhưng được sử dụng để hydrat hoá một container có nội dung HTML khi được kết xuất bởi [`ReactDOMServer`](/docs/react-dom-server.html). React sẽ cố gắng gắn trình lắng nghe sự kiện vào đánh dấu hiện có.

React mong đợi rằng nội dung được hiển thị giống hệt nhau giữa server và client. Nó có thể vá những khác biệt trong nội dung văn bản, nhưng bạn nên coi sự sai khác này là bug và sửa lỗi chúng. Trong môi trường phát triển, React cảnh báo về sự sai khác trong quá trình hydrat hoá. There are no guarantees that attribute differences will be patched up in case of mismatches. This is important for performance reasons because in most apps, mismatches are rare, and so validating all markup would be prohibitively expensive.

If a single element's attribute or text content is unavoidably different between the server and the client (for example, a timestamp), you may silence the warning by adding `suppressHydrationWarning={true}` to the element. It only works one level deep, and is intended to be an escape hatch. Don't overuse it. Unless it's text content, React still won't attempt to patch it up, so it may remain inconsistent until future updates.

If you intentionally need to render something different on the server and the client, you can do a two-pass rendering. Components that render something different on the client can read a state variable like `this.state.isClient`, which you can set to `true` in `componentDidMount()`. This way the initial render pass will render the same content as the server, avoiding mismatches, but an additional pass will happen synchronously right after hydration. Note that this approach will make your components slower because they have to render twice, so use it with caution.

Remember to be mindful of user experience on slow connections. The JavaScript code may load significantly later than the initial HTML render, so if you render something different in the client-only pass, the transition can be jarring. However, if executed well, it may be beneficial to render a "shell" of the application on the server, and only show some of the extra widgets on the client. To learn how to do this without getting the markup mismatch issues, refer to the explanation in the previous paragraph.

* * *

### `unmountComponentAtNode()` {#unmountcomponentatnode}

```javascript
ReactDOM.unmountComponentAtNode(container)
```

Remove a mounted React component from the DOM and clean up its event handlers and state. If no component was mounted in the container, calling this function does nothing. Returns `true` if a component was unmounted and `false` if there was no component to unmount.

* * *

### `findDOMNode()` {#finddomnode}

> Note:
>
> `findDOMNode` is an escape hatch used to access the underlying DOM node. In most cases, use of this escape hatch is discouraged because it pierces the component abstraction. [It has been deprecated in `StrictMode`.](/docs/strict-mode.html#warning-about-deprecated-finddomnode-usage)

```javascript
ReactDOM.findDOMNode(component)
```
If this component has been mounted into the DOM, this returns the corresponding native browser DOM element. This method is useful for reading values out of the DOM, such as form field values and performing DOM measurements. **In most cases, you can attach a ref to the DOM node and avoid using `findDOMNode` at all.**

When a component renders to `null` or `false`, `findDOMNode` returns `null`. When a component renders to a string, `findDOMNode` returns a text DOM node containing that value. As of React 16, a component may return a fragment with multiple children, in which case `findDOMNode` will return the DOM node corresponding to the first non-empty child.

> Note:
>
> `findDOMNode` only works on mounted components (that is, components that have been placed in the DOM). If you try to call this on a component that has not been mounted yet (like calling `findDOMNode()` in `render()` on a component that has yet to be created) an exception will be thrown.
>
> `findDOMNode` cannot be used on function components.

* * *

### `createPortal()` {#createportal}

```javascript
ReactDOM.createPortal(child, container)
```

Creates a portal. Portals provide a way to [render children into a DOM node that exists outside the hierarchy of the DOM component](/docs/portals.html).

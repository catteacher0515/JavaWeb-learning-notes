好的，我们现在就把 `.attr()` 和 `.prop()` 这对最重要也最容易混淆的属性操作方法，彻底地沉淀下来。

-----

# jQuery 属性操作的核心：.attr() 与 .prop() 的对决与真相

## 1\. 本质公理与路径

  * **公理**: 每一个网页元素都拥有**双重身份**。一个身份是写在 HTML 文件里的\*\*“静态蓝图” (Attribute)**，另一个身份是浏览器加载后在内存中生成的**“动态实体” (Property)\*\*。这两个身份既有联系，又有本质区别。
  * **路径**: 要想真正掌握属性操作，就必须先清晰地划开 Attribute 和 Property 的边界，然后为这两个不同的“身份”，分别使用专门的工具——`.attr()` 和 `.prop()`。用错了工具，就会导致难以预料的奇怪 Bug。

-----

## 2\. 概念划界：属性 (Attribute) vs. 特性 (Property)

这是理解一切的基石。我们用“建筑图纸”和“实体房子”的比喻来彻底分清它们。

| 对比维度 | Attribute (属性) | Property (特性) |
| :--- | :--- | :--- |
| **比喻** | 房子的**建筑图纸** | 已经建好的**实体房子** |
| **来源** | HTML 源代码，是你亲手写的 | 浏览器在内存中创建的 DOM 对象 |
| **值类型** | **永远是字符串** (e.g., `"true"`, `"checked"`) | 多种类型 (e.g., 布尔值 `true`, `false`, 数字) |
| **状态** | 描述**初始状态**，相对固定 | 反映**当前实时状态**，可随用户交互而改变 |
| **核心工具**| **`.attr()`** | **`.prop()`** |

**一个决定性的例子：复选框 (`checkbox`)**

**HTML 源代码 (图纸):**

```html
<input id="myCheck" type="checkbox" checked="checked">
```

  * 在这里，`checked="checked"` 是一个 **Attribute**。它只是在图纸上说：“这栋房子建好后，这个开关默认应该是开着的”。

**当页面加载后 (实体):**

1.  浏览器看到图纸，建好了这个复选框的“实体”，它的 `checked` **Property** 此时为布尔值 `true`。
2.  现在，用户用鼠标**点击了一下**，取消了勾选。
3.  此时，这个复选框实体的 `checked` **Property** 立刻变成了 `false`。
4.  但是，如果你此刻去查看网页源代码（图纸），那个 `checked="checked"` 的 **Attribute** 依然存在，它没有变。

**结论**: `.attr('checked')` 读取的是图纸上的初始设定，而 `.prop('checked')` 读取的是实体房子上开关的当前真实状态（`true` 或 `false`）。

-----

## 3\. 拆解与重构：两大方法的应用场景

### `.attr()` - “图纸”的读写器

`.attr()` 用于获取和设置在 HTML 标签中明确定义的属性，特别是那些没有 `true`/`false` 状态的属性。

  * **获取**: `$(selector).attr("属性名")`
    ```javascript
    // 获取图片的 src 地址
    var imageUrl = $("img").attr("src");
    ```
  * **设置**: `$(selector).attr("属性名", "新值")`
    ```javascript
    // 给所有链接添加一个 "新窗口打开" 的提示
    $("a").attr("title", "点击后会在新窗口打开");
    ```
  * **移除**: `$(selector).removeAttr("属性名")`
    ```javascript
    // 移除所有图片的 title 属性
    $("img").removeAttr("title");
    ```
  * **核心应用场景**: `id`, `class`, `href`, `src`, `title`, 以及自定义的 `data-*` 属性等。

### `.prop()` - “实体”的状态检测器

`.prop()` 用于获取和设置 DOM 对象的属性，特别是那些能反映元素**当前实时状态**的属性，尤其是布尔值属性。

#### 黄金法则

> **对于 `checked`, `selected`, `disabled`, `readonly` 这些具有 `true`/`false` 状态的属性，永远、总是、必须使用 `.prop()`。**

  * **获取**: `$(selector).prop("属性名")`
    ```javascript
    // 判断复选框当前是否真的被选中了？
    if ( $("#myCheck").prop("checked") ) {
        // ...执行操作
    }
    ```
  * **设置**: `$(selector).prop("属性名", true/false)`
    ```javascript
    // 用代码禁用一个输入框
    $("#myInput").prop("disabled", true);
    ```

-----

## 4\. 重构式行动清单：决策流程：到底用哪个？

当你犹豫不决时，按这个顺序问自己：

1.  我要操作的是表单元素的 **`value`** 吗？

      * **是** -\> 使用 **`.val()`**。

2.  我要操作的是一个**布尔状态**吗（比如一个东西是否被选中、是否被禁用）？

      * **是** -\> 使用 **`.prop()`**。

3.  以上都不是？那它就是一个常规的 HTML 属性。

      * **是** -\> 使用 **`.attr()`**。

-----

### 常见“陷阱”与最佳实践

#### 陷阱 1: 用 `.attr()` 获取 `checked` 状态

这是导致无数 Bug 的根源。

```javascript
// 当复选框被取消勾选后...
console.log( $("#myCheck").attr("checked") ); // 可能返回 "checked" 或 undefined，行为不稳定！
console.log( $("#myCheck").prop("checked") ); // 永远准确地返回 false
```

**切记**: 判断状态，相信实体 (`.prop()`)，别信图纸 (`.attr()`)。

#### 最佳实践: 使用 `.data()` 操作 `data-*` 属性

虽然用 `.attr('data-user-id')` 可以操作自定义属性，但 jQuery 提供了更专业的方法 `.data()`。

**HTML**: `<div id="user" data-user-id="123"></div>`

**jQuery**:

```javascript
// 获取 data 属性 (会自动把字符串 "123" 转换为数字 123)
var userId = $("#user").data("user-id"); 

// 设置 data 属性
$("#user").data("role", "admin");
```

`.data()` 的优势在于它能进行类型转换，并且性能更好。

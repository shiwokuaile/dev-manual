# 整合技术

## Mermaid

用于显示时序图的组件。[Mermaid User Guide | Mermaid](https://mermaid.js.org/intro/getting-started.html)

**安装**

```shell
npm install -g yarn
yarn add mermaid
```

**使用**

```javascript
import mermaid from "mermaid";
let config = { startOnLoad: true };
mermaid.initialize(config);
```

```html
<div>
  <div class="mermaid">
    sequenceDiagram
    Alice->>Bob: Hello Bob, how are you ?;
    Bob->>Alice: Fine, thank you. And you?;
    create participant Carl;
    Alice->>Carl: Hi Carl!;
    create actor D as Donald;
    Carl->>D: Hi!;
    destroy Carl;
    Alice-xCarl: We are too many;
    destroy Bob;
    Bob->>Alice: I agree;
  </div>
</div>
```

**效果**

![](https://assets.shiwokuaile.top/pic/202407/c49274560774b7280a095ce6bdcde71872106a.png)

**注意**

> 首行注明 UML 图类型
> 
> 每一行用分号结尾，否则除了首末两行，中间都会被认为是内容

# PDF Boxes

```txt
mediabox, cropbox, bleedbox, trimbox, artbox
```

PDF 实际上有5个不同类型的尺寸，而不仅仅只有你打开它后看到的大小。

An EPS has only a single BoundingBox but a PDF contains a MediaBox, CropBox, BleedBox, TrimBox and ArtBox.

- trimbox: 在所有裁剪操作之后,Trimbox 基本上确定了是导出PDF成品的页面大小。
- bleedbox: 包含了trimbox和所有的bleed。
- mediabox: 包含了bleedbox和任何的crop/bleed/etc...标记
- cropbox: 它指定了查看器中显示的区域，在Adobe Reader中，文档显示的大小就是cropbox的大小
- artbox: 最初用来指定艺术作品所覆盖的区域。

## mediabox

对大部分用户来说，mediabox对应着pdf页面的实际大小。但在用于出版时，mediabox包含了更多的有用的信息（比如说bleed，修建标记，文件名，日期等等），便于编辑。

## bleedbox

如果文档布局上的任何元素与文档边缘接触，则使用bleed。bleedbox的大小不同地方有不同的标准，常见的是比crop之后每条边长3~6mm。


## 
-----------------------------
EPS ???


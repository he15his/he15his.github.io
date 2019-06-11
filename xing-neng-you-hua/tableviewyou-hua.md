

* 使用UITableViewCell的重用机制

* 优化UITableViewCell高度计算
* 懒加载（延迟加载图片），还未显示的数据可先不处理
* 缓存图片
* 调整图片大小，保证图片大小和 UIImageView 大小相同，因为在运行中缩放图片很耗费资源
* 图片代码渲染 or 直接获取
* 避免使用过于复杂的xib
* cell 中尽可能少的 view



**避免图层混合**

- 确保控件的opaque属性设置为true，确保backgroundColor和父视图颜色一致且不透明。

- 如无特殊需要，不要设置低于1的alpha值。

- 确保UIImage没有alpha通道。

  

### 离屏渲染处理

#### 圆角

1.视图上添加一个子layer到最上层，用于遮盖该视图及其子视图，设置layer的图片为刚好能够遮盖成所需圆角样子

2.使用异步绘制圆角图片的方式



注意：png 图片 在 UIImageView 这样处理圆角是不会产生离屏渲染的。（ios9.0之后不会离屏渲染，ios9.0之前还是会离屏渲染）。



#### 阴影

使用Shadow Path可避免离屏渲染



#### 渐变

无法避免离屏渲染，只有叫 UI 给图片
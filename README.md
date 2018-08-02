# d3-transition

`transition` 是一个类 [selection](https://github.com/xswei/d3-selection) 的接口，用来对 `DOM` 进行动画修改。这种修改不是立即修改，而是在规定的事件内平滑过渡到目标状态。

应用过渡，首先要选中元素，然后调用 [*selection*.transition](#selection_transition)，并且设置期望的改变，例如:

```js
d3.select("body")
  .transition()
    .style("background-color", "red");
```

过渡支持大多数选择集的方法(比如 [*transition*.attr](#transition_attr) 和 [*transition*.style](#transition_style) 对应 [*selection*.attr](https://github.com/d3/d3-selection#selection_attr) 和 [*selection*.style](https://github.com/d3/d3-selection#selection_style))，但是并不是所有的方法都支持; 比如必须在对元素过渡之前 [append](https://github.com/d3/d3-selection#selection_append) 元素或者 [bind data](https://github.com/d3/d3-selection#joining-data)。[*transition*.remove](#transition_remove) 操作可以在动画结束时方便的移除元素。

为了计算过渡过程中的状态，过渡集成了大量的 [built-in interpolators](https://github.com/d3/d3-interpolate)。[Colors](https://github.com/d3/d3-interpolate#interpolateRgb), [numbers](https://github.com/d3/d3-interpolate#interpolateNumber), 以及 [transforms](https://github.com/d3/d3-interpolate#interpolateTransform) 会被自动检测。内嵌数字的 [Strings](https://github.com/d3/d3-interpolate#interpolateString) 也会被检测到，可以方便的对许多样式(比如 `padding` 或 `font size`) 以及 `path` 进行过渡。可以使用 [*transition*.attrTween](#transition_attrTween), [*transition*.styleTween](#transition_styleTween) 或 [*transition*.tween](#transition_tween) 指定一个自定义插值器。

## Installing

`NPM` 安装: `npm install d3-transition`. 此外还可以下载 [latest release](https://github.com/d3/d3-transition/releases/latest). 可以直接从 [d3js.org](https://d3js.org) 以 [standalone library](https://d3js.org/d3-transition.v1.min.js) 或作为 [D3 4.0](https://github.com/d3/d3) 的一部分载入. 支持 `AMD`, `CommonJS` 以及基本的标签引入形式，如果使用标签引入形式则会暴露全局 `d3` 变量:

```html
<script src="https://d3js.org/d3-color.v1.min.js"></script>
<script src="https://d3js.org/d3-dispatch.v1.min.js"></script>
<script src="https://d3js.org/d3-ease.v1.min.js"></script>
<script src="https://d3js.org/d3-interpolate.v1.min.js"></script>
<script src="https://d3js.org/d3-selection.v1.min.js"></script>
<script src="https://d3js.org/d3-timer.v1.min.js"></script>
<script src="https://d3js.org/d3-transition.v1.min.js"></script>
<script>

var transition = d3.transition();

</script>
```

[在浏览器中测试 `d3-transition`.](https://tonicdev.com/npm/d3-transition)

## API Reference

* [Selecting Elements](#selecting-elements)
* [Modifying Elements](#modifying-elements)
* [Timing](#timing)
* [Control Flow](#control-flow)
* [The Life of a Transition](#the-life-of-a-transition)

### Selecting Elements

过渡通过 [*selection*.transition](#selection_transition) 派生自 [selections](https://github.com/d3/d3-selection)。你也可以使用 [d3.transition](#transition) 在文档根元素上创建一个过渡。

Transitions are derived from [selections](https://github.com/d3/d3-selection) via [*selection*.transition](#selection_transition). You can also create a transition on the document root element using [d3.transition](#transition).

<a name="selection_transition" href="#selection_transition">#</a> <i>selection</i>.<b>transition</b>([<i>name</i>]) [<>](https://github.com/d3/d3-transition/blob/master/src/selection/transition.js "Source")

在指定的 *selection* 上返回指定的 *name* 的过渡。如果没有指定 *name* 则会使用 `null`。新的过渡仅仅与其他相同名字的过渡相排斥。

如果 *name* 是一个 [transition](#transition) 实例，则返回的过渡与指定的过渡具有相同的 `id` 和 `name`。如果已经选中的元素上已经存在相同 `id` 和 `name` 的过渡，则返回该元素已有的过渡。否则，返回的过渡的时间会从已经选中的每个选定元素的具有相同 `id` 的最近祖先继承。因此，这个方法可以用来同步多个选择集的过渡，或者重新选择特定元素的过渡并修改其配置。例如:

```js
var t = d3.transition()
    .duration(750)
    .ease(d3.easeLinear);

d3.selectAll(".apple").transition(t)
    .style("fill", "red");

d3.selectAll(".orange").transition(t)
    .style("fill", "orange");
```

如果指定的 *transition* 没有在已选中元素的祖先元素上找到(比如如果过渡 [already ended](#the-life-of-a-transition))，则默认的时间参数会被使用；但是在未来的版本中，这种情况可能会抛出错误。参考 [#59](https://github.com/d3/d3-transition/issues/59)。

If the specified *transition* is not found on a selected node or its ancestors (such as if the transition [already ended](#the-life-of-a-transition)), the default timing parameters are used; however, in a future release, this will likely be changed to throw an error. See [#59](https://github.com/d3/d3-transition/issues/59).

<a name="selection_interrupt" href="#selection_interrupt">#</a> <i>selection</i>.<b>interrupt</b>([<i>name</i>]) [<>](https://github.com/d3/d3-transition/blob/master/src/selection/interrupt.js "Source")

中断指定 *name* 的活动的过渡，并且取消指定 *name* 未执行的过渡(如果存在的话)。如果没有指定 *name* 则默认使用 `null`。

在元素上中断过渡不会影响其后代的任何过渡。例如，[axis transition](https://github.com/d3/d3-axis) 由多个独立的，同步的过渡组成，这些过渡是对 [G element](https://www.w3.org/TR/SVG/struct.html#Groups) 的后代元素进行过渡(`tick lines`, `tick labels`, `domain path`, *etc.*)。如果要中断 `axis` 的过渡，必须中断其所有后代元素的过渡:

```js
selection.selectAll("*").interrupt();
```

[universal selector(通配选择符)](https://developer.mozilla.org/en-US/docs/Web/CSS/Universal_selectors) `*` 表示选择所有的后代元素. 如果你也要中断 `G` 元素自身的过渡，则:

```js
selection.interrupt().selectAll("*").interrupt();
```

<a name="interrupt" href="#interrupt">#</a> d3.<b>interrupt</b>(<i>node</i>[, <i>name</i>]) [<>](https://github.com/d3/d3-transition/blob/master/src/interrupt.js "Source")

中断指定 *node* 上指定 *name* 的活跃的过渡，并取消未执行的指定 *name* 的过渡(如果存在的话)。如果没有指定 *name* 则使用 `null`。参考 [*selection*.interrupt](#selection_interrupt)。

<a name="transition" href="#transition">#</a> d3.<b>transition</b>([<i>name</i>]) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/index.js#L29 "Source")

在根元素 `document.documentElement` 上返回一个新的过渡，并指定 *name*。如果没有指定 *name*，则使用 `null`。新的过渡只与同名的过渡相排斥。*name* 也可以是一个 [transition](#transition) 实例；参考 [*selection*.transition](#selection_transition)。这个方法等价于:

```js
d3.selection()
  .transition(name)
```

这个函数也可以被用来测试是否是过渡实例 (`instanceof d3.transition`) 或者用来扩展过渡原型链。

<a name="transition_select" href="#transition_select">#</a> <i>transition</i>.<b>select</b>(<i>selector</i>) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/select.js "Source")

对于每一个选中的元素，选中匹配指定 *selector* 的第一个后代元素(如果存在的话) 并在结果选择集上返回一个过渡。*selector* 可以是一个字符串也可以是一个函数。如果是函数的话，会为每一个选中的元素依次调用，并传递当前数据 `d` 以及索引 `i`, 函数内部 `this` 指向当前 `DOM` 元素。新的过渡拥有相同的 `id`, `name` 以及时间；如果选中的元素已经存在相同 `id` 的过渡，则已存的过渡会被返回。

这个方法等价于通过 [*transition*.selection](#transition_selection) 从过渡中获取选择集，通过 [*selection*.select](https://github.com/d3/d3-selection#selection_select) 创建一个子选择集 然后通过 [*selection*.transition](#selection_transition) 创建一个新的过渡:

```js
transition
  .selection()
  .select(selector)
  .transition(transition)
```

<a name="transition_selectAll" href="#transition_selectAll">#</a> <i>transition</i>.<b>selectAll</b>(<i>selector</i>) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/selectAll.js "Source")

对于每一个选中的元素，选中匹配指定 *selector* 的所有后代元素(如果存在的话) 并在结果选择集上返回一个过渡。*selector* 可以是一个字符串也可以是一个函数。如果是函数的话，会为每一个选中的元素依次调用，并传递当前数据 `d` 以及索引 `i`, 函数内部 `this` 指向当前 `DOM` 元素。新的过渡拥有相同的 `id`, `name` 以及时间；如果选中的元素已经存在相同 `id` 的过渡，则已存的过渡会被返回。

这个方法等价于通过 [*transition*.selection](#transition_selection) 从过渡中获取选择集，通过 [*selection*.selectAll](https://github.com/d3/d3-selection#selection_selectAll) 创建一个子选择集 然后通过 [*selection*.transition](#selection_transition) 创建一个新的过渡:

```js
transition
  .selection()
  .selectAll(selector)
  .transition(transition)
```

<a name="transition_filter" href="#transition_filter">#</a> <i>transition</i>.<b>filter</b>(<i>filter</i>) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/filter.js "Source")

对于每一个选中的元素，选中匹配指定 *filter* 的元素并在结果选择集上返回一个过渡。*filter* 可以是一个字符串也可以是一个函数。如果是函数的话，会为每一个选中的元素依次调用，并传递当前数据 `d` 以及索引 `i`, 函数内部 `this` 指向当前 `DOM` 元素。新的过渡拥有相同的 `id`, `name` 以及时间；如果选中的元素已经存在相同 `id` 的过渡，则已存的过渡会被返回。

这个方法等价于通过 [*transition*.selection](#transition_selection) 从过渡中获取选择集，通过 [*selection*.filter](https://github.com/d3/d3-selection#selection_filter) 创建一个子选择集 然后通过 [*selection*.transition](#selection_transition) 创建一个新的过渡:

```js
transition
  .selection()
  .filter(filter)
  .transition(transition)
```

<a name="transition_merge" href="#transition_merge">#</a> <i>transition</i>.<b>merge</b>(<i>other</i>) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/merge.js "Source")

返回一个将当前过渡与指定的 *other* 过渡合并的新的过渡，并且指定的过渡必须与当前过渡有相同的 `id`。返回的过渡与当前过渡有相同的分组数量，相同的父节点以及相同的 `id`。任何当前过渡中缺失(`null`)的元素将会被来自 *other* 过渡中对应的元素填充(如果非空)。

这个方法等价于通过 [*transition*.selection](#transition_selection) 从过渡中获取选择集，通过 [*selection*.merge](https://github.com/d3/d3-selection#selection_merge) 创建一个子选择集然后通过 [*selection*.transition](#selection_transition) 创建一个新的过渡:

```js
transition
  .selection()
  .merge(other.selection())
  .transition(transition)
```

<a name="transition_transition" href="#transition_transition">#</a> <i>transition</i>.<b>transition</b>() [<>](https://github.com/d3/d3-transition/blob/master/src/transition/transition.js "Source")

当前过渡结束时，在相同的选定元素上返回一个新的过渡，这个过渡将在当前过渡结束时开始。新的过渡继承了参考时间，也就是新的过渡的开始时间等于之前过渡的 [delay](#transition_delay) 和 [duration](#transition_duration)] 之和。新的过渡也继承了当前过渡的 `name`, `duration` 以及 [easing](#transition_ease). 这个方法可以用来创建一些列的链式过渡, 例如：

```js
d3.selectAll(".apple")
  .transition() // First fade to green.
    .style("fill", "green")
  .transition() // Then red.
    .style("fill", "red")
  .transition() // Wait one second. Then brown, and remove.
    .delay(1000)
    .style("fill", "brown")
    .remove();
```

每个过渡的延时都与前序过渡有关。上述例子中，`apples` 将在过渡为 `brown` 之前保持 `1` 秒的 `red`。

<a name="transition_selection" href="#transition_selection">#</a> <i>transition</i>.<b>selection</b>() [<>](https://github.com/d3/d3-transition/blob/master/src/transition/selection.js "Source")

返回当前过渡对应的 [selection](https://github.com/xswei/d3-selection#selection).

<a name="active" href="#active">#</a> d3.<b>active</b>(<i>node</i>[, <i>name</i>]) [<>](https://github.com/d3/d3-transition/blob/master/src/active.js "Source")

返回指定 *node* 上指定 *name* 的活动的过渡(如果存在的话)。如果没有指定 *name* 则使用 `null`。如果指定的 *node* 上没有对应的过渡则返回 `null`。这个方法在创建循环过渡时很有用，比如:

```js
d3.selectAll("circle").transition()
    .delay(function(d, i) { return i * 50; })
    .on("start", function repeat() {
        d3.active(this)
            .style("fill", "red")
          .transition()
            .style("fill", "green")
          .transition()
            .style("fill", "blue")
          .transition()
            .on("start", repeat);
      });
```

参考 [chained transitions](http://bl.ocks.org/mbostock/70d5541b547cc222aa02) 获取更多例子.

### Modifying Elements

在选中元素并使用 [*selection*.transition](#selection_transition) 创建过渡之后，使用过渡的变换方法来影响文档的内容。

<a name="transition_attr" href="#transition_attr">#</a> <i>transition</i>.<b>attr</b>(<i>name</i>, <i>value</i>) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/attr.js "Source")

对于每个选中的元素，将为指定的属性 *name* 分配 [attribute tween(属性补间)](#transition_attrTween) 值。补间的初始值为过渡开始时的属性值，目标值可以通过常量和函数指定。如果是函数的话，会立即为每个选中的元素调用，并传递当前的数据 `d` 以及索引 `i`, 函数内部 `this` 指向当前 `DOM` 元素。

如果目标值为 `null` 则当过渡开始时此属性会被移除。否则会根据目标值的类型依据以下算法选择恰当的插值器:

1. 如果 *value* 为数值, 使用 [interpolateNumber](https://github.com/xswei/d3-interpolate#interpolateNumber).
2. 如果 *value* 为 [color](https://github.com/xswei/d3-color#color) 或可以被强制转为颜色的字符串, 使用 [interpolateRgb](https://github.com/d3/d3-interpolate#interpolateRgb).
3. 使用 [interpolateString](https://github.com/d3/d3-interpolate#interpolateString).

如果想使用其他插值器，则使用 [*transition*.attrTween](#transition_attrTween) 方法.

<a name="transition_attrTween" href="#transition_attrTween">#</a> <i>transition</i>.<b>attrTween</b>(<i>name</i>[, <i>factory</i>]) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/attrTween.js "Source")

如果指定了 *factory* 并且不为 `null`, 则将指定 *name* 的属性 [tween](#transition_tween) 设置为指定的插值器 *factory*。*factory* 是一个返回 [interpolator](https://github.com/d3/d3-interpolate) 的函数; 当过渡开始时，*factory* 会为每个选中的元素进行调用，并依次传递当前元素绑定的数据 `d` 以及索引 `i`, 函数内部 `this` 指向当前 `DOM` 元素。返回的插值器会在过渡过程中的每一帧进行调用，并依次传入 [eased(缓动)](#transition_ease) 时间 *t*, 通常情况下在 [0, 1] 范围内。最后插值器返回的值将会被用来设置为当前属性值。差孩子气必须返回字符串。(在过渡开始时移除属性使用[*transition*.attr](#transition_attr); 在过渡结束时移除属性使用 [*transition*.on](#transition_on) 来监听 *end* 事件.)

如果指定的 *factory* 为 `null`, 则表示移除之前改属性名对应的属性补间(如果存在的话)。如果 *factory* 没有指定则返回当前指定 *name* 的插值器工厂函数，如果不存在则返回 `undefined`。

例如，在 `red` 和 `blue` 之间进行插值:

```js
transition.attrTween("fill", function() {
  return d3.interpolateRgb("red", "blue");
});
```

或者从当前填充颜色过渡到 `blue`，与 [*transition*.attr](#transition_attr) 类似:

```js
transition.attrTween("fill", function() {
  return d3.interpolateRgb(this.getAttribute("fill"), "blue");
});
```

或者应用一个自定义的 `rainbow` 插值器:

```js
transition.attrTween("fill", function() {
  return function(t) {
    return "hsl(" + t * 360 + ",100%,50%)";
  };
});
```

这个方法在指定自定义插值器时非常有用，比如可以对 [SVG paths](http://bl.ocks.org/mbostock/3916621) 进行过渡。一个很有用的是对 *data interpolation*，利用 [d3.interpolateObject](https://github.com/d3/d3-interpolate#interpolateObject) 对两个数据值进行插值, 然后使用插值过程中的值(比如使用 [shape](https://github.com/d3/d3-shape)) 可以被用来设置为属性值。

<a name="transition_style" href="#transition_style">#</a> <i>transition</i>.<b>style</b>(<i>name</i>, <i>value</i>[, <i>priority</i>]) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/style.js "Source")

对于每个选中的元素，将为指定的样式 *name* 分配 [style tween(样式补间)](#transition_styleTween) 值, 并设置指定的 *priority*(优先级)。补间的初始值为过渡开始时的内联样式值(如果存在)，否则为计算值，目标值可以通过常量和函数指定。如果是函数的话，会立即为每个选中的元素调用，并传递当前的数据 `d` 以及索引 `i`, 函数内部 `this` 指向当前 `DOM` 元素。

如果目标值为 `null` 则当过渡开始时此属性会被移除。否则会根据目标值的类型依据以下算法选择恰当的插值器:

If the target value is null, the style is removed when the transition starts. Otherwise, an interpolator is chosen based on the type of the target value, using the following algorithm:

1. 如果 *value* 为数值, 使用 [interpolateNumber](https://github.com/d3/d3-interpolate#interpolateNumber).
2. 如果 *value* 为 [color](https://github.com/d3/d3-color#color) 或可以被强制转为颜色的字符串, 使用 [interpolateRgb](https://github.com/d3/d3-interpolate#interpolateRgb).
3. 使用 [interpolateString](https://github.com/d3/d3-interpolate#interpolateString).

如果想使用其他插值器，则使用 [*transition*.styleTween](#transition_styleTween) 方法.

<a name="transition_styleTween" href="#transition_styleTween">#</a> <i>transition</i>.<b>styleTween</b>(<i>name</i>[, <i>factory</i>[, <i>priority</i>]])) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/styleTween.js "Source")

如果指定了 *factory* 并且不为 `null`, 则将指定 *name* 的样式 [tween(补间)](#transition_tween) 设置为指定的插值器 *factory*。插值器工厂是一个返回 [interpolator](https://github.com/xswei/d3-interpolate) 的函数; 当过渡开始时, *factory* 会为每个选中的元素调用并依次传入当前元素绑定的数据 `d` 以及索引 `i`, 函数内部 `this` 指向当前 `DOM` 元素。返回的插值器会在过渡过程中的每一帧进行调用并依次传入 [eased(缓动)](#transition_ease) 时间 *t*, 通常处于 [0, 1] 范围之内。最后插值器返回的值会被设置为带有指定 *priority* 的样式值。插值器必须返回一个字符串。(在过渡开始时移除样式使用 [*transition*.style](#transition_style); 在过渡结束时移除样式使用 [*transition*.on](#transition_on) 来监听 *end* 事件。)

如果指定的 *factory* 为 `null`, 则表示移除之前改属性名对应的样式补间(如果存在的话)。如果 *factory* 没有指定则返回当前指定 *name* 的插值器工厂函数，如果不存在则返回 `undefined`。

例如，在 `red` 和 `blue` 之间进行填充样式插值:

```js
transition.styleTween("fill", function() {
  return d3.interpolateRgb("red", "blue");
});
```

或者从当前填充颜色过渡到 `blue`，与 [*transition*.style](#transition_style) 类似:

```js
transition.styleTween("fill", function() {
  return d3.interpolateRgb(this.style.fill, "blue");
});
```

或者应用一个自定义的 `rainbow` 插值器:

```js
transition.styleTween("fill", function() {
  return function(t) {
    return "hsl(" + t * 360 + ",100%,50%)";
  };
});
```

这个方法在自定义插值器时非常有用，比如对 *data interpolation*，[d3.interpolateObject](https://github.com/xswei/d3-interpolate#interpolateObject) 可以用来对数据值进行插值，插值过程中的补间值可以用来设置样式值。

<a name="transition_text" href="#transition_text">#</a> <i>transition</i>.<b>text</b>(<i>value</i>) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/text.js "Source")

对于每个选中的元素，在过渡开始时设置 [text content](http://www.w3.org/TR/DOM-Level-3-Core/core.html#Node3-textContent) 为指定的目标 *value*。*value* 可以是一个常量也可以是一个函数。如果是函数则会立即为每个选中的元素调用，并依次传递当前数据 `d` 以及索引 `i`，函数内部 `this` 指向当前 `DOM` 元素。函数的返回值将会被作为每个元素的文本内容。`null` 表示清空内容。

使用 [*transition*.tween](#transition_tween) ([for example(例子)](http://bl.ocks.org/mbostock/7004f92cac972edef365)) 对文本插值要比在开始时就设置内容要好很多，也可以使用一个淡入淡出的元素来替代 ([for example(例子)](http://bl.ocks.org/mbostock/f7dcecb19c4af317e464))。文本默认情况下不会被插值因为通常情况下是不能被插值的。

<a name="transition_remove" href="#transition_remove">#</a> <i>transition</i>.<b>remove</b>() [<>](https://github.com/d3/d3-transition/blob/master/src/transition/remove.js "Source")

对于每个选中的元素，如果在过渡结束后没有活动的过渡或者没有还未执行的过渡则 [removes](https://github.com/xswei/d3-selection#selection_remove) 此元素，否则什么都不做。

<a name="transition_tween" href="#transition_tween">#</a> <i>transition</i>.<b>tween</b>(<i>name</i>[, <i>value</i>]) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/tween.js "Source")

对每个选中的元素，使用指定的 *value* 函数作为指定 *name* 的补间。*value* 必须是一个返回函数的函数。在过渡开始时，指定的函数会为每一个选中的元素进行调用，并传递当前元素绑定的数据 `d` 以及索引 `i`，函数内部 `this` 指向当前 `DOM` 元素。返回的函数会在过渡中的每一帧进行调用，并传递当前 [eased](#transition_ease) 时间 *t*, 通常情况下处于 [0, 1] 之间。如果指定的 *value* 为 `null` 则表示移除之前指定 *name* 对应的 *tween* 补间(如果存在的话)。 

例如，将 `fill` 属性进行插值，类似于 [*transition*.attr](#transition_attr):

```js
transition.tween("attr.fill", function() {
  var node = this, i = d3.interpolateRgb(node.getAttribute("fill"), "blue");
  return function(t) {
    node.setAttribute("fill", i(t));
  };
});
```

这个方法在自定义插值器或进行其他操作时很有用, 比如对 [scroll offset](http://bl.ocks.org/mbostock/1649463) 进行过渡。

### Timing

过渡的 [easing](#transition_ease), [delay](#transition_delay) 和 [duration](#transition_duration) 都是可配置的。例如在元素排序时候每个元素的 [stagger the reordering(交错排序)](http://bl.ocks.org/mbostock/3885705) 可以提高感知能力。参考 [Animated Transitions in Statistical Data Graphics(统计数据图形中的动画转换)](http://vis.berkeley.edu/papers/animated_transitions/) 获取过渡动画的相关建议。

<a name="transition_delay" href="#transition_delay">#</a> <i>transition</i>.<b>delay</b>([<i>value</i>]) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/delay.js "Source")

对于每个选中的元素，将当前元素的过渡的延时设置为指定的 *value*(毫秒)。*value* 可以是一个常量也可以是一个函数，如果是函数则会为每个元素立即调用并依次传递当前数据 `d` 以及索引 `i`，函数内部 `this` 指向当前 `DOM` 元素。函数的返回值被用来设置为该元素的过渡延时。如果没有指定延时，则默认为 `0`。

如果没有指定 *value* 则返回当前第一个非空元素的延时时间。在已知过渡仅仅包含一个元素的情况下，这种获取形式通常有用。

将当前元素过渡的延时设置为当前元素索引 `i` 的倍数是一种方便的创建交错过渡的方式。例如:

```js
transition.delay(function(d, i) { return i * 10; });
```

当然，你可以以函数的形式动态计算延时，或者在计算基于索引的延时之前 [sort the selection(对选择集排序)](https://github.com/d3/d3-selection#selection_sort)。

<a name="transition_duration" href="#transition_duration">#</a> <i>transition</i>.<b>duration</b>([<i>value</i>]) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/duration.js "Source")

对于每个选中的元素，设置过渡时长为指定的 *value*(毫秒)。*value* 可以是一个常量也可以是一个函数。如果是函数则会为每个元素立即调用并依次传递当前数据 `d` 以及索引 `i`，函数内部 `this` 指向当前 `DOM` 元素。函数的返回值被用来设置为该元素的过渡时长。如果没有指定过渡时长，则默认为 `250ms`。

如果没有指定 *value* 则返回当前第一个非空元素的过渡时长。在已知过渡仅仅包含一个元素的情况下，这种获取形式通常有用。

<a name="transition_ease" href="#transition_ease">#</a> <i>transition</i>.<b>ease</b>([<i>value</i>]) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/ease.js "Source")

为所有选中的元素指定过渡的 [easing function(缓动函数)](https://github.com/xswei/d3-ease)。*value* 必须为函数。缓动函数会在过渡中的每一帧进行调用，并传递归一化的时间 *t*，处于 [0, 1] 之内；并且必须返回缓动时间 *tʹ*, *tʹ* 通常处于 [0, 1] 之间。一个好的缓动函数在 *t* = 0 时候返回 `0` 并且 *t* = 1 时候返回 `1`。如果没有指定缓动函数则默认为 [d3.easeCubic](https://github.com/xswei/d3-ease#easeCubic)

如果没有指定 *value* 则返回当前过渡中第一个非空元素的缓动函数。在已知过渡只包含一个元素的情况下通常是有用的。

### Control Flow

高级用法，过渡提供了一些自定义控制流的方法。

<a name="transition_on" href="#transition_on">#</a> <i>transition</i>.<b>on</b>(<i>typenames</i>[, <i>listener</i>]) [<>](https://github.com/d3/d3-transition/blob/master/src/transition/on.js "Source")

为每个选中的元素的指定的事件: *typenames* 添加或移除一个 *listener*。*typenames* 为以下字符串类型中的一种:

* `start` - 过渡开始.
* `end` - 过渡结束.
* `interrupt` - 过渡被中断.

参考 [The Life of a Transition](#the-life-of-a-transition) 获取更多信息。要注意的是这些不是由 [*selection*.on](https://github.com/d3/d3-selection#selection_on) 和 [*selection*.dispatch](https://github.com/d3/d3-selection#selection_dispatch) 实现的原生 `DOM` 事件，而是过渡独有的事件。

See [The Life of a Transition](#the-life-of-a-transition) for more. Note that these are *not* native DOM events as implemented by [*selection*.on](https://github.com/d3/d3-selection#selection_on) and [*selection*.dispatch](https://github.com/d3/d3-selection#selection_dispatch), but transition events!

`type` 可以是一个 `.` 与后面跟着的 `name` 组成; 可选的 `name` 允许在同一个类型上注册多个回调。比如 `start.foo` 和 `start.bar`。也可以多个 `typenames` 通过空格隔开，比如 `interrupt end` 或 `start.foo start.bar`。

当指定的过渡事件在选中的节点上触发的时候，对应的 *listener* 将会为被过渡的元素调用，并传递当前数据 `d` 以及索引 `i`，函数内部 `this` 指向当前 `DOM` 元素。事件监听器总是读取元素上绑定的最新的数据，而索引则是在分配监听器的时候就被固定；可以重新注册监听器的方式更新索引。

如果已选中的元素上已经存在相同 *typename* 的事件监听器，则注册新的监听器之间会将旧的移除。如果要移除事件监听器则将 *listener* 为 `null` 即可。移除指定 `name` 所有的事件监听器则将 `typename` 设置为 `.foo` 形式并将 *listener* 设置为 `null`；移除所有没有 `name` 的事件监听器则将 `typename` 设置为 `.` 并将 *listener* 设置为 `null`。

如果没有指定 *listener* 则返回当前第一个非空元素上对应 *typename* 的事件监听器(如果存在的话)。如果元素上注册了多个 *typenames* 则返回第一个匹配的监听器。

<a name="transition_each" href="#transition_each">#</a> <i>transition</i>.<b>each</b>(<i>function</i>) [<>](https://github.com/d3/d3-selection/blob/master/src/selection/each.js "Source")

为过渡中的每个选中的元素调用指定的 *function*，并传递当前元素 `d` 以及索引 `i`，函数内部 `this` 指向当前 `DOM` 元素。这个方法可以被用来为每个选中的元素调用任意代码，并且创建了一个能访问当前元素父节点和子节点数据的上下文。等价于 [*selection*.each](https://github.com/xswei/d3-selection#selection_each)。

<a name="transition_call" href="#transition_call">#</a> <i>transition</i>.<b>call</b>(<i>function</i>[, <i>arguments…</i>]) [<>](https://github.com/d3/d3-selection/blob/master/src/selection/call.js "Source")

调用一次指定的 *function*，并传递可选的参数。返回当前过渡。这等价于手动调用函数，但简化了方法链。例如以可服用函数的形式过渡设置属性:

```js
function color(transition, fill, stroke) {
  transition
      .style("fill", fill)
      .style("stroke", stroke);
}
```

然后可以:

```js
d3.selectAll("div").transition().call(color, "red", "blue");
```

等价于:

```js
color(d3.selectAll("div").transition(), "red", "blue");
```

等价于 [*selection*.call](https://github.com/xswei/d3-selection#selection_call).

<a name="transition_empty" href="#transition_empty">#</a> <i>transition</i>.<b>empt y</b>() [<>](https://github.com/d3/d3-selection/blob/master/src/selection/empty.js "Source")

如果当前过渡中不包含任何非空元素则返回 `true`。等价于 [*selection*.empty](https://github.com/d3/d3-selection#selection_empty).

<a name="transition_nodes" href="#transition_nodes">#</a> <i>transition</i>.<b>nodes</b>() [<>](https://github.com/d3/d3-selection/blob/master/src/selection/nodes.js "Source")

返回当前过渡中所有的非空元素。等价于 [selection*.nodes](https://github.com/xswei/d3-selection#selection_nodes).

<a name="transition_node" href="#transition_node">#</a> <i>transition</i>.<b>node</b>() [<>](https://github.com/xswei/d3-selection/blob/master/src/selection/node.js "Source")

返回当前过渡中第一个非空元素。如果不包含任何元素则返回 `null`。等价于 [selection*.node](https://github.com/xswei/d3-selection#selection_node).

<a name="transition_size" href="#transition_size">#</a> <i>transition</i>.<b>size</b>() [<>](https://github.com/xswei/d3-selection/blob/master/src/selection/size.js "Source")

返回当前过渡中元素总数。等价于 [*selection*.size](https://github.com/xswei/d3-selection#selection_size).

### The Life of a Transition

Immediately after creating a transition, such as by [*selection*.transition](#selection_transition) or [*transition*.transition](#transition_transition), you may configure the transition using methods such as [*transition*.delay](#transition_delay), [*transition*.duration](#transition_duration), [*transition*.attr](#transition_attr) and [*transition*.style](#transition_style). Methods that specify target values (such as *transition*.attr) are evaluated synchronously; however, methods that require the starting value for interpolation, such as [*transition*.attrTween](#transition_attrTween) and [*transition*.styleTween](#transition_styleTween), must be deferred until the transition starts.

Shortly after creation, either at the end of the current frame or during the next frame, the transition is scheduled. At this point, the delay and `start` event listeners may no longer be changed; attempting to do so throws an error with the message “too late: already scheduled” (or if the transition has ended, “transition not found”).

When the transition subsequently starts, it interrupts the active transition of the same name on the same element, if any, dispatching an `interrupt` event to registered listeners. (Note that interrupts happen on start, not creation, and thus even a zero-delay transition will not immediately interrupt the active transition: the old transition is given a final frame. Use [*selection*.interrupt](#selection_interrupt) to interrupt immediately.) The starting transition also cancels any pending transitions of the same name on the same element that were created before the starting transition. The transition then dispatches a `start` event to registered listeners. This is the last moment at which the transition may be modified: after starting, the transition’s timing, tweens, and listeners may no longer be changed; attempting to do so throws an error with the message “too late: already started” (or if the transition has ended, “transition not found”). The transition initializes its tweens immediately after starting.

During the frame the transition starts, but *after* all transitions starting this frame have been started, the transition invokes its tweens for the first time. Batching tween initialization, which typically involves reading from the DOM, improves performance by avoiding interleaved DOM reads and writes.

For each frame that a transition is active, it invokes its tweens with an [eased](#transition_ease) *t*-value ranging from 0 to 1. Within each frame, the transition invokes its tweens in the order they were registered.

When a transition ends, it invokes its tweens a final time with a (non-eased) *t*-value of 1. It then dispatches an `end` event to registered listeners. This is the last moment at which the transition may be inspected: after ending, the transition is deleted from the element, and its configuration is destroyed. (A transition’s configuration is also destroyed on interrupt or cancel.) Attempting to inspect a transition after it is destroyed throws an error with the message “transition not found”.
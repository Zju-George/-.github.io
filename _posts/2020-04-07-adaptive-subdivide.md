---
layout: post
title: "Adaptive Subdivision"
---

<br />

**雕刻操作**就是通过曲面参数化将二维图像映射到三维曲面上。对曲面上每个点，**采样**(双线性插值采样)得到对应的像素值，根据像素值决定是否进行几何操作。eg.，利用二值图像，黑色部分试图保留，白色部分试图镂空，那么曲面上的点采样值小于 0.5 的将会被保留，而大于 0.5 的将会被删去，这样就实现了镂空操作。如下图人脸镂空：

<img src="/assets/face_hollow2.jpg" alt="人脸镂空" width="80%" height="80%" align="center" />

而**细分**是为了雕刻操作前增加模型几何信息，使得雕刻操作后雕刻边缘的锯齿状变小的一个重要操作。

Text can be **bold**, _italic_, or ~~strikethrough~~.

[Link to another page: About]({{ '/about.html' | absolute_url }}).

# 自适应细分的原因

在这里，细分的最基本规则被定义为让一个三角形一分为四。如下图：

<img src="/assets/subdivide.png" alt="细分规则" width="80%" height="80%" align="center" />

所谓**全局细分**就是无差别的对每一个三角形，一分为四。然而全局细分会带来的一个显著问题是：有些三角形的三个顶点采样值都大于 0.5 或者都小于 0.5，也即这个三角形三个顶点都要保留或都要删去，
更直白的说，这个三角形所在位置不在雕刻边缘，所以不需要继续细分；因此全局细分增加了很多~~无用~~没有意义的几何信息。这会使得程序速度变慢，也增加模型的保存大小。

This is a normal paragraph following a header. GitHub is a code hosting platform for version control and collaboration. It lets you and others work together on projects from anywhere.

## [](#header-2)Header 2

> This is a blockquote following a header.
>
> When something is important enough, you do it even if the odds are not in your favor.

### [](#header-3)Header 3

```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require('./lang/' + l)
  return true;
}
```

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### [](#header-4)Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### [](#header-5)Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### [](#header-6)Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

### Large image

![Branching](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```

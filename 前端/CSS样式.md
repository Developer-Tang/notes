## 选择器

### 通用选择器

```css
* {
}
```

### 标签选择器

```html
<p></p>
<style>
    p {
    }
</style>
```

### ID选择器

```html
<p class='black'></p>
<style>
    #nav {
    }
</style>
```

### 类选择器

```html
<p class='black'></p>
<style>
    .black {
    }
</style>
```

### 后代选择器

```html
<ul>
    <li></li>
    <li><a></a></li>
    <li><span><a></a></span></li>
</ul>
<style>
    ul li a {
    }
</style>
```

### 子选择器

```html
<ul>
    <li></li>
    <li><a></a></li> <!-- 生效 -->
    <li><span><a></a></span></li> <!-- 不生效 不能隔标签 -->
</ul>
<style>
    li > a {
    }
</style>
```

### 相邻兄弟选择器

```html
<h1></h1>
<p></p>
<style>
    h1 + p {
    }
</style>
```

### 通用兄弟选择器

```html
<h1></h1>
<a></a>
<p></p>
<style>
    h1 ∼ p {
    }
</style>
```

### 属性选择器

```html
<html>
<input />
<input />
<input type="password" />
<style>
    p[type='password'] {
    }
</style>
</html>
```

## 长度单位

#React Native 中的样式管理 Managing Styles in React Native

原文链接:[https://medium.com/@tommylackemann/managing-styles-in-react-native-3546d3482d73](https://medium.com/@tommylackemann/managing-styles-in-react-native-3546d3482d73)

[React Native](https://facebook.github.io/react-native/) 是一个框架, 它可以让那些熟悉JavaScript的全栈工程师开发出功能丰富的安卓或苹果设备的应用.例如 Facebook, Instagram 和 Airbnb这些公司都正在利用React Nativet 开发APP,你只需要一套代码和修改少量配置就可分别打包出两个平台上运行的APP.

React Native不会强迫您学习Objective-C，Swift或Java，这对于想要构建与市场上最好的APP相似的应用程序的Web开发人员来说非常适合.然而它的学习曲线还是有一点陡的,而且还没有一个绝对正确的构建React Native APP的方式.这篇简单的文章介绍了从传统的CSS样式转换为React Native StyleSheets的最佳实践指南.

### StyleSheets 不是 CSS

StyleSheets同你以前所写的CSS既像又不像.对于初学者,它使用纯粹JavaScript的方式会强迫你重构你的肌肉记忆.例如,在CSS中你可能会这样写:

```
.header {
  padding: 20px;
  margin: 15px 0;
  position: absolute;
}

```
在React Native中,没有像像素(pixels)或类名(classes)这样一类东西,至少是没有预置的.相反,我们将尺寸写入某一“单位”，最终根据屏幕的像素密度转换为像素.如果我们想在React Native写出与上面的header相同的StyleSheet,将会像下面这样.

```
import { StyleSheet } from 'react-native';

```

```
export default StyleSheet.create({
  header: {
    padding: 20,
    marginTop: 15,
    marginBottom: 15,
    position: 'absolute',
  },
});

```

需要注意的是`absolute`被写作了String而不像传统的CSS.另外一个重点是在CSS中的语法糖在React Native中是没有的.在React Native中声明样式属性需要表意明确.当我们些JavaScript时,我们不能写出诸如`margin: 15 0`这样的语法.相反,我们将其分解为不同驼峰命名的属性(大多数情况).
StyleSheets 被导入我们的组件同时通过声明`style`属性来被特定组件使用.我们将在稍后看到一个例子.

为了理解StyleSheets !== CSS,我们将讨论如何编码我们的应用,以充分利用这种风格.

### 复用组件,不仅仅是样式
在构建网络应用时，我们有无数的方式组织我们的CSS，无论是使用Sass，Less，PostCSS等工具。这使我们可以方便地写出层次分明的结构.最后我们的工具将全部CSS构建到一起,为整个应用提供合并后的样式.

在React Native中,我们必须以略微不同的方式考虑样式写法.我们想继续写小块小块的样式来代替写一个大的StyleSheet.这就意味着所有可复用的内容都应该是一个组件.让我们来看一个例子:
在传统的网络应用中,我们可能会写下面的class来定义buttons


```
.button {
  background-color: #111;
  color: #fff;
  padding: 15px;
  border-radius: 5px;
}

```

在React Native中,如果不写一个大的StyleSheet(这不是一个好方式)是没有更好办法共享样式的,但我们仍然想将其分解至他自己的组件中,比如一个`Button`组件.

首先,我们需要新建两个文件,`components/Button/index.js` 和 `components/Button/styles.js`.文件内容如下:

```
# components/Button/index.js

```

```
import React from 'react';
import {
  TouchableHighlight,
  Text,
} from 'react-native';
import styles from './styles';

```

```
const Button = () => (
  <TouchableHighlight style={styles.container}>
    <Text style={styles.button}>Click Me</Text>
  </TouchableHighlight>
)

```

```
export default Button;

```

```
# components/Button/styles.js

```

```
import { StyleSheet } from 'react-native';

```

```
export default StyleSheet.create({
  container: {
    borderRadius: 5,
  },
  button: {
    backgroundColor: '#111',
    color: '#fff',
    borderRadius: 5,
    padding: 15,
  },
});

```

现在,无论何时我们想在应用中要引入一个button,我们只需简单的导入(import)这个`Button`组件即可,它的必要样式已经写好.

接下来再谈谈上面的`container` 和 `button`

### 黑边框

React Native中的`TouchableHighlight`组件是一个特殊的组件，可以为用户提供他们刚刚执行的轻按（点击）事件的反馈.没有它,用户点击您的按钮后,将不会收到任何反馈(尽管它仍然可以工作).

`TouchableHighlight`会有一些奇怪的透明度方面的问题.如果我们不引入`borderRadius: 5`,当我们点击按钮时将看到它周围有一些丑陋的黑边.你想要确保无论何时使用`TouchableHighlight`时,都可以明确展示组件中的透明或不透明的元素.为了解决点击按钮时周围的黑框问题,我们简单地给`TouchableHighlight`组件一个合适的`border radius`来确保里面没有透明像素.

### 定义通用样式

有可能你会有一些相同的样式,比如颜色,字体大小,内边距等等.对于这些在多个组件中需要共享的样式,较好的做法是写一个能被其他样式文件引用的通用样式.

这里有一个通用样式文件的示例:
```
# styles/common.js

```

```
export const COLOR_PRIMARY = '#58C9B9';
export const COLOR_SECONDARY = '#111';
export const FONT_NORMAL = 'OpenSans-Regular';
export const FONT_BOLD = 'OpenSans-Bold';
export const BORDER_RADIUS = 5;

```

现在我们写样式时就可以很容易地使用这些通用样式.

```
# components/Button/styles.js

```

```
import { StyleSheet } from 'react-native';
import { COLOR_PRIMARY, BORDER_RADIUS } from './../styles/common';

```

```
export default StyleSheet.create({
  container: {
    borderRadius: BORDER_RADIUS,
  },
  button: {
    backgroundColor: COLOR_PRIMARY,
    borderRadius: BORDER_RADIUS,
  },
});

```

### 结束语
React Native是一种构建Android和iPhone设备上的原生APP的一种迅速和容易的方式.

在开始编写组件代码前,通过做一些规划和组织减少不必要的重复代码,会有利于长期的维护和后续的修改.一个通用的样式文件(StyleSheet)会变得很有用,特别是当产品想要修改APP的字体或颜色时.

通过遵循这些做法，您将有一个精简和可管理的React Native应用程序，任何开发人员都可以维护.
                
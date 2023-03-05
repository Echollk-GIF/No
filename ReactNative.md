[TOC]

# Tips

RN没有div h1等普通HTML元素，常用View Text

创建项目 expo init myproject

插件快捷命令:rnf(react native function)、rnfs(带StyleSheet)

handle函数不要加()，否则会在渲染时自动执行

# 创建项目

```js
//expo
npm install -g expo-cli
expo init projectName
npm start

//native
brew install watchman
react-native init projectName
```

# CSS

RN中的样式与CSS的不同：没有继承性（RN中的继承只发生在Text组件上）、样式名采用小驼峰命名、所有尺寸都没有单位（在安卓上字体解释为单位为sp、在ios上字体被解释为pt）、有些特殊的样式名(marginHorizontal水平外边距、marginVertical垂直外边距)

## StyleSheet

可以通过在style属性中调用StyleSheet声明样式

```react
import { StyleSheet, Text, View } from 'react-native'

export default function App () {
  return (
    <View style={styles.container}>
      <Text>Open up App.js to start working on your app!</Text>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
})
```

## 变换

```react
<Text style={{transform:[
    {translateY:300},
    {scale:3}
  ]}}></Text>
```

## Dimensions

Dimensions可以用来获取屏幕的尺寸

这个属性只有初始化时会获取，旋转屏幕不会重新获取。可以在useEffect中通过Dimensions.addEventListener('change',updateLayout)方式，通过updateLauyout函数重新获取宽度和高度，并在return中取消监听Dimensions.removeEventListener('change',updateLayout)

```react
import { StatusBar } from 'expo-status-bar'
import { StyleSheet, Text, View, Dimensions } from 'react-native'
export default function App () {
  const windowWidth = Dimensions.get('window').width
  const windowHeight = Dimensions.get('window').height
  return (
    <View style={styles.container}>
      <Text>{windowWidth}</Text>
      <Text>{windowHeight}</Text>
      <StatusBar />
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  }
})
```

## Flex

**容器属性**

flexDirection：RN里flex的默认方向是从上到下即'colume'，Web CSS中默认为row

alignItems:RN里默认为stretch(弹性元素被在侧轴方向被拉伸到与容器相同的高度或宽度)，Web CSS中默认为flex-start

flex:相比Web CSS的flex接受多参数，如flex:2 2 10%，RN中flex只接受一个参数

RN中的flex不支持align-content、flex-basis、order、flex-grow、flex-flow、flex-shrink

flexWrap:默认nowrap、wrap

**子视图属性**

alignSelf：定义对齐方式（注意：该属性可重写容器属性的alignItems属性），默认为auto（继承父元素的align-items属性，如果没有父容器则为stretch，stretch元素被拉伸以适应容器、center元素中心、flex-start位于容器开头、flex-end位于容器结尾）

flex:定义可伸缩元素的能力，默认为0

**flex其他属性**

borderBottom(RIght、Top、Left)Width 底部、右边、顶部、左边的边框高度

borderWidth 边框高度

marginVertical上下外边距

marginHorizontal左右外边距

## Margin

marginVertical只设置上下margin

# 核心组件

## Text

属性：

numberOfLines最多显示几行

## Button && Alert

总结：

Button常用属性：title、onPress、color（注意不要以style定义button的颜色）

Alert常用属性：Alert.alert() 可以再里面依次写警告标题、内容、以及N个按钮

```react
import { View, StyleSheet, Button, Alert } from 'react-native'
export default function App () {
  const handlePressTwo = () => {
    Alert.alert(
      "警告标题",
      "警告内容",
      [
        {
          text: "取消",
          onPress: () => {
            console.log('Cancel')
          },
          style: 'cancel'
        },
        {
          text: "确定",
          onPress: () => {
            console.log('OK')
          },
          style: "default"
        }
      ]
    )
  }
  const handlePressThree = () => {
    Alert.alert(
      "更新提示",
      "发现新版本，是否现在更新",
      [
        {
          text: "稍后再试",
          onPress: () => {
            console.log('稍后提醒我')
          },
        },
        {
          text: "取消",
          onPress: () => {
            console.log('Cancel')
          },
          style: 'cancel'
        },
        {
          text: "确定",
          onPress: () => {
            console.log('OK')
          },
          style: "default"
        },
      ]
    )
  }
  return (
    <View style={styles.container}>
      <Button
        title='第一个按钮'
        color={'red'}
        onPress={() => {
          Alert.alert('我是第一个按钮')
        }}
      ></Button>
      <Button
        title='第两个按钮'
        color={'red'}
        onPress={handlePressTwo}
      ></Button>
      <Button
        title='第三个按钮'
        color={'red'}
        onPress={handlePressThree}
      ></Button>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'space-around',
    alignItems: 'center'
  }
})
```

## StatusBar && Switch

```react
import { StyleSheet, View, StatusBar, Switch } from 'react-native'
import React, { useState } from 'react'

export default function App () {
  const [hideStatus, setHideStatus] = useState()
  return (
    <View style={[styles.container]}>
      <StatusBar
        hidden={hideStatus}
        barStyle={'dark-content'} />
      <Switch
        trackColor={{ false: 'red', true: 'green' }}
        thumbColor={'blue'}
        value={hideStatus}
        onValueChange={() => {
          setHideStatus(!hideStatus)
        }} />
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  }
})
```

总结：

StatusBar常用属性：hidden、backgroundColor、barStyle

Switch常用属性：trackColor背景色、thumbColor小圆点颜色、value、onValueChange

## ActivityIndicator && Platform

```react
import { StyleSheet, View, ActivityIndicator, Platform } from 'react-native'
import React from 'react'

export default function App () {
  if (Platform.OS === 'android') {
    alert('安卓')
  } else if (Platform.OS === 'ios') {
    alert('IOS')
  }
  return (
    <View style={styles.container}>
      <ActivityIndicator
        color={'red'}
        size={'large'} />
      {/* 数字指定大小只在安卓端有效 */}
      <ActivityIndicator
        color={'red'}
        size={70} />
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center'
  }
})

//也可以设置baseStyle、IOSStyle、AndroidStyle,在style中设置
style={{
       ...styles.baseStyle,
       ...Platform.select({
       ios:styles.IOSStyle,
       android:styles.AndroidStyle
      })
      }}
```

总结：

ActivityIndicator常用属性：color、size

Platform不是组件，可以用来判断当前系统

## Image

```react
import { StyleSheet, Text, View, Image, Dimensions } from 'react-native'
import React from 'react'

export default function App () {
  return (
    <View style={styles.container}>
      <Text>App</Text>
      <Image style={styles.itemImage}
        //本地图片
        source={require('./assets/favicon.png')}></Image>
      <Image source={{
        //网络图片
        uri: 'http://www.xxx.com/logo.png'
      }}></Image>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  itemImage: {
    height: Dimensions.get('window').height / 5,
    width: Dimensions.get('window').width / 5,
    marginVertical: 20
  }
})
```

总结：

Image常用属性：style、source(本地图片用require，线上图片用{uri:xxxx})

## ImageBackGround

> 一个可以使用图片当作背景的容器,相当于以前的 `div+背景图片`

```jsx
  <ImageBackground source={...} style={{width: '100%', height: '100%'}}>
    <Text>Inside</Text>
  </ImageBackground>
```

## TextInput

总结：

TextInput常用属性要根据输入框类型，具体可以看示例

TextInput一般需要有一个宽度才能显示

常用属性：style、placeholder、value、onChangeText、secureTextEntry（密码）

```react
import { StyleSheet, View, TextInput, Dimensions, Button } from 'react-native'
import React, { useState } from 'react'

export default function App () {
  const [username, setUsername] = useState('')
  const [password, setPassword] = useState()
  const [phone, setPhone] = useState('')
  const [text, setText] = useState()
  const doLogin = () => {
    alert(username + password + text)
  }
  return (
    <View style={styles.container}>
      <TextInput
        style={styles.input}
        placeholder="请输入用户名"
        value={username}
        onChangeText={(val) => {
          setUsername(val)
        }} />
      <TextInput
        style={styles.input}
        placeholder="请输入密码"
        value={password}
        secureTextEntry={true}
        onChangeText={(val) => {
          setPassword(val)
        }} />
      <TextInput
        style={styles.input}
        placeholder="请输入手机号"
        keyboardType='number-pad'
        value={phone}
        onChangeText={(val) => {
          setPhone(val)
        }} />
      <TextInput
        style={styles.input}
        placeholder="请输入自我介绍"
        multiline={true}
        numberOfLines={5}
        textAlignVertical="top"
        value={text}
        onChangeText={(val) => {
          setText(val)
        }} />
      <View>
        <Button
          title='登录'
          onPress={doLogin}></Button>
      </View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  input: {
    width: Dimensions.get('window').width - 20,
    height: 40,
    margin: 10,
    borderWidth: 1,
    borderColor: 'red',
    paddingHorizontal: 5
  }
})
```

## Touchable

自己封装的组件没办法直接设置点击后的回调，就可以用touchable包一层再设置

TouchableHighlight 触碰后，高亮显示

TouchableOpacity 触碰后，透明度降低（模糊显示）

TouchableWithoutFeedback 触碰后，无任何相应

TouchableNativeFeedback（安卓系统且Platfrom.Version>=21）

```react
import {
  StyleSheet, View, Text,
  TouchableHighlight,
  TouchableOpacity,
  TouchableWithoutFeedback
} from 'react-native'
import React from 'react'

export default function App () {
  return (
    <View style={styles.container}>
      <TouchableHighlight
        onPress={() => console.log('触碰高亮显示')}>
        <View style={styles.item}>
          <Text>触碰高亮</Text>
        </View>
      </TouchableHighlight>
      <TouchableOpacity
        activeOpacity={0.5}
        onPress={() => console.log('触碰透明度变化')}>
        <View style={styles.item}>
          <Text>触碰透明度变化</Text>
        </View>
      </TouchableOpacity>
      <TouchableWithoutFeedback
        onPress={() => console.log('触碰无响应')}>
        <View style={styles.item}>
          <Text>触碰无响应</Text>
        </View>
      </TouchableWithoutFeedback>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  item: {
    marginBottom: 20,
    padding: 10,
    borderWidth: 1,
    borderColor: 'red'
  }
})
```

## ScrollView && SafeAreaView

SafeAreaView相比普通的View会避免刘海屏问题

ScrollView在安卓下会有一个显示不全的问题，可以在ScrollView内部的最下方加一个View，根据Platform来给iOS0高度，安卓根据丢失文本高度给高度

```react
import {
  StyleSheet, View, Text,
  ScrollView
} from 'react-native'
import React from 'react'

export default function App () {
  return (
    <View>
      <Text>111</Text>
      <Text>111</Text>
      <Text>111</Text>
      <ScrollView
        horizontal={true}>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
        <Text style={styles.nav}>新闻</Text>
      </ScrollView>
      <ScrollView
        contentContainerStyle={{ margin: 30 }}
        showsVerticalScrollIndicator={false}>
        <Text style={styles.text}>
          aaaaaa
          aaaaaa
          aaaaaa
          aaaaaa
          aaaaaa
          aaaaaa
          aaaaaa
        </Text>
      </ScrollView>
    </View>
  )
}

const styles = StyleSheet.create({
  text: {
    fontSize: 130
  },
  nav: {
    margin: 10,
    height: 50,
    fontSize: 20
  }
})
```

## SectionList

```react
import React, { useState } from "react"
import { StyleSheet, Text, View, SafeAreaView, SectionList, StatusBar } from "react-native"

const DATA = [
  {
    title: "Main dishes",
    data: ["Pizza", "Burger", "Risotto"]
  },
  {
    title: "Sides",
    data: ["French Fries", "Onion Rings", "Fried Shrimps"]
  },
  {
    title: "Drinks",
    data: ["Water", "Coke", "Beer"]
  },
  {
    title: "Desserts",
    data: ["Cheese Cake", "Ice Cream"]
  }
]

const Item = ({ title }) => (
  <View style={styles.item}>
    <Text style={styles.title}>{title}</Text>
  </View>
)

const App = () => {
  const [isFresh, setIsRefresh] = useState(false)
  const loadData = () => {
    setIsRefresh(true)
    //模拟请求数据
    setTimeout(() => {
      setIsRefresh(false)
      alert('下拉刷新')
    }, 2000)
  }
  return (
    <SafeAreaView style={styles.container}>
      <SectionList
        sections={DATA}
        keyExtractor={(item, index) => item + index}
        renderItem={({ item }) => <Item title={item} />}
        renderSectionHeader={({ section: { title } }) => (
          <Text style={styles.header}>{title}</Text>
        )}
        //项目之间的分隔符
        ItemSeparatorComponent={() => {
          return <View style={{ borderBottomWidth: 1, borderBottomColor: 'red' }}></View>
        }}
        //列表数据为空时，展示的组件
        ListEmptyComponent={() => {
          return <Text>空空如也</Text>
        }}
        //下拉刷新
        refreshing={isFresh}
        onRefresh={loadData}
        //上拉加载
        onEndReachedThreshold={0.1}//声明触底的比率，0.1意思是距离地底部还剩10%
        onEndReached={() => {
          alert('到底了')
        }}
        //声明列表的头部
        ListHeaderComponent={() => {
          return <Text>头部</Text>
        }}
        ListFooterComponent={() => {
          return <Text>没有更多了</Text>
        }}
      />
    </SafeAreaView>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingTop: StatusBar.currentHeight,
    marginHorizontal: 16
  },
  item: {
    backgroundColor: "#f9c2ff",
    padding: 20,
    marginVertical: 8
  },
  header: {
    fontSize: 32,
    backgroundColor: "#fff"
  },
  title: {
    fontSize: 24
  }
})

export default App
```

## FastList

ScrollView会简单粗暴地把所有子元素一次性全部渲染出来。

FlatList会惰性渲染子元素，只在它们将要出现在屏幕中时开始渲染。

此外FlatList还可以方便地渲染行间分隔线，支持多列布局，无限滚动加载等等。

常用属性：

- data列表的数据源
- renderItem则从数据源中逐个解析数据，然后返回一个设定好格式的组件来渲染
- keyExtractor={item => item.id} 密钥提取器（旧版本需要这样做）
- numColumns={2} 指定列数
- onEndReached={()=>{}}当加载到底部时触发回调
- onEndReachedThreshold={0.2}具体底部多远触发上面的回调

```react
import React, { useState } from 'react'
import { SafeAreaView, View, FlatList, StyleSheet, Text, StatusBar, TouchableOpacity } from 'react-native'

const DATA = [
  {
    id: 'bd7acbea-c1b1-46c2-aed5-3ad53abb28ba',
    title: 'First Item',
  },
  {
    id: '3ac68afc-c605-48d3-a4f8-fbd91aa97f63',
    title: 'Second Item',
  },
  {
    id: '58694a0f-3da1-471f-bd96-145571e29d72',
    title: 'Third Item',
  },
  {
    id: '58694a0f-3da1-471f-bd96-145571e29d73',
    title: 'Third Item1',
  },
  {
    id: '58694a0f-3da1-471f-bd96-145571e29d74',
    title: 'Third Item2',
  },
  {
    id: '58694a0f-3da1-471f-bd96-145571e29d75',
    title: 'Third Item3',
  },
  {
    id: '58694a0f-3da1-471f-bd96-145571e29d76',
    title: 'Third Item4',
  },
  {
    id: '58694a0f-3da1-471f-bd96-145571e29d77',
    title: 'Third Item5',
  },
  {
    id: '58694a0f-3da1-471f-bd96-145571e29d78',
    title: 'Third Item6',
  },
  {
    id: '58694a0f-3da1-471f-bd96-145571e29d79',
    title: 'Third Item7',
  },
  {
    id: '18694a0f-3da1-471f-bd96-145571e29d79',
    title: 'Third Item8',
  },
  {
    id: '28694a0f-3da1-471f-bd96-145571e29d79',
    title: 'Third Item9',
  },
]

const Item = ({ title }) => {
  return (
    <View style={styles.item}>
      <Text style={styles.title}>{title}</Text>
    </View>
  )
}

const App = () => {
  const [isFresh, setIsRefresh] = useState(false)
  const [selectedId, setSelesctedId] = useState(null)
  const loadData = () => {
    setIsRefresh(true)
    //模拟请求数据
    setTimeout(() => {
      setIsRefresh(false)
      alert('下拉刷新')
    }, 2000)
  }
  const renderItem = ({ item }) => {
    const backgroundColor = item.id === selectedId ? '#dfb' : '#f9c2ff'
    return (
      <TouchableOpacity
        onPress={() => {
          setSelesctedId(item.id)
        }}
        style={[styles.item, { backgroundColor }]}>
        <Item title={item.title} />
      </TouchableOpacity>
    )
  }
  return (
    <SafeAreaView style={styles.container}>
      <FlatList
        data={DATA}
        renderItem={renderItem}
        keyExtractor={item => item.id}
        //水平布局模式
        // horizontal={true}
        //初始滚动索引
        initialScrollIndex={3}
        //指定初始渲染数据的数量，一般数量要填满一屏幕（有点类似懒加载）
        initialNumToRender={4}
        //指定列数，数据项必须等高（无法支持瀑布流）
        numColumns={2}
        //列表反转  
        // inverted={true}
        extraData={selectedId}
        //项目之间的分隔符
        ItemSeparatorComponent={() => {
          return <View style={{ borderBottomWidth: 1, borderBottomColor: 'red' }}></View>
        }}
        //列表数据为空时，展示的组件
        ListEmptyComponent={() => {
          return <Text>空空如也</Text>
        }}
        //下拉刷新
        refreshing={isFresh}
        onRefresh={loadData}
        //上拉加载
        onEndReachedThreshold={0.1}//声明触底的比率，0.1意思是距离地底部还剩10%
        onEndReached={() => {
          alert('到底了')
        }}
        //声明列表的头部
        ListHeaderComponent={() => {
          return <Text>头部</Text>
        }}
        ListFooterComponent={() => {
          return <Text>没有更多了</Text>
        }}
      />
    </SafeAreaView>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: StatusBar.currentHeight || 0,
  },
  item: {
    backgroundColor: '#f9c2ff',
    padding: 20,
    marginVertical: 8,
    marginHorizontal: 16,
  },
  title: {
    fontSize: 32,
  },
})

export default App
```

## Animated

1.RN中可以直接使用的动画组件：

Animated.View

Animated.Text

Animated.ScrollView

Animated.Image

2.创建动画的步骤：

创建初始值 

- Animated.Value()单个值
- Animated.ValueXY()向量值

将初始值绑定到动画组件上

- 一般将其绑定到某个样式属性下，比如opacity、translate

通过动画类型API，一帧一帧地更改初始值

- Animated.decay()加速效果
- Animated.spring()弹跳效果
- Animated.timing()时间渐变效果

```react
import React, { useRef } from "react"
import { Animated, Text, View, StyleSheet, Button } from "react-native"

const App = () => {
  // fadeAnim will be used as the value for opacity. Initial Value: 0
  const fadeAnim = useRef(new Animated.Value(0)).current

  const fadeIn = () => {
    // Will change fadeAnim value to 1 in 5 seconds
    Animated.timing(fadeAnim, {
      toValue: 1,
      duration: 5000,
      //启用原生方式渲染动画，渲染效率更高
      useNativeDriver: true
    }).start(() => {
      //这里面是动画结束之后的回调函数
      alert('显示出来了')
    })
  }

  const fadeOut = () => {
    // Will change fadeAnim value to 0 in 5 seconds
    Animated.timing(fadeAnim, {
      toValue: 0,
      duration: 5000,
      //启用原生方式渲染动画，渲染效率更高
      useNativeDriver: true
    }).start(() => {
      alert('我消失了')
    })
  }

  return (
    <View style={styles.container}>
      <Animated.View
        style={[
          styles.fadingContainer,
          {
            opacity: fadeAnim // Bind opacity to animated value
          }
        ]}
      >
        <Text style={styles.fadingText}>Fading View!</Text>
      </Animated.View>
      <View style={styles.buttonRow}>
        <Button title="Fade In" onPress={fadeIn} />
        <Button title="Fade Out" onPress={fadeOut} />
      </View>
    </View>
  )
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center"
  },
  fadingContainer: {
    paddingVertical: 8,
    paddingHorizontal: 16,
    backgroundColor: "powderblue"
  },
  fadingText: {
    fontSize: 28,
    textAlign: "center",
    margin: 10
  },
  buttonRow: {
    flexDirection: "row",
    marginVertical: 16
  }
})

export default App
```

## KeyboardAvoidingView

使软键盘不会覆盖输入

常用属性：behavior（例如position适合ios、height、padding适合安卓）、keyboardVerticalOffset（软键盘输入时重新定位）

# 第三方组件

## react-native-webview

yarn add react-native-webview

可以将网页嵌入到当前页面

```react
import React, { Component } from 'react';
import { StyleSheet, Text, View } from 'react-native';
import { WebView } from 'react-native-webview';

// ...
class MyWebComponent extends Component {
  render() {
    return <WebView source={{ uri: 'https://reactnative.dev/' }} />;
  }
}
```

## Picker(下拉框)

yarn add @react-native-picker/picker

```react
const pickerRef = useRef();

function open() {
  pickerRef.current.focus();
}

function close() {
  pickerRef.current.blur();
}

return <Picker
  ref={pickerRef}
  selectedValue={selectedLanguage}
  onValueChange={(itemValue, itemIndex) =>
    setSelectedLanguage(itemValue)
  }>
  <Picker.Item label="Java" value="java" />
  <Picker.Item label="JavaScript" value="js" />
</Picker>
```

## 轮播图(react-native-swiper)

```react
import React, { Component } from 'react'
import { AppRegistry, StyleSheet, Text, View } from 'react-native'

import Swiper from 'react-native-swiper'

const styles = StyleSheet.create({
  wrapper: {},
  slide1: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#9DD6EB'
  },
  slide2: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#97CAE5'
  },
  slide3: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#92BBD9'
  },
  text: {
    color: '#fff',
    fontSize: 30,
    fontWeight: 'bold'
  }
})

export default class SwiperComponent extends Component {
  render() {
    return (
      <Swiper style={styles.wrapper} showsButtons={true}>
        <View style={styles.slide1}>
          <Text style={styles.text}>Hello Swiper</Text>
        </View>
        <View style={styles.slide2}>
          <Text style={styles.text}>Beautiful</Text>
        </View>
        <View style={styles.slide3}>
          <Text style={styles.text}>And simple</Text>
        </View>
      </Swiper>
    )
  }
}

AppRegistry.registerComponent('myproject', () => SwiperComponent)
```

## AsyncStorage

yarn add @react-native-async-storage/async-storage

```react
import {AsyncStorage} from 'react-native';

//Persisting data:
_storeData = async () => {
  try {
    await AsyncStorage.setItem(
      '@MySuperStore:key',
      'I like to save it.',
    );
  } catch (error) {
    // Error saving data
  }
};

//Fetching data:
_retrieveData = async () => {
  try {
    const value = await AsyncStorage.getItem('TASKS');
    if (value !== null) {
      // We have data!!
      console.log(value);
    }
  } catch (error) {
    // Error retrieving data
  }
};
```

封装storage.js

```react
import AsyncStorage from 'react-native-async-storage/async-storage'

class Storage{
  static set (key,value){
    return AsyncStorage.setItem(key,JSON.stringify(value))
  }
  
  static get(key){
    return AsyncStorage.getItem(key).then(value=>{
      if(value && value !== ''){
        const jsonValue = JSON.parse(value)
        return jsonValue
      }
    }).catch(()=>null)
  }
  
  static update(key,newValue){
    return AsyncStorage.getItem(key).then(oldValue=>{
      newValue = typeof newValue === 'string' ? newValue : Object.assign({},oldValue,newValue)
      return AsyncStorage.setItem(key,JSON.stringify(newValue))
    })
  }
  
  static delete(key){
    return AsyncStorage.removeItem(key)
  }
  
  static clear(){
    return AsyncStorage.clear()
  }
}

export default Storage
```

## Geolocation(定位)

yarn add @react-native-community/geolocation

```react
import Geolocation from '@react-native-community/geolocation';
import AsyncStorage from 'react-native-async-storage/async-storage'

useEffect(()=>{
  const location = storage.get('coords')
  
  //如果本地存储中没有位置信息则获取地理位置
  if(location === undefined || location === ''){
    Geolocation.getCurrentPosition(
  info => {
    console.log(info)
    //获取地理位置成功后保存下来
    AsyncStorage.setItem('coords',JSON.stringify(info.coords))
  },
  error => Alert.alert('报错',JSON.stringify(error)),
  {
    timeout:20000
  }
);
  }
},[])

```

## 轮播图(react-native-snap-carousel)

```
yarn add react-native-snap-carousel
```

```react
import Carousel from 'react-native-snap-carousel';

export class MyCarousel extends Component {

    _renderItem = ({item, index}) => {
        return (
            <View style={styles.slide}>
                <Text style={styles.title}>{ item.title }</Text>
            </View>
        );
    }

    render () {
        return (
            <Carousel
              ref={(c) => { this._carousel = c; }}
              data={this.state.entries}
              renderItem={this._renderItem}
              sliderWidth={sliderWidth}
              itemWidth={itemWidth}
            />
        );
    }
}
```

## 图标(react-native-vector-icons)

# 路由

## 3.X版本

### 基础路由

```js
npm install react-navigation
//如果您使用的是 Expo，为确保获得您应该运行的库的兼容版本：
npx expo install react-native-gesture-handler react-native-reanimated
//否则，直接使用 yarn 或 npm 即可：
npm install react-native-gesture-handler react-native-reanimated
```

```react
//npm install --save react-navigation-stack
import { createStackNavigator, createAppContainer } from 'react-navigation';

import CategoriesScreen from '../screens/CategoriesScreen';
import CategoryMealsScreen from '../screens/CategoryMealsScreen';
import MealDetailScreen from '../screens/MealDetailScreen';

const MealsNavigator = createStackNavigator({
  Categories: CategoriesScreen,
  CategoryMeals: {
    screen: CategoryMealsScreen，
  },
  MealDetail: MealDetailScreen
},{
  defaultNavigationOptions:{
      headerStyle:{
    		backgroundColor:'black'
  		},
  		headerTintColor:'white'
  }
});

export default createAppContainer(MealsNavigator);
```

导航方式

```react
props.navigation.navigate({routeName:'CategoryMeals'},params:{categoryId:'1'})
props.navigation.navigate('CategoryMeals',{categoryId:'1'})
//在目标页面中以const id = props.navigation.getParam('categoryId')的形式获取或者
CategoriesScreen.navigationOptions = navigationData =>{
  const id = navigationData.navigation.getParam('categoryId')
  const selectedCategory = CATEGORIES.find(cat=>cat.id === catId);
  return {
    headerTitle:selectedCategory.title
  }
}

props.navigation.push('CategoryMeals')//push可以重复进入同一路由页面
props.navigation.goBack();
props.navigation.pop();//pop只有在堆栈导航器中能用,goBack也可以在其他导航器中使用
props.navigation.popToTop();
```

标题

```js
//CategoriesScreen.js
CategoriesScreen.navigationOptions = {
	headerTitle:'Meal Categories',
  headerStyle:{
    backgroundColor:'black'
  },
  headerTintColor:'white'
}
```

底部导航器

createBottomTabNavigator

```react
tabBarOptions: {
  activeTintColor: '#e91e63',
  labelStyle: {
    fontSize: 12,
  },
  style: {
    backgroundColor: 'blue',
  },
}
const MealsFavTabNavigator = createBottomTabNavigator({
  Meals:mealsNavigator,
  Favorites:FavoritesScreen
},{
  tabBarOptions:tabBarOptions
})
```

抽屉导航器

createDrawerNavigator

## 5.X版本

### 非expo版本

```js
npm install @react-navigation/native@^5.x//核心包
npm install react-native-reanimated react-native-gesture-handler react-native-screens react-native-safe-area-context @react-native-community/masked-view
//动画库 手势库  实现安卓和ios的原生组件 异形屏适配 堆栈式导航器依赖的库（不会显示使用）
```

#### 堆栈式导航器

```
npm install @react-navigation/stack@^5.x
```

```react
import { NavigationContainer } from '@react-navigation/native';//所有导航器都要放在这个NavigationContainer中
import { createStackNavigator } from '@react-navigation/stack';//引入堆栈导航器
//createStackNavigator会返回一个对象包含两个属性，Navigator组件和Screen组件

class Navigator extends React.Component{
  render(){
    return(
      <NavigationContainer>
        <Stack.Navigator 
          screenOptions={{
            headerTitleAlign:'center'，
            headerTitle：'首页',
            headerStyleInterpolator:headerStyleInterpolators.forUIKit
            cardStyleInInterpolator:cardStyleInInterpolators.forHorizontalIOS
            gestureEnabled:true//开启安卓端的手势系统
            gestureDirection:'horizontal'//手势方向
            headerStatusBarHeight:StatusBar.currentHeight
            headerStyle:{
              ...Platform.select({
                android:{
                  elevation:0,
                  borderBottomWidth:StyleSheet.hairlineWidth
                },
                ios:{}
              })
            }
          }}
          headerMode="float"
          >
          <Stack.Screen name="Home" component={Home} options={{headerTitleAlign:'left'}}/>
          //在Screen中设置options的优先级比Navigator中要高
          <Stack.Screen name="Detail" component={Detail}/>
        </Stack.Navigator>
      </NavigationContainer>
    )
  }
}

//跳转路由传参
props.navigation.navigate("home",{
  id:100
})
//接受参数（类组件）
this.props.route.params.id
```

#### 标签导航器

```
yarn add @react-navigation/bottom-tabs
```

```react
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Tab = createBottomTabNavigator()
class BottomTabs extends React.Component{
  render(){
    return(
      <NavigationContainer>
        <Tab.Navigator tabBarOptions={{
            activeTintColor:'#ffffff'
          }}>
          <Tab.Screen 
            name="Home"
            component={Home}
            options={{
              tabBarLabel:'首页',
              tabBarIcon:({color,size})=><Icon name="xx" size={size} color={color}></Icon>
            }}
            />
          <Tab.Screen name="Detail" component={Detail}/>
        </Tab.Navigator>
      </NavigationContainer>
    )
  }
}

const {navigation} = this.props;
navigation.setOptions({
  headerTitle:'我听'
})
```

#### 顶部标签导航器

```js
yarn add @react-navigation/material-top-tab react-native-tab-view
```

```react
import { NavigationContainer } from '@react-navigation/native';
import { createMaterialTopTabNavigator } from '@react-navigation/material-top-tab';

const Tab = createMaterialTopTabNavigator()
class TopTabs extends React.Component{
  render(){
    return(
      <NavigationContainer>
        <Tab.Navigator
          lazy={true}
          tabBarOptions={{
            scrollEnabled:true//滑动还是均分
            tabStyle:{
              width:80,
            },
            indicatorStyle:{
              height:4,
              width:20,
              marginLeft:30,
              borderRadius:2,
              backfroundColor:'#f86442'
            }
          }}>
          <Tab.Screen 
            name="Home"
            component={Home}
            options={{
              tabBarLabel:'首页',
              tabBarIcon:({color,size})=><Icon name="xx" size={size} color={color}></Icon>
            }}
            />
          <Tab.Screen name="Detail" component={Detail}/>
        </Tab.Navigator>
      </NavigationContainer>
    )
  }
}
```


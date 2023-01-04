创建项目 react-native init myproject

插件快捷命令:rnf(react native function)

RN中的样式与CSS的不同：没有继承性（RN中的继承只发生在Text组件上）、样式名采用小驼峰命名、所有尺寸都没有单位、有些特殊的样式名(marginHorizontal水平外边距、marginVertical垂直外边距)

可以通过在style属性中调用StyleSheet声明样式(个人觉得有点像scss)

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


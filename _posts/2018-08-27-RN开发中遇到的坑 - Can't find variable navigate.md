---
layout: post  
title:  "RN开发中遇到的坑 - Can't find variable: navigate"  
date: 2018-08-27  
description: "测试RN开发的组件，在使用navigate做跳转的时候出现了红屏提示。 "
tag: ReactNative
---

# RN开发中遇到的坑 - Can't find variable: navigate


测试RN开发的组件，在使用navigate做跳转的时候出现了红屏提示。
![error](https://img-blog.csdn.net/20180827170121324?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyODk1Mzc5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```javascript
class loginView extends Component {
  static navigationOptions = {
    tabBarVisible: false, // 隐藏底部导航栏
    header:null,  //隐藏顶部导航栏
  };

  constructor(props){
      super(props);
      this.state = {title : '首界面'};
      this.btn_press = this.btn_press.bind(this);
    }

    render() {
    //注意这里已经添加但是仍然会红屏
    const {navigate} = this.props.navigation;
      return (
        <ImageBackground style={{height:height}}
            source={require('../img/login_background.jpg')} >
            <View style={styles.mainView}>
                  <Image style={styles.iv_style} source={require('../img/user.jpg')} />
                  <TextInput
                        style={styles.et_id_style}
                        placeholder={'请输入电子邮箱或手机号'}
                        underlineColorAndroid='transparent'
                        disableFullscreenUI={true}
                        />
                  <TextInput
                        style={styles.et_pwd_style}
                        placeholder={'请输入密码'}
                        password={true}
                        underlineColorAndroid='transparent'
                        secureTextEntry={true}
                        disableFullscreenUI={true}
                        />
                  <TouchableOpacity
                        activeOpacity={0.5}
                        onPress={() => this.btn_press()}
                        >
                        <View style={styles.btn_login} >
                            <Text style={styles.tv_login}>登陆</Text>
                        </View>
                  </TouchableOpacity>
                  <Text style={styles.tv_login}> {this.state.title}</Text>
            </View>
       </ImageBackground>
    );
  }

  btn_press(){
  //点击事件
    console.log("onPressIn");
    ToastAndroid.show("A pikachu appeared nearby !", ToastAndroid.SHORT);
    navigate('Profile');
  }
}

loginView.defaultProps={
  appName : '测试应用'
};

const App = createStackNavigator({
  Home: { screen: loginView },
  Profile: { screen: Three },
});
```
在stackoverflow看了下别人的解决方案如下：
例如在代码 
```javascript
navigate('Profile'); 
```
跳转前添加下面代码
```javascript
const {navigate} = this.props.navigation;
```

之后可能会有warning提示：
Warning: isMounted(...) is deprecated

添加以下代码
```javascript
import {
  YellowBox,
} from 'react-native';

YellowBox.ignoreWarnings(['Warning: isMounted(...) is deprecated', 'Module RCTImageLoader']);
```
可消去提示

-------------------
参考链接：https://stackoverflow.com/questions/43717456/cant-find-variable-navigate



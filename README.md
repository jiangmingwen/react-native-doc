# react-native-doc
关于 React-native个人踩坑笔记

### 1. Q: react-native-vector-icons 所有配置完毕后不生效

​       A：复制fonts字体文件到 android/app/src/main/assets/fonts/目录下



### 2.Q：运行 react-native run-android 命令报错： Could not determine the dependencies of task ':app:preDebugBuild'.

###  Could not find com.github.yalantis:ucrop:2.2.2-native.

A:  android/build.gradle 

```  maven {
allprojects {
    repositories {
        mavenLocal()
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            url("$rootDir/../node_modules/react-native/android")
        }
        maven {
            // Android JSC is installed from npm
            url("$rootDir/../node_modules/jsc-android/dist")
        }

        maven { 
            url "https://jitpack.io" 
        }
        maven {
            url 'https://maven.google.com/'
            name 'Google'
        }

        maven { url 'https://maven.aliyun.com/repository/google'}
        maven { url 'https://maven.aliyun.com/repository/jcenter'}
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public'}

        google()
        jcenter()
    }
}
```

### 3. Q：运行react-native run-android 命令报错：Task :react-native-webview:mergeDebugShaders FAILED

​       A： 重新运行 或者  cd android  & gradlew.bat clean



### 4. Q： Image 和 ImageBackground  设置图片圆角 style样式无效

​       A：  imageStyle = {borderRadius: 5}



### 5. Q： Animate  中    transform 运动卡顿

​     A： 使用 useNativeDriver 解决，如下

```
Animated.timing(this.state.translateY, {

                toValue: Constant.ScreenHeight,

                duration: 300,

                easing: Easing.linear,

                useNativeDriver: true,    //here is the fuck

})
```

### 7. Q： [Could not resolve project :react-native-camera. on Android

### ](https://github.com/react-native-community/react-native-camera/issues/2150#)

​    A： android/app/build.gradle

```
android {
    defaultConfig {
        applicationId "com.react_wms"
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode 1
        versionName "0.0.1"
        missingDimensionStrategy 'react-native-camera', 'general' // here update
        multiDexEnabled true
    }
}
```

### 8. Q: Cannot fit requested classes in a single dex file (# methods: 73133 > 65536)

```
I fixed my problem with the solution below:

## In Gradle build file, add dependency:

> implementation 'com.android.support:multidex:1.0.3'

## And then in the "defaultConfig" section, add:

> multiDexEnabled true
```

### 9. Q: Task : xxxxxxxxxxxxxxxxxxxxxxxx :compileDebugJavaWithJavac FAILED

```
react-native link
```

### 10.Q: task':app:transformNativeLibsWithStripDebugSymbolForDebug'

 Could not read path 'F:\topweb\wms_app\development\eng\code\trunk\client\react_wms\android\app\build\intermediates\transforms\stripDebugSymbol\debug\0\lib'.

```
android && gradlew clean 
cd .. && react-native run-android
```

### 11.TextInput : Invalid prop `value` of type `function` supplied to `TextInput`, expected `string`.

```

<TextInput
  keyboardType = "numeric"
  value={`${num}`} //这里改进一下就OK
/>
```

### 12.ReactNative Android9.0以上打包apk后http请求不到解决方法

在android/app/AdnroidMainifest.xml 文件中修改

```
<application  android:usesCleartextTraffic="true" ...
```

### 13.release 签名

1) 生成签名文件

`keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-alias`

2）将生成的my-release-key.jks文件拷贝到 android/app/ 目录下

3）编辑android/gradle.properties 文件，添加如下代码：

```
   android.useAndroidX=true

   android.enableJetifier=true

   MYAPP_RELEASE_STORE_FILE=my-release-key.jks

   MYAPP_RELEASE_KEY_ALIAS=my-alias

   MYAPP_RELEASE_STORE_PASSWORD=PASSWORD

   MYAPP_RELEASE_KEY_PASSWORD=PASSWORD


```

4）编辑android/app/build.gradle文件，修改 android.signingConfigs

```
signingConfigs {

        debug {

            storeFile file('debug.keystore')

            storePassword 'android'

            keyAlias 'androiddebugkey'

            keyPassword 'android'

        }

        release {

            if (project.hasProperty('MYAPP_RELEASE_STORE_FILE')) {

                storeFile file(MYAPP_RELEASE_STORE_FILE)

                storePassword MYAPP_RELEASE_STORE_PASSWORD

                keyAlias MYAPP_RELEASE_KEY_ALIAS

                keyPassword MYAPP_RELEASE_KEY_PASSWORD

            }

        }

 }
```

14. react-native-code-push

    1) . 搭建code-push-server

    ​       a. `git clone https://github.com/lisong/code-push-server.git`

    ​       b. `npm install code-push-server -g`

​              c. 初始化mysql数据库

​              `code-push-server-db init --dbhost localhost --dbuser root --dbpassword [password]`

​              d.修改配置文件code-push-server项目中的config/config.js

```
var os = require('os');

var config = {};
config.development = {
  // Config for database, only support mysql.
  db: {
    username: process.env.RDS_USERNAME || "root",
    password: process.env.RDS_PASSWORD || "jay123",
    database: process.env.DATA_BASE || "codepush",
    host: process.env.RDS_HOST || "127.0.0.1",
    port: process.env.RDS_PORT || 3306,
    dialect: "mysql",
    logging: false,
    operatorsAliases: false,
  },
  // Config for qiniu (http://www.qiniu.com/) cloud storage when storageType value is "qiniu".
  qiniu: {
    accessKey: "",
    secretKey: "",
    bucketName: "",
    downloadUrl: "" // Binary files download host address.
  },
  // Config for upyun (https://www.upyun.com/) storage when storageType value is "upyun"
  upyun: {
    storageDir: process.env.UPYUN_STORAGE_DIR,
    serviceName: process.env.UPYUN_SERVICE_NAME,
    operatorName: process.env.UPYUN_OPERATOR_NAME,
    operatorPass: process.env.UPYUN_OPERATOR_PASS,
    downloadUrl: process.env.DOWNLOAD_URL,
  },
  // Config for Amazon s3 (https://aws.amazon.com/cn/s3/) storage when storageType value is "s3".
  s3: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
    sessionToken: process.env.AWS_SESSION_TOKEN, //(optional)
    bucketName: process.env.BUCKET_NAME,
    region: process.env.REGION,
    downloadUrl: process.env.DOWNLOAD_URL, // binary files download host address.
  },
  // Config for Aliyun OSS (https://www.aliyun.com/product/oss) when storageType value is "oss".
  oss: {
    accessKeyId: "",
    secretAccessKey: "",
    endpoint: "",
    bucketName: "",
    prefix: "", // Key prefix in object key
    downloadUrl: "", // binary files download host address.
  },
  // Config for tencentyun COS (https://cloud.tencent.com/product/cos) when storageType value is "oss".
  tencentcloud: {
    accessKeyId: "",
    secretAccessKey: "",
    bucketName: "",
    region: "",
    downloadUrl: "", // binary files download host address.
  },
  // Config for local storage when storageType value is "local".
  local: {
    // Binary files storage dir, Do not use tmpdir and it's public download dir.
    storageDir: process.env.STORAGE_DIR || "I:/codepushserver/tablee/workspaces/storage",
    // Binary files download host address which Code Push Server listen to. the files storage in storageDir.
    downloadUrl: process.env.LOCAL_DOWNLOAD_URL || "http://192.168.2.134:3000/download",
    // public static download spacename.
    public: '/download'
  },
  jwt: {
    // Recommended: 63 random alpha-numeric characters
    // Generate using: https://www.grc.com/passwords.htm
    tokenSecret: process.env.TOKEN_SECRET ||'Z6wUU9wm3Kwhg4oxedfWB83cAjzm4ksvOXqog'
  },
  common: {
    /*

- tryLoginTimes is control login error times to avoid force attack.
- if value is 0, no limit for login auth, it may not safe for account. when it's a number, it means you can
- try that times today. but it need config redis server.
  /
      tryLoginTimes: 0,
      // CodePush Web(https://github.com/lisong/code-push-web) login address.
      //codePushWebUrl: "http://127.0.0.1:3001/login",
      // create patch updates's number. default value is 3
      diffNums: 3,
      // data dir for caclulate diff files. it's optimization.
      dataDir: process.env.DATA_DIR || os.tmpdir(),
      // storageType which is your binary package files store. options value is ("local" | "qiniu" | "s3"| "oss" || "tencentcloud")
      storageType: process.env.STORAGE_TYPE || "local",
      // options value is (true | false), when it's true, it will cache updateCheck results in redis.
      updateCheckCache: false,
      // options value is (true | false), when it's true, it will cache rollout results in redis
      rolloutClientUniqueIdCache: false,
    },
    // Config for smtp email，register module need validate user email project source https://github.com/nodemailer/nodemailer
    smtpConfig:{
      host: "smtp.aliyun.com",
      port: 465,
      secure: true,
      auth: {
        user: "",
        pass: ""
      }
    },
    // Config for redis (register module, tryLoginTimes module)
    redis: {
      default: {
        host: "127.0.0.1",
        port: 6379,
        retry_strategy: function (options) {
          if (options.error.code === 'ECONNREFUSED') {
            // End reconnecting on a specific error and flush all commands with a individual error
            return new Error('The server refused the connection');
          }
          if (options.total_retry_time > 1000 * 60 * 60) {
              // End reconnecting after a specific timeout and flush all commands with a individual error
              return new Error('Retry time exhausted');
          }
          if (options.times_connected > 10) {
              // End reconnecting with built in error
              return undefined;
          }
          // reconnect after
          return Math.max(options.attempt * 100, 3000);
        }
      }
    }
  }

config.development.log4js = {
  appenders: {console: { type: 'console'}},
  categories : {
    "default": { appenders: ['console'], level:'error'},
    "startup": { appenders: ['console'], level:'info'},
    "http": { appenders: ['console'], level:'info'}
  }
}

config.production = Object.assign({}, config.development);
module.exports = config; 
```

​          f: 启动 热更新服务 ​      `node ./bin/www`

​          g: 登入：`code-push login http://127.0.0.1:3000`,这时会打开浏览器，输入 admin,123456,登入，获取到token，写入 终端里

​          h: 添加应用 `code-push app add appName android react-native`

​          i: react-native项目中， `npm install --save react-native-code-push`

​          j:对于大于0.60.0的rn,需要手动link，自动link会出错， 项目根目录中创建 react-native.config.js文件，

​           写入如下代码：

```
       module.exports = {
    		dependencies: {
      			'react-native-code-push': {
       				 platforms: {
          				android: null, // disable Android platform, other platforms will still autolink
        			},
   			   },
   		 	},
 		 };

```

​           k: 修改android/setting.gradle 文件，第二行加入代码：

```
include ':react-native-code-push'
project(':react-native-code-push').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-code-push/android/app')
```

​          l: 修改android/app/build.gradle 文件

```
...
apply from: "../../node_modules/react-native/react.gradle"
apply from: "../../node_modules/react-native-code-push/android/codepush.gradle"
...
dependencies {
    implementation project(':react-native-code-push')
    ...
}   
```

​        m:  修改android/app/src/main/.../MainApplication.java文件

```
 		@Override
        protected List<ReactPackage> getPackages() {
          @SuppressWarnings("UnnecessaryLocalVariable")
          List<ReactPackage> packages = new PackageList(this).getPackages();
          packages.add(  
           new CodePush(
           getResources().getString(R.string.reactNativeCodePush_androidDeploymentKey), 
           getApplicationContext(), BuildConfig.DEBUG, 
           getResources().getString(R.string.reactNativeCodePush_androidServerURL) 
           )
          );
          return packages;
        }
        
        @Override
        protected String getJSBundleFile() {
        return CodePush.getJSBundleFile();
        }
        
```

​        n: 修改android/app/main/res/values/string.xml

```
<resources>
	<string moduleConfig="true" name="reactNativeCodePush_androidDeploymentKey">sEjXCCBT4WfVGlCwssPLQmydVChz4ksvOXqog</string>
    <string moduleConfig="true" name="reactNativeCodePush_androidServerURL">http://192.168.2.134:3000</string>
    <string name="app_name">appName</string>
</resources>

```

​            o: 发布热更新，在项目目录下，运行命令：

```
code-push release-react appName android
```


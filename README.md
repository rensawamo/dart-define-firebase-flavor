## firebaseの環境をflavorを使って分ける

<img width="267" alt="image" src="https://github.com/rensawamo/firebase-flavor/assets/106803080/7b8b257e-65b6-4818-8759-2172ca04bc9f">


###  使用するコマンド
```sh
#  実行
$  flutter run --dart-define-from-file=dart_defines/$(FLAVOR).json
or
$  flutter run --dart-define=FLAVOR=$(FLAVOR)

# アプリビルドコマンドの例
# --release　つけると release でビルドできる
flutter build ios --dart-define-from-file=dart_defines/dev.env

```

### gitignoreの設定
```sh
ios/GoogleService-Info.plist
ios/Firebase/devGoogleService-Info.plist
ios/Firebase/sthGoogleService-Info.plist
ios/Firebase/prdGoogleService-Info.plist
android/app/src/firebase/dev-google-services.json
android/app/src/firebase/stg-google-services.json
android/app/src/firebase/prd-google-services.json
```

### dart_defineの設定
ルートディレクトリに dart_definesというフォルダを作成しflavorの設定を追加
```sh
# ex
{
    "flavor": "dev",
    "appIdSuffix": "com.example.YOURAPPNAME.FLAVOR",
  }
```

### launch.jsonの設定
 下記の設定により ビルドがしやすくなる
```sh
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "dev",
            "request": "launch",
            "type": "dart",
            "flutterMode": "debug",
            "args": [
                "--dart-define-from-file=dart_defines/dev.json"
            ]
        },
        {
            "name": "stg",
            "request": "launch",
            "type": "dart",
            "flutterMode": "debug",
            "args": [
                "--dart-define-from-file=dart_defines/stg.json"
            ]
        },
        {
            "name": "prd",
            "request": "launch",
            "type": "dart",
            "flutterMode": "release",
            "args": [
                "--dart-define-from-file=dart_defines/prd.json"
            ]
        }
    ]
}
```
# アイコンの生成
### assets/launcher_icons/にmyiconを準備
- dev.png
- stg.png
- prd.png

### ルートに以下のファイルと設定を追加
- flutter_launcher_icons-dev.yaml
- flutter_launcher_icons-stg.yaml
- flutter_launcher_icons-prd.yaml

```sh
flutter_icons:
  android: true
  ios: true
  image_path: "assets/launcher_icons/$(FLAVOR).png"

```

### pubspec.yamlにflutter_laucher_icons`の依存関係を追加
```sh
dev_dependencies:
 ...
  flutter_launcher_icons: ^0.13.1
```

### pubspec.yamlの一番最後に以下を追記
```sh
flutter_icons:
  android: true
  ios: true
  assets:
    - ./assets/images/
    - ./assets/launcher_icons/dev.png
    - ./assets/launcher_icons/prd.png
    - ./assets/launcher_icons/stg.png

```

### 最後に以下のコマンドを実行する
```sh
$ flutter pub run flutter_launcher_icons:main
```


# iOS ver 

※ Runner.xcworkspaceを開く (水色の方 青の方(Runner.xcodeproj)は firebaseのimportエラーが出るので開かない)

### .xconfigの追加
Runner > Flutterにdev.xcconfigとprd.xcconfigを追加


![image](https://github.com/rensawamo/firebase-flavor/assets/106803080/9aa416ea-228b-4546-b29e-60bb74aefc5b)

```sh
# dev
FLAVOR=dev
APP_ID_PREFIX=.dev
```

```sh
# prd
FLAVOR=prd
APP_ID_PREFIX=
```

### Debug.xcconfigとRelease.xcconfigに以下を追加
後ほど、シェルスクリプトで生成されるファイルの　依存を示す
```sh
#include "DartDefine.xcconfig"
```

### ios/scripts/dart_define.sh　を作成し以下に編集
flavor　に応じて 上記で作成した.xcconfigの使用を切り替えるようにする
```sh
#/bin/bash
echo $DART_DEFINE | tr ',' '\n' | while read line; 
do
  echo $line | base64 -d | tr ',' '\n' | xargs -I@ bash -c "echo @ | grep 'FLAVOR' | sed 's/.*=//'"
  echo $flavor
done | (
  read flavor
  echo "#include \"$flavor.xcconfig\"" > $SRCROOT/Flutter/DartDefine.xcconfig
)
```

### ファイルの権限の設定
```sh
$ chmod 755 ios/scripts/dart_define.sh 
```

### Pre-actionを設定する
xcodeの以下より　
![image](https://github.com/rensawamo/firebase-flavor/assets/106803080/32efa11e-3126-4cba-961e-6e8d52972589)

Edit Scheme > Build > Pre Acion > New Run Script Actionに以下を記載
```sh
sh "$SRCROOT/scripts/dart_define.sh"
```

![image](https://github.com/rensawamo/firebase-flavor/assets/106803080/5ad343cf-59fa-4054-af42-8e0ee21e8615)


### アイコンの切り替え
Runner > Build Settingsを開き、Primary App Icon Set Nameを以下に書き換える
```sh
AppIcon-$(FLAVOR)
```

### Bundle idの切り替え
TARGETS > Runner > Build Settingsを開き、Product Bundle Identifierを以下に書き換える
```sh
$com.example.YOURAPPNAME$(APP_ID_PREFIX)
```

### info　ファイルのbudle設定
TARGETS > Runner > Infoを開き、Bundle display nameを以下に書き換える

```sh
YOURAPPNAME$(APP_ID_PREFIX)
```
![image](https://github.com/rensawamo/firebase-flavor/assets/106803080/a7402c1e-7da7-4648-92a4-5d1419b5f97d)


### Xcode　より  以下のようにFirebaseフォルダにそれぞれの GoogleService-Infoを追加
※ 以下のように設定  別のcodeエディターからファイルが発見できないため

![image](https://github.com/rensawamo/firebase-flavor/assets/106803080/942b52a6-f57b-41ad-8149-3a29b7392d2e)

      　　

すべての　環境のものを格納

　　　

![image](https://github.com/rensawamo/firebase-flavor/assets/106803080/7785e6b1-e00f-4100-b064-581546a8897c)


### 使用する  GoogleService-Infoを ルートの GoogleService-Infoにコピーする記述
TARGETS/Runner/Build Phasesを開き、右上の+ボタンからNew Run Script Phaseを選択
```sh
cp -f ${SRCROOT}/Firebase/${FLAVOR}GoogleService-Info.plist ${SRCROOT}/GoogleService-Info.plist
```

上記のコマンドを実行できるようにする
![image](https://github.com/rensawamo/firebase-flavor/assets/106803080/f048371b-6b67-41a5-8f7c-c4925ca93b47)

### GoogleService-Info.plistの参照を作成
ルートにGoogleService-InfoをfirebaseからDWして 毎回書き換えるためのファイルを置いておく。(なんの環境でもいい)



![image](https://github.com/rensawamo/dart-define-firebase-flavor/assets/106803080/2958324e-652a-4efe-84a1-02bab2015382)





# Android
### android/app に firebaseフォルダを作成して各 flavorの google-services.jsonを 以下のように名前を変えていれる
必要なflvorだけ用意


![image](https://github.com/rensawamo/dart-define-firebase-flavor/assets/106803080/de030a68-c2d7-46a6-816a-965536ae9dbf)




### android/app/build.gradle の設定
```sh
# plugins {...}の下
# 環境変数の定義
def dartEnvironmentVariables = project
            .property('dart-defines')
            .split(',')
            .collectEntries {
                new String(it.decodeBase64(), 'UTF-8')
                    .split(',')
                    .collectEntries {
                        def pair = it.split('=')
                        [(pair.first()): pair.last()]
                    }
            }
# icon imagaのコピー
task copyIcons(type: Copy) {
    from "src/${dartEnvironmentVariables.FLAVOR}/res"
    into 'src/main/res'
}
# firebaseファイルから flavorのgoole-services.jsonを選択
task copyFirebaseSource(type: Copy) {
    from "src/firebase/${dartEnvironmentVariables.FLAVOR}-google-services.json"
    into './'
    rename { String fileName ->
        fileName = "google-services.json"
    }
}

tasks.whenTaskAdded {
    it.dependsOn copyFirebaseSource
    it.dependsOn copyIcons
}

```


### suffixの設定
```sh
defaultConfig {
        applicationId "com.example.YOURAPPNAME"
        minSdkVersion 26
        targetSdkVersion 30
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName

        if(dartEnvironmentVariables.FLAVOR == 'dev') {
            applicationIdSuffix ".dev"
            manifestPlaceholders += [appNamePrefix:".dev"]
        } else if(dartEnvironmentVariables.FLAVOR == 'stg') {
            applicationIdSuffix ".stg"
            manifestPlaceholders += [appNamePrefix:".stg"]
        } else if(dartEnvironmentVariables.FLAVOR == 'prd') {
            applicationIdSuffix ""
            manifestPlaceholders += [appNamePrefix:""]
        } else {
          print("error:Flavor not found")
        }
    }
```

### appの名前を変える
android/app/src/main/AndroidManifest.xmlの以下に設定を変える
```sh
<application
        android:label="YOURAPPNAME${appNamePrefix}"
```


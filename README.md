## firebaseの環境をflavorを使って分ける

### Production(本番) flavorの appidを取得
/android/app/build.gradleから appidを取得する
```sh
android {
    namespace "com.example.googletry"   ←　(defalutでは com.exampleになっている)
```


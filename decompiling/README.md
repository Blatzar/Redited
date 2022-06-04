## Starting modifying reddit

1. Get [apktool](https://github.com/iBotPeaches/Apktool/releases)
2. Get [uber-apk-signer](https://github.com/patrickfav/uber-apk-signer)
3. Create a signing certificate, this can be done inside [android studio](https://developer.android.com/studio/publish/app-signing#generate-key) or
using [keytool](https://stackoverflow.com/questions/11308077/generate-key-and-certificate-using-keytool)
4. Get your reddit apk of choice from the internet or likewise
5. Decompile the app using the following command, substitute the apktool version for what you downloaded.
```fish
java -jar apktool_2.6.1.jar d reddit.apk -r
```
6. Open your text editor of choice in the new directory `reddit` and edit away, I recommend using [jadx-gui](https://github.com/skylot/jadx/releases) for finding stuff to edit.
7. Compile the app again using the following command
```fish
java -jar apktool_2.6.1.jar b reddit/ --use-aapt2
```
8. Sign your apk with uber-apk-signer like this, you can use other signing software but i prefer this. Substitute keystore path, alias and passwords appropriately. 
```fish
uber-apk-signer -a reddit/dist/reddit.apk --ks /path/to/your/keystore.jks --ksAlias "keyalias" --ksKeyPass "password" --ksPass "password"
```
9. Install the apk!

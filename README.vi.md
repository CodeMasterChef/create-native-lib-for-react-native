
![enter image description here](https://i.imgur.com/FCgIKGa.jpg)

# Bước 1: Tạo thư viện
Sử dụng generator để tạo thư viện: [react-native-create-library](https://github.com/frostney/react-native-create-library)
Cài đặt generator với lệnh: 

    npm install -g react-native-create-library

Tạo cấu trúc thư mục, giả sử cần tạo library có tên là **MyFancyLibrary**:

    react-native-create-library MyFancyLibrary

Hai tập tin quan trọng nhất đó là RNMyFancyLibraryModule.java (chứa toàn bộ xử lí của native) và RNMyFancyLibraryPackage.java. Vấn đề duy nhất là đối với React Native phiên bản 0.47.0 trở lên, ta cần xóa đoạn code trong tập tin RNMyFancyLibraryPackage.java: 

   

```java
	 // Deprecated from RN 0.47
    public List<Class<? extends JavaScriptModule>> createJSModules() {
      return Collections.emptyList();
     }
```
Để thêm phần xử lí native, ta cần thay đổi nội dung tập tin RNMyFancyLibraryModule.java . Ví dụ: tôi cần tạo một thư viện cho React Native sử dụng `android.widget.Toast` để hiện thị toast. Tôi chỉnh sửa tập tin RNMyFancyLibraryModule.java  như sau(tham khảo toàn bộ code tại  [đây](https://github.com/gitvani/ReactNativeMyFancyLibrary/blob/master/android/src/main/java/com/reactlibrary/RNMyFancyLibraryModule.java) ): 
```java
import android.widget.Toast;

import java.util.HashMap;
import java.util.Map;

........................

  private static final String DURATION_SHORT_KEY = "SHORT";
  private static final String DURATION_LONG_KEY = "LONG";
........................

  @Override
  public Map<String, Object> getConstants() {
    final Map<String, Object> constants = new HashMap<>();
    constants.put(DURATION_SHORT_KEY, Toast.LENGTH_SHORT);
    constants.put(DURATION_LONG_KEY, Toast.LENGTH_LONG);
    return constants;
  }

  @ReactMethod
  public void show(String message, int duration) {
    Toast.makeText(getReactApplicationContext(), message, duration).show();
  }
```
**Tùy chỉnh nâng cao khác:**
**Có thể đổi tên thư viện** (tên khi import vào React Native) bằng cách chỉnh sửa đoạn code sau trong tận tin RNMyFancyLibraryModule.java :
``` java
  @Override
  public String getName() {
    return "RNMyFancyLibrary";
  }
```
**Có thể đổi tên package** khi import vào java: 
1. Thay đổi android/src/main/AndroidManifest.xml.
2. Đổi tên thư mục bắt đâu từ android/src/main/java để phù hợp với tên package.
3. Điều chỉnh com.reactlibrary; trong hai tập tin RNMyFancyLibraryModule.java và RNMyFancyLibraryPackage.java.

# Bước 2: Tải thư viện lên github
Tạo Repository và push lên bình thường. Ví dụ, tôi có tải lên tại https://github.com/gitvani/ReactNativeMyFancyLibrary

#Bước 3: Cài đặt thư viện vào React Native: 
1. **Cài đặt từ link gitgub ở bước 2.** 
Ví dụ:

    npm install --save git+https://github.com/gitvani/ReactNativeMyFancyLibrary 

2. **Liên kết thư viện:**
Sử dụng lệnh:
      react-native link

Khi đó, một số thay đổi được tạo ra trong các tập tin thư mục android: 
**android/settings.gradle**
``` java
include ':react-native-my-fancy-library'
project(':react-native-my-fancy-library').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-my-fancy-library/android')
```
**android/app/src/main/java/com/testeverything/MainApplication.java**
```java
import com.reactlibrary.RNMyFancyLibraryPackage;
......................
 @Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
          new MainReactPackage(),
            new RNMyFancyLibraryPackage() // new here
      );
    }
```
Một vấn đề duy nhất đó là generator mà ta đang dùng không thể tạo thay đổi cần thiết trong tập tin **android/app/build.gradle,** do đó ta phải tự làm một cách thủ công:
```
dependencies {
    compile fileTree(dir: "libs", include: ["*.jar"])
    compile "com.android.support:appcompat-v7:23.0.1"
    compile "com.facebook.react:react-native:+"  // From node_modules
    compile project(':react-native-my-fancy-library') //new here
}
```
#Bước 4:  chạy thử 
Chỉnh sửa trong index.android.js: 
``` javascript
import RNMyFancyLibrary from 'react-native-my-fancy-library';
.................
render() {
    RNMyFancyLibrary.show('Haha, it runs fine', RNMyFancyLibrary.LONG); // new here

    return (
      <View style={styles.container}>
        <Text style={styles.instructions}>
          Hello new world!!!
        </Text>
      </View>
    );
  }
....................
```
Build lại:

    react-native run-android



#Tham khảo: 
http://cmichel.io/how-to-create-react-native-android-library/
https://github.com/frostney/react-native-create-library

![enter image description here](https://i.imgur.com/FCgIKGa.jpg)

# create-native-lib-for-react
This is instruction for creating a **Android** native library for React Native. For Vietnamese, click [here](https://github.com/gitvani/create-native-lib-for-react/blob/master/readme.vi.md) .


# Step 1: Create a library

Use the generator to create the library: [react-native-create-library](https://github.com/frostney/react-native-create-library)

Install generator with command:

    npm install -g react-native-create-library

Create the directory structure, assuming you need to create a library named ** MyFancyLibrary **:

    react-native-create-library MyFancyLibrary

The two most important files are RNMyFancyLibraryModule.java (which contains all native processing) and RNMyFancyLibraryPackage.java. The only problem is that for React Native version 0.47.0, we need to delete the code in the file RNMyFancyLibraryPackage.java:

   

```java
	 // Deprecated from RN 0.47
    public List<Class<? extends JavaScriptModule>> createJSModules() {
      return Collections.emptyList();
     }
```
To add native logic processing, we need to change the code in file RNMyFancyLibraryModule.java. For example: I need to create a library for React Native using `android.widget.Toast` to display the toast. I edited the file RNMyFancyLibraryModule.java as follows (refer to the full code at [here](https://github.com/gitvani/ReactNativeMyFancyLibrary/blob/master/android/src/main/java/com/reactlibrary/RNMyFancyLibraryModule.java) ): 
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
 **More advanced customization:**

**You can rename the library** (name when imported into React Native) by editing the following code in the RNMyFancyLibraryModule.java credentials:
``` java
  @Override
  public String getName() {
    return "RNMyFancyLibrary";
  }
```
**Can rename package when imported into java**:

1. Change android / src / main / AndroidManifest.xml.

2. Rename the folders starting from android/src/main/java to match your package name.

3. Adjust com.reactlibrary in two files RNMyFancyLibraryModule.java and RNMyFancyLibraryPackage.java.

# Step 2: Upload the library to github
Create a repository and push it up normally. For example, I have uploaded at https://github.com/gitvani/ReactNativeMyFancyLibrary

# Step 3: Install the library into React Native:

1. **Install from the gitgub link in step 2.**

For example:

    npm install --save git+https://github.com/gitvani/ReactNativeMyFancyLibrary 

2. **Link library:**

Use the command:
      react-native link

By this way, some changes are made in the android directory files:
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
The only problem is that the generator we are using can not make the necessary changes in the android / app / build.gradle file, **so we have to do it manually:**
```
dependencies {
    compile fileTree(dir: "libs", include: ["*.jar"])
    compile "com.android.support:appcompat-v7:23.0.1"
    compile "com.facebook.react:react-native:+"  // From node_modules
    compile project(':react-native-my-fancy-library') //new here
}
```
# Step 4: Build mobile app: 

Editing in index.android.js:
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
Build again with command:

    react-native run-android



# Reference: 

http://cmichel.io/how-to-create-react-native-android-library/

https://github.com/frostney/react-native-create-library

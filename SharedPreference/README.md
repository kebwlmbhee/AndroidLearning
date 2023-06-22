# SharedPreference

- [SharedPreference](#sharedpreference)
  - [基本操作](#基本操作)
  - [ViewModel with SharedPreference](#viewmodel-with-sharedpreference)

15 ~ 16

## 基本操作

**共用的資料，可以在外部和內部進行存取**
  1. getPreferences 方法，每個 Activity 只有一個

        ```java
        // MainActivity.java
        package com.example.sharedpreferences;

        import android.content.Context;
        import android.content.SharedPreferences;
        import android.os.Bundle;
        import android.util.Log;

        import androidx.appcompat.app.AppCompatActivity;

        public class MainActivity extends AppCompatActivity {

            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);

                SharedPreferences shp = getPreferences(Context.MODE_PRIVATE);
                SharedPreferences.Editor editor = shp.edit();
                // 寫入 NUMBER 為 100 的數字
                editor.putInt("NUMBER", 100);
                // 非同步提交，避免不同部件提早獲取時還未被寫入
                editor.apply();

                // 讀取 NUMBER，第一個是要讀取的變數名稱，第二個是如果沒有的默認返回值
                int x = shp.getInt("NUMBER", 0);
                String TAG = "MyLog";
                Log.d(TAG, "OnCreate: " + x);
            }
        }
        ```
        ```xml
        <!-- 在 Device File Explorer 裡的 data/data/package_name/shared_prefs 下會生成一個 MainActivity xml -->
        <?xml version='1.0' encoding='utf-8'standalone='yes' ?>
        <map>
            <int name="NUMBER" value="100" />
        </map>
        ```
  2. getSharedPreferences()，可以在一個 Activity 下創建多個
        ```java
        // MainActivity.java
        package com.example.sharedpreferences;

        import android.content.Context;
        import android.content.SharedPreferences;
        import android.os.Bundle;
        import android.util.Log;

        import androidx.appcompat.app.AppCompatActivity;

        public class MainActivity extends AppCompatActivity {

            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);

                // 只改這行
                SharedPreferences shp = getSharedPreferences("MY_DATA", Context.MODE_PRIVATE);
                SharedPreferences.Editor editor = shp.edit();
                // 寫入 NUMBER 為 100 的數字
                editor.putInt("NUMBER", 100);
                // 非同步提交，避免不同部件提早獲取時還未被寫入
                editor.apply();

                // 讀取 NUMBER，第一個是要讀取的變數名稱，第二個是如果沒有的默認返回值
                int x = shp.getInt("NUMBER", 0);
                String TAG = "MyLog";
                Log.d(TAG, "OnCreate: " + x);
            }
        }
        ```
        ```xml
        <!-- 在 Device File Explorer 裡的 data/data/package_name/shared_prefs 下會生成一個 MY_DATA xml -->
        <?xml version='1.0' encoding='utf-8' standalone='yes' ?>
        <map>
            <int name="NUMBER" value="100" />
        </map>
        ```
  3. 在 Activity 外部存取 SharedPreferences
        ```java
        // MainActivity.java
        package com.example.sharedpreferences;

        import android.os.Bundle;
        import android.util.Log;

        import androidx.appcompat.app.AppCompatActivity;

        public class MainActivity extends AppCompatActivity {

            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);

                // 不能傳遞 this，Activity 被殺掉時會導致 memory leak
                // 傳遞 getApplicationContext，因為它是單例
                MyData myData = new MyData(getApplicationContext());

                myData.number = 1000;
                myData.save();
                int y = myData.load();

                String TAG = "MyLog";
                Log.d(TAG, "onCreate: " + y);
            }
        }
        ```
        ```java
        // MyData.java
        package com.example.sharedpreferences;

        import android.content.Context;
        import android.content.SharedPreferences;

        public class MyData {

            public int number;
            private Context context;

            public MyData(Context context) {
                this.context = context;
            }
            public void save() {
                // in strings.xml，避免 hardCoded
                //     <string name="MY_DATA">my_data</string>
                String name = context.getResources().getString(R.string.MY_DATA);
                SharedPreferences shp = context.getSharedPreferences(name, Context.MODE_PRIVATE);
                SharedPreferences.Editor editor = shp.edit();
                // in strings.xml，避免 hardCoded
                //     <string name="MY_KEY">my_key</string>
                String key = context.getResources().getString(R.string.MY_KEY);
                editor.putInt(key, number);
                editor.apply();
            }

            public int load() {
                // in strings.xml，避免 hardCoded
                //     <string name="MY_DATA">my_data</string>
                String name = context.getResources().getString(R.string.MY_DATA);
                SharedPreferences shp = context.getSharedPreferences(name, Context.MODE_PRIVATE);
                // in strings.xml，避免 hardCoded
                //     <string name="MY_KEY">my_key</string>
                String key = context.getResources().getString(R.string.MY_KEY);
                // in int.xml(自己建的)，避免 hardCoded
                //     <integer name="defValue"> 0 </integer>
                int def_value = context.getResources().getInteger(R.integer.defValue);
                int x = shp.getInt(key, def_value);
                number = x;
                return x;
            }
        }
        ```

## ViewModel with SharedPreference
請先查看上方的 [ViewModel](#viewmodel) 和 [SharedPreference](#sharedpreference)

![ViewModel_With_SharedPreference](Graph/ViewModel_With_SharedPreference.png)
```java
// MyViewModel.java
package com.example.viewmodelshp;

import android.app.Application;
import android.content.Context;
import android.content.SharedPreferences;
import androidx.annotation.NonNull;
import androidx.lifecycle.AndroidViewModel;
import androidx.lifecycle.LiveData;
import androidx.lifecycle.SavedStateHandle;
public class MyViewModel extends AndroidViewModel{
    SavedStateHandle handle;
    String key = getApplication().getResources().getString(R.string.key);
    String shp_name = getApplication().getResources().getString(R.string.shp_name);
    public MyViewModel(@NonNull Application application, SavedStateHandle handle) {
        super(application);
        this.handle = handle;
        if (!handle.contains(key)) {
            load();
        }
    }
    public LiveData<Integer> getNumber() {
        return handle.getLiveData(key);
    }
    void load() {
        SharedPreferences shp = getApplication().getSharedPreferences(shp_name, Context.MODE_PRIVATE);
        int x = shp.getInt(key, 0);
        handle.set(key, x);
    }
    void save() {
        SharedPreferences shp = getApplication().getSharedPreferences(shp_name, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = shp.edit();
        editor.putInt(key, getNumber().getValue() == null ? 0 : getNumber().getValue());
        editor.apply();
    }
    public void add(int x) {
        handle.set(key, getNumber().getValue() == null ? 0 : getNumber().getValue() + x);
    }
}
```
```java
// MainActivity.java
package com.example.viewmodelshp;

import androidx.appcompat.app.AppCompatActivity;
import androidx.databinding.DataBindingUtil;
import androidx.databinding.ViewDataBinding;
import androidx.lifecycle.avedStateViewModelFactory;
import androidx.lifecycle.ViewModelProvider;
import android.os.Bundle;
import com.example.viewmodelshp.databinding.ctivityMainBinding;
public class MainActivity extends ppCompatActivity {
    MyViewModel myViewModel;
    ActivityMainBinding binding;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
        myViewModel = new ViewModelProvider(this).get(MyViewModel.class);
        binding.setData(myViewModel);
        binding.setLifecycleOwner(this);
    }
    // onStop 不太可靠，如果是後台自動殺死，就不會呼叫到，同理 onDestroy 在 onStop 沒呼叫前也不會呼叫到
    // 如果程式閃退，死機、沒電、系統意外關機就不會被保存
    // 也可以放在 MyViewModel.java 的 add() 裡，但就是比較耗時，會一直觸發
    @Override
    protected void onPause() {
        super.onPause();
        myViewModel.save();
    }
}
```
```xml
<!-- activity_main.xml -->
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

        <variable
            name="data"
            type="com.example.viewmodelshp.MyViewModel" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{String.valueOf(data.getNumber())}"
            android:textSize="36sp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintVertical_bias="0.335" />

        <Button
            android:id="@+id/button"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="@{()->data.add(1)}"
            android:text="@string/button_plus"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintHorizontal_bias="0.284"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintVertical_bias="0.499" />

        <Button
            android:id="@+id/button2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="@{()->data.add(-1)}"
            android:text="@string/button_minus"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintHorizontal_bias="0.712"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintVertical_bias="0.499" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```
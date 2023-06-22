# MVVM

- [MVVM](#mvvm)
  - [基本操作](#基本操作)
  - [ViewModel](#viewmodel)
  - [ViewModelWithLiveData](#viewmodelwithlivedata)
  - [DataBinding](#databinding)
  - [ViewModel SavedState](#viewmodel-savedstate)

10 ~ 14

**DataBinding 前先添加**

```gradle
// build.gradle(Module:app)
android {
        // 添加此行
        dataBinding {
            enabled true
        }
}
```

## 基本操作

**使用了 MVVM 後，因為 Controller 不再管理資料，Activity 短暫被消滅(返回鍵、Portrait <-> Landscape、更改語言、關閉)時，再次重啟仍會保留資料**


## ViewModel

  1. 定義類繼承 ViewModel
   
        ```java
        package com.example.viewmodel;
        import androidx.lifecycle.ViewModel;
        public class MyViewModel extends ViewModel {
            public int number = 0;
        }
        ```
  2. 透過 ViewModel 來使用數據
        ```java
        package com.example.viewmodel;
        import androidx.appcompat.apAppCompatActivity;
        import androidx.lifecyclViewModelProvider;
        import android.os.Bundle;
        import android.view.View;
        import android.widget.Button;
        import android.widget.TextView;
        public class MainActivity extendAppCompatActivity {
            // 宣告 Object
            MyViewModel myViewModel;
            TextView textView;
            Button button1, button2;
            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
                
                // 關聯 MyViewModel 類
                myViewModel = new ViewModelProvider(this).get(MyViewModel.class);
                textView = findViewById(R.id.textView);
                // 取得 myViewMode 裡的 number，這裡的 number 不再為 Controller 所持有，而是單獨存放在 myViewModel 的類中
                textView.setText(String.valueOf(myViewModel.number));
                button1 = findViewById(R.id.button);
                button2 = findViewById(R.id.button2);
                button1.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        myViewModel.number++;
                        // 存取/寫入 myViewMode 裡的 number
                        textView.setText(String.valueOf(myViewModel.number));
                    }
                });
                button2.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        // 存取/寫入 myViewMode 裡的 number
                        myViewModel.number += 2;
                        textView.setText(String.valueOf(myViewModel.number));
                    }
                });
            }
        }
        ```
## ViewModelWithLiveData

  1. 定義類繼承 ViewModel，使用 LiveData 儲存資料
   
        ```java
        // ViewModelWithLiveData.java
        package com.example.livedatatest;

        import androidx.lifecycle.MutableLiveData;
        import androidx.lifecycle.ViewModel;

        public class ViewModelWithLiveData extends ViewModel {
            // 要觀察的資料類型，並使用 MutableLiveData，因其具備可變性
            private MutableLiveData<Integer> LikedNumber;

            public MutableLiveData<Integer> getLikedNumber() {
                if(LikedNumber == null) {
                    LikedNumber = new MutableLiveData<>();
                    LikedNumber.setValue(0);
                }
                return LikedNumber;
            }

            public void addLikedNumber(int n) {
                LikedNumber.setValue(LikedNumber.getValue() + n);
            }
        }
        ```
  2. 關聯與註冊，比起沒用 LiveData 的 ViewModel，不用一直寫 textView.setText(...)

        ```java
        // MainActivity.java
        package com.example.livedatatest;

        import androidx.appcompat.app.AppCompatActivity;
        import androidx.lifecycle.Observer;
        import androidx.lifecycle.ViewModelProvider;

        import android.os.Bundle;
        import android.view.View;
        import android.widget.ImageButton;
        import android.widget.TextView;

        public class MainActivity extends AppCompatActivity {

            ViewModelWithLiveData viewModelWithLiveData;
            TextView textView;
            ImageButton imageButtonLike, imageButtonDislike;

            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_main);
                textView = findViewById(R.id.textView);
                imageButtonLike = findViewById(R.id.imageButton);
                imageButtonDislike = findViewById(R.id.imageButton2);

                viewModelWithLiveData = new ViewModelProvider(this).get(ViewModelWithLiveData.class);

                // 使用 observe 來註冊何時響應
                // 當 Android 覺得該 Activity 不再活躍時，有自動回收機制避免 memory leak，所以不用手動設置
                viewModelWithLiveData.getLikedNumber().observe(this, new Observer<Integer>() {
                    @Override
                    // 變動時進行響應
                    public void onChanged(Integer integer) {
                        textView.setText(String.valueOf(integer));
                    }
                });

                imageButtonLike.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        viewModelWithLiveData.addLikedNumber(1);
                    }
                });

                imageButtonDislike.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        viewModelWithLiveData.addLikedNumber(-1);
                    }
                });
            }
        }
        ```
## DataBinding

  1. 先進到 build.gradle(Module:app) 配置 dataBinding 為 true
        ```java
        // build.gradle(Module:app)
        android {
            namespace 'com.example.XXXXXXXXXX'
            compileSdk XX

            defaultConfig {
                applicationId "com.example.XXXXXXXXXX"
                minSdk XX
                targetSdk XX
                versionCode X
                versionName "X.X"

                testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"

                // 添加此行
                dataBinding {
                    enabled true
                }
            }
        }
        ```
  2. 進到 layout 的 xml 文件中，點擊上面的燈泡，右鍵，轉換為 DataBinding style。將數據回綁到 xml file 中，就不需要在 MainActivity 裡進行綁定，減輕 Controller 的負載量
        ```java
        // activity_main.xml
        <?xml version="1.0" encoding="utf-8"?>
        <layout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:app="http://schemas.android.com/apk/res-auto"
            xmlns:tools="http://schemas.android.com/tools">

        // 加入 data
            <data>
                <variable
                    name="data"
                    type="com.example.databinding.MyViewModel" />
            </data>

            <androidx.constraintlayout.widget.ConstraintLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                tools:context=".MainActivity">

                <TextView
                    android:id="@+id/textView"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    // 反向 binding，即回綁
                    android:text="@{String.valueOf(data.number)}"
                    android:textSize="36sp"
                    app:layout_constraintBottom_toBottomOf="parent"
                    app:layout_constraintEnd_toEndOf="parent"
                    app:layout_constraintStart_toStartOf="parent"
                    app:layout_constraintTop_toTopOf="parent"
                    app:layout_constraintVertical_bias="0.325" />

                <Button
                    android:id="@+id/button"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Button"

                    // 反向 binding，即回綁
                    android:onClick="@{()->data.add()}"
                    app:layout_constraintBottom_toBottomOf="parent"
                    app:layout_constraintEnd_toEndOf="parent"
                    app:layout_constraintHorizontal_bias="0.5"
                    app:layout_constraintStart_toStartOf="parent"
                    app:layout_constraintTop_toTopOf="parent" />

            </androidx.constraintlayout.widget.ConstraintLayout>
        </layout>
        ```
  3. 定義類繼承 ViewModel
        ```java
        // MyViewModel.java
        package com.example.databinding;

        import androidx.lifecycle.MutableLiveData;
        import androidx.lifecycle.ViewModel;


        // 繼承 ViewModel
        public class MyViewModel extends ViewModel {
            private MutableLiveData<Integer> number;

            public MutableLiveData<Integer> getNumber() {
                if(number == null) {
                number = new MutableLiveData<>();
                number.setValue(0);
                }
                return number;
            }

            public void add() {
                number.setValue(number.getValue() + 1);
            }
        }

        ```
  4. 套用在 Activity java
        ```java
        // MainActivity.java
        package com.example.databinding;

            import android.os.Bundle;

            import androidx.appcompat.app.AppCompatActivity;
            import androidx.databinding.DataBindingUtil;
            import androidx.lifecycle.ViewModelProvider;

            import com.example.databinding.databinding.ActivityMainBinding;

            public class MainActivity extends AppCompatActivity {
                MyViewModel myViewModel;

                // 取決於 layout 名稱，為名稱+binding
                // (activity_main.xml 就是 ActivityMainBinding)
                ActivityMainBinding binding;

                @Override
                protected void onCreate(Bundle savedInstanceState) {
                    super.onCreate(savedInstanceState);
                    // Databinding 的綁定方式
                    binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
                    // 關聯 myViewModel class
                    myViewModel = new ViewModelProvider(this).get(MyViewModel.class);
                    // 設置 layout 的 data
                    binding.setData(myViewModel);
                    // 一定要加，註冊綁定狀態
                    binding.setLifecycleOwner(this);
                }
            }
        ```

## ViewModel SavedState

**儘管 MVVM 使用 ViewModel 維持了 Activity 重啟時的數據存儲，但當 ViewModel 也被系統殺掉時(就是整個系統被殺掉，通常發生在記憶體不夠用或過久未使用時，系統自動會銷毀該應用程式)**


**要注意的是，如果是用戶手動殺掉，則 ViewModel SavedState 不會保留**

  1. 使用 onSavedInstanceState，跟一般的方法一樣(不建議)
        ```java
        // MyViewModel.java
        package com.example.viewmodelrestore;

        import androidx.lifecycle.MutableLiveData;
        import androidx.lifecycleSavedStateHandle;
        import androidx.lifecycle.ViewModel;

        public class MyViewModel extendsViewModel {
            private MutableLiveData<Integer> number;

            public MutableLiveData<Integer> getNumber() {
                if(number == null) {
                    number = new MutableLiveData<>();
                    number.setValue(0);
                }
                return number;
            }

            public void add() {
                number.setValue(number.getValue() + 1);
            }
        }
        ```
        ```java
        // MainActivity.java
        package com.example.viewmodelrestore;

        import androidx.annotation.NonNull;
        import androidx.appcompat.app.ppCompatActivity;
        import androidx.databinding.ataBindingUtil;
        import androidx.lifecycle.iewModelProvider;

        import android.os.Bundle;

        import com.example.viewmodelrestore.atabinding.ActivityMainBinding;

        public class MainActivity extends ppCompatActivity {
            MyViewModel myViewModel;
            ActivityMainBinding binding;
            final static String KEY_NUMBER = "my_number";

            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
                myViewModel = new ViewModelProvider(this).get(MyViewModel.class);

                // 加載
                if(savedInstanceState != null) {
                    myViewModel.getNumber().setValue(savedInstanceState.getInt(KEY_NUMBER));
                }

                binding.setData(myViewModel);
                binding.setLifecycleOwner(this);
            }

            // 儲存
            @Override
            protected void onSaveInstanceState(@NonNull Bundle outState) {
                super.onSaveInstanceState(outState);
                outState.putInt(KEY_NUMBER, myViewModel.getNumber().getValue());
            }
        }
        ```
  2. ViewModel SavedState(建議)
        ```java
        // MyViewModel.java
        package com.example.viewmodelrestore;

        import androidx.lifecycle.MutableLiveData;
        import androidx.lifecycle.avedStateHandle;
        import androidx.lifecycle.ViewModel;

        public class MyViewModel extends iewModel {
            private SavedStateHandle handle;
            public MyViewModel(SavedStateHandle handle) {
                this.handle = handle;
            }
            public MutableLiveData<Integer> getNumber() {
                // 文檔所示，傳回來的 handle 不會為空
                // 這個情況只有在程式未被加載過，第一次加載時這個判斷才會成立
                if(!handle.contains(MainActivity.KEY_NUMBER)) {
                    // 初始化為 0
                    handle.set(MainActivity.KEY_NUMBER, 0);
                }
                return handle.getLiveData(MainActivity.KEY_NUMBER);
            }

            public void add() {
                getNumber().setValue(getNumber().getValue() + 1);
            }
        }
        ```
        ```java
        // MainActivity.java
        package com.example.viewmodelrestore;

        import androidx.appcompat.app.ppCompatActivity;
        import androidx.databinding.ataBindingUtil;
        import androidx.lifecycle.iewModelProvider;

        import android.os.Bundle;

        import com.example.viewmodelrestore.atabinding.ActivityMainBinding;

        public class MainActivity extends ppCompatActivity {
            MyViewModel myViewModel;
            ActivityMainBinding binding;
            public final static String KEY_NUMBER = "my_number";

            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
                myViewModel = new ViewModelProvider(this).get(MyViewModel.class);
                binding.setData(myViewModel);
                binding.setLifecycleOwner(this);
            }
        }
        ```
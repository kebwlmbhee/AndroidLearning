# Navigation

- [Navigation](#navigation)
  - [基本操作](#基本操作)
  - [不同 Activity 間的資料傳遞](#不同-activity-間的資料傳遞)
  - [自製關鍵幀動畫 (tween animation)](#自製關鍵幀動畫-tween-animation)
  - [Navigation\_With\_ViewModel](#navigation_with_viewmodel)

17 ~ 20

## 基本操作

17

**務必先加入依賴**
```gradle
// build.gradle(Module:app)
dependencies {
    constraints {
        implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.8.0") {
            because("kotlin-stdlib-jdk7 is now a part of kotlin-stdlib")
        }
        implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.8.0") {
            because("kotlin-stdlib-jdk8 is now a part of kotlin-stdlib")
        }
    }
}
```

**使用 Navigation 可以在不同 Activity 間進行交互**

- NavHost: 存放頁面，是一個容器同時也是一個控制器，用來承載 Fragment 並管理(使用 Stack)他們的導航
- Fragment: Activity 中模塊化的部分，可將頁面分為好幾塊，用來顯示頁面的其中小部分內容，依存於 Activity，Activity 先創建再建立 Fragment 附加上去
- NavController: 切換頁面的邏輯，需要定義一些切換的方法
- NavGraph: 使用圖形化的頁面來做切換頁面的邏輯，是 NavController 的圖形化實作方法

步驟：

1. 在 build.gradle(app) 中最下方加入 dependencies

    ```gradle
    dependencies {
        constraints {
            implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.8.0") {
                because("kotlin-stdlib-jdk7 is now a part of kotlin-stdlib")
            }
            implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.8.0") {
                because("kotlin-stdlib-jdk8 is now a part of kotlin-stdlib")
            }
        }
    }
    ```
2. package 下右鍵 -> New -> Fragment，會產生相應的 layout 和 java file

    ![Create_Fragment](/Graph/Create_Fragment.png)
3. res 下右鍵 -> New -> Android Resource File -> Type 選擇 Navigation，新增 my_nav_graph.xml

    ![Create_Navigation_Resouce_File](/Graph/Create_Navigation_Resouce_File.png)
4. add a destination, placeholder 是還沒有建立 Fragment 時先用一個進行佔位，建立後可以使用右鍵選擇 Start，相連後箭頭指向可用來做頁面導向

    4.5. 右側的 Entry 和 End 可以設置切換時的動畫

5. 在主檔案的 xml 中建立 NavHostFragment，選擇剛剛建立好的 Navigation xml file(Step 3)
6. 編寫 java file
```java
// HomeFragment.java 下加入以下程式碼

// 繼承 onViewCreated，在這裡執行 View 建立好之後操作
@Override
public void onViewCreated(@NonNull View view,@Nullable Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    // Step 4 裡箭頭有自己的 id，就是這裡的 R.id.xxx
    getView().findViewById(R.id.button).setOnClickListener(
            Navigation.createNavigateOnClickListener(
                    R.id.action_homeFragment_to_detailFragment)
    );
}
```
```java
// DetailFragment.java 下加入以下程式碼

// 繼承 onViewCreated，在這裡執行 View 建立好之後操作
@Override
public void onViewCreated(@NonNull View view,@Nullable Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    // Step 4 裡箭頭有自己的 id，就是這裡的 R.id.xxx
    getView().findViewById(R.id.button2).setOnClickListener(
            Navigation.createNavigateOnClickListener(
                    R.id.action_detailFragment_to_homeFragment)
    );
}
```
```java
// MainActivity.java

package com.example.navigationdemo;

import android.os.Bundle;

import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.Toolbar;
import androidx.navigation.NavController;
import androidx.navigation.Navigation;
import androidx.navigation.fragment.NavHostFragment;
import androidx.navigation.ui.NavigationUI;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 這裡的 id 要放的是 activity_main (放置 NavHostFragment 的 Activity) 裡
        // NavHostFragment 的 id
        NavHostFragment navHostFragment = (NavHostFragment) getSupportFragmentManager().
                findFragmentById(R.id.fragmentContainerView);
        NavController controller = navHostFragment.getNavController();
        // 沒有 toolbar 會錯誤，在 activity_main 建立一個簡易 toolbar
        Toolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        NavigationUI.setupActionBarWithNavController(this, controller);
    }

    // 返回時的操作
    @Override
    public boolean onSupportNavigateUp() {
        // 原本的返回值不要
    //     return super.onSupportNavigateUp();
        NavController controller = Navigation.findNavController(this, R.id.fragmentContainerView);
        // 回傳返回操作，當點擊左上 Toolbar 的返回按鈕，則會回傳這個值，這個操作會將父節點的 stack 再次 push 
        return controller.navigateUp();
    }
}
```

## 不同 Activity 間的資料傳遞

18

![Navigation_Pass_Data](/Graph/Navigation_Pass_Data.png)

1. 重覆上述 [基本 Navigation](#基本-navigation) 1 ~ 5 步
2. 在 detailFragment 中加入 Arguments
   
   ![Add_Argument](/Graph/Add_Arguments.png)
   - 2.5 也可以在連接的箭頭裡放入取代值

        ![Navigation_Action_DefVal](/Graph/Navigation_Action_DefVal.png)
3. 編寫 Java File

```java
// HomeFragment.java
package com.example.navigationdemo2;

import android.os.Bundle;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;
import androidx.navigation.NavController;
import androidx.navigation.Navigation;

import android.text.TextUtils;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

/**
 * A simple {@link Fragment} subclass.
 * Use the {@link HomeFragment#newInstance} factory method to
 * create an instance of this fragment.
 */
public class HomeFragment extends Fragment {

    // TODO: Rename parameter arguments, choose names that match
    // the fragment initialization parameters, e.g. ARG_ITEM_NUMBER
    private static final String ARG_PARAM1 = "param1";
    private static final String ARG_PARAM2 = "param2";

    // TODO: Rename and change types of parameters
    private String mParam1;
    private String mParam2;

    public HomeFragment() {
        // Required empty public constructor
    }

    /**
     * Use this factory method to create a new instance of
     * this fragment using the provided parameters.
     *
     * @param param1 Parameter 1.
     * @param param2 Parameter 2.
     * @return A new instance of fragment Home.
     */
    // TODO: Rename and change types and number of parameters
    public static HomeFragment newInstance(String param1, String param2) {
        HomeFragment fragment = new HomeFragment();
        Bundle args = new Bundle();
        args.putString(ARG_PARAM1, param1);
        args.putString(ARG_PARAM2, param2);
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getArguments() != null) {
            mParam1 = getArguments().getString(ARG_PARAM1);
            mParam2 = getArguments().getString(ARG_PARAM2);
        }
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_home, container, false);
    }

    // 在 onViewCreated 下操作
    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        Button button = getView().findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                EditText editText = getView().findViewById(R.id.editText);
                String string = editText.getText().toString();

                // 輸入為空時的錯誤加返回
                if(TextUtils.isEmpty(string)) {
                    Toast.makeText(getActivity(), "請輸入名字", Toast.LENGTH_SHORT).show();
                    return;
                }

                // 創建 bundle，放入字串
                Bundle bundle = new Bundle();
                bundle.putString("my_name", string);

                NavController controller = Navigation.findNavController(v);
                // 透過 action 來傳遞內容
                controller.navigate(R.id.action_homeFragment_to_detailFragment, bundle);
            }
        });

    }
}
```

```java
// DetailFragment.java
package com.example.navigationdemo2;

import android.os.Bundle;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

/**
 * A simple {@link Fragment} subclass.
 * Use the {@link DetailFragment#newInstance} factory method to
 * create an instance of this fragment.
 */
public class DetailFragment extends Fragment {

    // TODO: Rename parameter arguments, choose names that match
    // the fragment initialization parameters, e.g. ARG_ITEM_NUMBER
    private static final String ARG_PARAM1 = "param1";
    private static final String ARG_PARAM2 = "param2";

    // TODO: Rename and change types of parameters
    private String mParam1;
    private String mParam2;

    public DetailFragment() {
        // Required empty public constructor
    }

    /**
     * Use this factory method to create a new instance of
     * this fragment using the provided parameters.
     *
     * @param param1 Parameter 1.
     * @param param2 Parameter 2.
     * @return A new instance of fragment DetailFragment.
     */
    // TODO: Rename and change types and number of parameters
    public static DetailFragment newInstance(String param1, String param2) {
        DetailFragment fragment = new DetailFragment();
        Bundle args = new Bundle();
        args.putString(ARG_PARAM1, param1);
        args.putString(ARG_PARAM2, param2);
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getArguments() != null) {
            mParam1 = getArguments().getString(ARG_PARAM1);
            mParam2 = getArguments().getString(ARG_PARAM2);
        }
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_detail, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        // 放在 my_nav 中 detailFragment 裡的 Arguments(Step 2)
        String string = getArguments().getString("name");
        // 獲取 homeFragment bundle 裡的字串 
        String string2 = getArguments().getString("my_name");
        TextView textView = getView().findViewById(R.id.textView);
        textView.setText(string2);
    }
}
```

## 自製關鍵幀動畫 (tween animation)

1. 新增 Animation：package 下右鍵 -> New -> Android Resource File

    ![Add_New_Animation_XML](/Graph/Add_New_Animation_XML.png)
2. 編寫 Animation XML file
   ```xml
   <!-- slide_from_left.xml -->
   <?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android">
        <!-- 從左側
            到正中
            300 毫秒 -->
        <translate
            android:duration="300"
            android:fromXDelta="-100%"
            android:toXDelta="0%"></translate>
    </set>
   ```
   ```xml
   <!-- slide_to_right.xml -->
    <?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android">
        <!-- 從中間
            到右側
            300 毫秒 -->
        <translate
            android:fromXDelta="0%"
            android:toXDelta="100%"
            android:duration = "300"></translate>
    </set>
   ```
3. !! 將動畫套用到之前已經建立的 my_nav (Navigation Resource) 的 action 上 !!

    ![Action_Using_Animation](/Graph/Action_Using_Animation.png)

補充

縮放、旋轉的動畫實現，在實際應用上動畫不要過於花俏(不要進行疊加)，此處僅用於展示

```xml
<!-- scale_rotate.xml -->
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!--  x 軸從 0.0 到 1.0
          y 軸從 0.0 到 1.0
          x 軸以中心點為軸心
          y 軸以中心點為軸心
          時間 1 秒  -->
    <scale
        android:duration="1000"
        android:fromXScale="0.0"
        android:fromYScale="0.0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="1.0"
        android:toYScale="1.0"></scale>

    <!--  從 0 度
          到 360 度
          x 軸以中心點為軸心
          y 軸以中心點為軸心
          時間 1 秒  -->
    <rotate
        android:duration="1000"
        android:fromDegrees="0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toDegrees="360"></rotate>
</set>
```

## Navigation_With_ViewModel

![Navigation_With_ViewModel](/Graph/Navigation_With_ViewModel.png)

19

1. 重覆上述 [基本 Navigation](#基本-navigation) 1 ~ 5 步
2. 將 layout xml 更改為 DataBinding，綁定函式和變數
3. 編寫 File

```java
// MyViewModel.java
package com.example.navviewmodel;

import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.ViewModel;

public class MyViewModel extends ViewModel {
    private MutableLiveData<Integer> number;

    public MutableLiveData<Integer> getNumber() {
        if(number == null) {
            number = new MutableLiveData<>();
            number.setValue(0);
        }
        return number;
    }

    public void add(int x) {
        getNumber().setValue(getNumber().getValue() + x);

        if(getNumber().getValue() < 0) {
            getNumber().setValue(0);
        }
    }
}
```

```java
// MasterFragment.java
package com.example.navviewmodel;

import android.os.Bundle;

import androidx.databinding.DataBindingUtil;
import androidx.fragment.app.Fragment;
import androidx.lifecycle.ViewModelProvider;
import androidx.navigation.NavController;
import androidx.navigation.Navigation;

import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.SeekBar;

import com.example.navviewmodel.databinding.FragmentMasterBinding;

/**
 * A simple {@link Fragment} subclass.
 * Use the {@link MasterFragment#newInstance} factory method to
 * create an instance of this fragment.
 */
public class MasterFragment extends Fragment {

    // TODO: Rename parameter arguments, choose names that match
    // the fragment initialization parameters, e.g. ARG_ITEM_NUMBER
    private static final String ARG_PARAM1 = "param1";
    private static final String ARG_PARAM2 = "param2";

    // TODO: Rename and change types of parameters
    private String mParam1;
    private String mParam2;

    public MasterFragment() {
        // Required empty public constructor
    }

    /**
     * Use this factory method to create a new instance of
     * this fragment using the provided parameters.
     *
     * @param param1 Parameter 1.
     * @param param2 Parameter 2.
     * @return A new instance of fragment MasterFragment.
     */
    // TODO: Rename and change types and number of parameters
    public static MasterFragment newInstance(String param1, String param2) {
        MasterFragment fragment = new MasterFragment();
        Bundle args = new Bundle();
        args.putString(ARG_PARAM1, param1);
        args.putString(ARG_PARAM2, param2);
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getArguments() != null) {
            mParam1 = getArguments().getString(ARG_PARAM1);
            mParam2 = getArguments().getString(ARG_PARAM2);
        }
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {

        MyViewModel myViewModel;
        // getActivity 獲取關聯的 Activity 實例
        myViewModel = new ViewModelProvider(getActivity()).get(MyViewModel.class);
        FragmentMasterBinding binding;
        // Activity 是使用 setContentView, Fragment 是使用 inflate
        binding = DataBindingUtil.inflate(inflater, R.layout.fragment_master, container, false);
        binding.setData(myViewModel);
        binding.setLifecycleOwner(getActivity());

        binding.button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                NavController controller = Navigation.findNavController(v);
                controller.navigate(R.id.action_masterFragment_to_detailFragment);
            }
        });

        // 初始化，如果裡面已有值，將 progress 初始滑動至該值
        binding.seekBar.setProgress(myViewModel.getNumber().getValue());

        binding.seekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                // 將 number 的值設為 progress
                myViewModel.getNumber().setValue(progress);
            }

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {

            }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {

            }
        });

        // 返回一個 binding 的根節點，其為 View 類型
        return binding.getRoot();

        // Inflate the layout for this fragment
//        return inflater.inflate(R.layout.fragment_master, container, false);
    }
}
```

```java
// DetailFragment.java
package com.example.navviewmodel;

import android.os.Bundle;

import androidx.databinding.DataBindingUtil;
import androidx.fragment.app.Fragment;
import androidx.lifecycle.ViewModelProvider;
import androidx.navigation.NavController;
import androidx.navigation.Navigation;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import com.example.navviewmodel.databinding.FragmentDetailBinding;

/**
 * A simple {@link Fragment} subclass.
 * Use the {@link DetailFragment#newInstance} factory method to
 * create an instance of this fragment.
 */
public class DetailFragment extends Fragment {

    // TODO: Rename parameter arguments, choose names that match
    // the fragment initialization parameters, e.g. ARG_ITEM_NUMBER
    private static final String ARG_PARAM1 = "param1";
    private static final String ARG_PARAM2 = "param2";

    // TODO: Rename and change types of parameters
    private String mParam1;
    private String mParam2;

    public DetailFragment() {
        // Required empty public constructor
    }

    /**
     * Use this factory method to create a new instance of
     * this fragment using the provided parameters.
     *
     * @param param1 Parameter 1.
     * @param param2 Parameter 2.
     * @return A new instance of fragment DetailFragment.
     */
    // TODO: Rename and change types and number of parameters
    public static DetailFragment newInstance(String param1, String param2) {
        DetailFragment fragment = new DetailFragment();
        Bundle args = new Bundle();
        args.putString(ARG_PARAM1, param1);
        args.putString(ARG_PARAM2, param2);
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getArguments() != null) {
            mParam1 = getArguments().getString(ARG_PARAM1);
            mParam2 = getArguments().getString(ARG_PARAM2);
        }
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        MyViewModel myViewModel;
        // getActivity 獲取關聯的 Activity 實例
        myViewModel = new ViewModelProvider(getActivity()).get(MyViewModel.class);
        FragmentDetailBinding binding;
        // Activity 是使用 setContentView, Fragment 是使用 inflate
        binding = DataBindingUtil.inflate(inflater, R.layout.fragment_detail, container, false);
        binding.setData(myViewModel);
        binding.setLifecycleOwner(getActivity());

        binding.button4.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                NavController controller = Navigation.findNavController(v);
                controller.navigate(R.id.action_detailFragment_to_masterFragment);
            }
        });

        // 返回一個 binding 的根節點，其為 View 類型
        return binding.getRoot();

        // Inflate the layout for this fragment
//        return inflater.inflate(R.layout.fragment_detail, container, false);
    }
}
```

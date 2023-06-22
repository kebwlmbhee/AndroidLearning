# MVC
**最一般的處理方式，為人垢病的就是要一直保存資料，避免 Activity 被短暫消滅後造成資料丟失**

```java
// MainActivity.java
package com.example.orientation;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.os.PersistableBundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    String TAG = "myLog";
    Button button2;
    TextView textView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        textView = findViewById(R.id.textView);
        button2 = findViewById(R.id.button2);

        // onCreate 後馬上找尋是否有之前暫存的數據，有的話載入進來
        if(savedInstanceState != null) {
            String s = savedInstanceState.getString("KEY");
            textView.setText(s);
        }

        button2.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                textView.setText(R.string.button2);
            }
        });
    }

    // 為避免 Activity onDestroy 後數據丟失，要先保存在 SaveInstanceState
    @Override
    protected void onSaveInstanceState(@NonNull Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putString("KEY", textView.getText().toString());
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy: ");
    }
}
```
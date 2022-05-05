## Activity Result API

修改《第一行代码》中已经弃用的startActivityForResult()，改为Activity Result API

参考：

https://blog.csdn.net/guolin_blog/article/details/121063078 kotlin

https://programmer.ink/think/android-gets-results-from-other-activities-registerforactivityresult.html java



```JAVA

public class SecondActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.second_layout);
        Button button2 = (Button) findViewById(R.id.button_2);
        button2.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent();
                intent.putExtra("data_return", "Hello FirstActivity");
                setResult(RESULT_OK, intent);
                finish();
            }
        });


    }
}
```



先调用registerForActivityResult()方法来注册一个对Activity结果的监听。

registerForActivityResult() accepts ActivityResultContract and ActivityResultCallback as parameters and returns ActivityResultLauncher to start another activity, where:

- ActivityResultContract defines the input type required to generate the result and the output type of the result. These API s can provide default contracts for basic intent operations such as taking photos and requesting permission, and can also create custom contracts.
- ActivityResultCallback is a single method interface with onActivityResult() method, which can accept objects of output type defined in ActivityResultContract:



registerForActivityResult()方法有ActivityResultContract and ActivityResultCallback两个参数，并且会返回ActivityResultLauncher进行启动别的活动

官方的方法为

```
ActivityResultLauncher<String> mGetContent = registerForActivityResult(new GetContent(),
    new ActivityResultCallback<Uri>() {
        @Override
        public void onActivityResult(Uri uri) {
            // Handle the returned Uri
        }
});
```

在此情况下写为

```java
ActivityResultLauncher launcher = registerForActivityResult( new ActivityResultContracts.StartActivityForResult(),
            new ActivityResultCallback<ActivityResult>() {
                @Override
                public void onActivityResult(ActivityResult result) {
                    if (result.getResultCode() == Activity.RESULT_OK) {
                        // There are no request codes
                        Intent data = result.getData();
                        String returnedData = data.getStringExtra("data_return");
                        Log.d("FirstActivity", returnedData);

                    }
                }
            });

```

### Start Activity to get results

registerForActivityResult() only registers a result callback for the result obtained by the activity, but it does not start another activity and issue a result request itself. The operation of starting an activity is the responsibility of the instance object of the ActivityResultLauncher returned by registerForActivityResult().

If there are input parameters, the instance object of ActivityResultLauncher will match the type of ActivityResultContract according to the input parameters. Calling the launch() method of the instance object of ActivityResultLauncher will start the activity and get the results. When the user completes the subsequent activities and returns, the system will execute the onActivityResult() method in the ActivityResultCallback.

需要创建一个ActivityResultLauncher 去进行跳转活动的操作，用launch()这个方法可以跳转活动并且得到结果，当完成了这个活动后，系统会在ActivityResultCallback中执行onActivityResult()

官方示例

```java

ActivityResultLauncher<String> mGetContent = registerForActivityResult(new GetContent(),
    new ActivityResultCallback<Uri>() {
        @Override
        public void onActivityResult(Uri uri) {
            // Handle the returned Uri
        }
});

@Override
public void onCreate(@Nullable savedInstanceState: Bundle) {
    // ...

    Button selectButton = findViewById(R.id.select_button);

    selectButton.setOnClickListener(new OnClickListener() {
        @Override
        public void onClick(View view) {
            // Pass in the mime type you'd like to allow the user to select
            // as the input
            mGetContent.launch("image/*");
        }
    });
}
```



firstactivity的按钮函数应该为

```
button1.setOnClickListener(new View.OnClickListener() {

            public void onClick(View v) {
                Intent intent = new Intent(FirstActivity.this, SecondActivity.class);
                launcher.launch(intent);
            }
        });
```


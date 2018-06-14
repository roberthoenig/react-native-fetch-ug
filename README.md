This repository features a minimal code example to reproduce a bug in React Native's Fetch API.

## Code

```.java
package com.example.robert.demoapp;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;

import java.io.IOException;
import java.util.concurrent.TimeUnit;

import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.Headers;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;
import okhttp3.ResponseBody;

public class MainActivity extends AppCompatActivity {

    public final OkHttpClient.Builder clientBuilder = new OkHttpClient.Builder();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        clientBuilder.readTimeout(0, TimeUnit.SECONDS);
    }

    public void fetch(View view) {
        Request request = new Request.Builder()
                .url("http://publicobject.com/helloworld.txt")
                .build();
        OkHttpClient client = clientBuilder.build();
        client.newCall(request).enqueue(new Callback() {
            @Override public void onFailure(Call call, IOException e) {
                e.printStackTrace();
            }

            @Override public void onResponse(Call call, Response response) throws IOException {
                    ResponseBody responseBody = response.body();
                    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

                    Headers responseHeaders = response.headers();
                    for (int i = 0, size = responseHeaders.size(); i < size; i++) {
                        Log.d("response", responseHeaders.name(i) + ": " + responseHeaders.value(i));
                    }
            }
        });
    }

    public void evict(View view) {
        OkHttpClient client = clientBuilder.build();
        client.connectionPool().evictAll();
    }
}
```

`fetch()` and `evict()` are connected to a button each.

## Reproduction

1. Quickly press FETCH repeatedly. The requested resources come in, headers are logged.
2. Quickly turn off data.
3. Turn on data.
4. Press FETCH again. Wait for some time. **No response will come in.**
5. Press EVICT CONNECTION POOL.
6. Press FETCH again. The requested resource comes in, headers are logged.

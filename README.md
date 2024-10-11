<uses-permission android:name="android.permission.INTERNET" />
dependencies {
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
    implementation 'com.google.android.material:material:1.4.0'
    implementation 'androidx.recyclerview:recyclerview:1.2.1'
    implementation 'com.squareup.picasso:picasso:2.71828'  // لتسهيل تحميل الصور إذا لزم الأمر
}
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <Spinner
        android:id="@+id/reciterSpinner"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="16dp"/>

    <RecyclerView
        android:id="@+id/surahRecyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
package com.example.quranapp;

import android.media.MediaPlayer;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.Spinner;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;
import com.example.quranapp.model.Surah;
import com.example.quranapp.network.ApiClient;
import com.example.quranapp.network.ApiService;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;

public class MainActivity extends AppCompatActivity {

    private RecyclerView surahRecyclerView;
    private SurahAdapter surahAdapter;
    private Spinner reciterSpinner;
    private MediaPlayer mediaPlayer;
    private List<String> reciters;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        surahRecyclerView = findViewById(R.id.surahRecyclerView);
        reciterSpinner = findViewById(R.id.reciterSpinner);
        mediaPlayer = new MediaPlayer();

        // إعداد RecyclerView
        surahRecyclerView.setLayoutManager(new LinearLayoutManager(this));
        surahAdapter = new SurahAdapter(new ArrayList<>(), this::playAudio);
        surahRecyclerView.setAdapter(surahAdapter);

        // جلب السور
        fetchSurahs();

        // إعداد المقرئين
        reciters = new ArrayList<>();
        reciters.add("Abdul Basit");
        reciters.add("Al-Afasy");
        reciters.add("Al-Shuraim");

        ArrayAdapter<String> adapter = new ArrayAdapter<>(this, android.R.layout.simple_spinner_item, reciters);
        adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        reciterSpinner.setAdapter(adapter);

        reciterSpinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                // تغيير المقرئ عند الاختيار
            }

            @Override
            public void onNothingSelected(AdapterView<?> parent) {}
        });
    }

    private void fetchSurahs() {
        ApiService apiService = ApiClient.getRetrofitInstance().create(ApiService.class);
        Call<List<Surah>> call = apiService.getSurahs();
        call.enqueue(new Callback<List<Surah>>() {
            @Override
            public void onResponse(Call<List<Surah>> call, Response<List<Surah>> response) {
                if (response.isSuccessful() && response.body() != null) {
                    surahAdapter.updateSurahs(response.body());
                }
            }

            @Override
            public void onFailure(Call<List<Surah>> call, Throwable t) {
                Log.e("MainActivity", "Error fetching surahs", t);
            }
        });
    }

    private void playAudio(String url) {
        try {
            if (mediaPlayer.isPlaying()) {
                mediaPlayer.stop();
                mediaPlayer.reset();
            }
            mediaPlayer.setDataSource(url);
            mediaPlayer.prepare();
            mediaPlayer.start();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mediaPlayer != null) {
            mediaPlayer.release();
            mediaPlayer = null;
        }
    }
}
 -
تطبيق للقران الكريم بصوت جميل

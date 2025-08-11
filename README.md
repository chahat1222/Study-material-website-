# Study-material-website-
Study material 
package com.chahat.studyapp;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
    Button adminLogin;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        adminLogin = findViewById(R.id.adminLogin);

        adminLogin.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startActivity(new Intent(MainActivity.this, AdminLoginActivity.class));
            }
        });
    }
}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:background="#FFFFFF">

    <Button
        android:id="@+id/adminLogin"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Admin Login"
        android:padding="16dp"
        android:textSize="18sp"/>
</LinearLayout>
package com.chahat.studyapp;

import android.content.Intent;
import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

public class AdminLoginActivity extends AppCompatActivity {
    EditText username, password;
    Button loginBtn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_admin_login);

        username = findViewById(R.id.username);
        password = findViewById(R.id.password);
        loginBtn = findViewById(R.id.loginBtn);

        loginBtn.setOnClickListener(view -> {
            String user = username.getText().toString().trim();
            String pass = password.getText().toString().trim();

            if (user.equals("admin") && pass.equals("admin123")) {
                startActivity(new Intent(AdminLoginActivity.this, UploadActivity.class));
                finish();
            } else {
                Toast.makeText(this, "गलत Username या Password", Toast.LENGTH_SHORT).show();
            }
        });
    }
}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="24dp"
    android:orientation="vertical"
    android:gravity="center"
    android:background="#FFFFFF">

    <EditText
        android:id="@+id/username"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Username"
        android:inputType="text"/>

    <EditText
        android:id="@+id/password"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Password"
        android:inputType="textPassword"
        android:layout_marginTop="16dp"/>

    <Button
        android:id="@+id/loginBtn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Login"
        android:layout_marginTop="24dp"/>
</LinearLayout>
package com.chahat.studyapp;

import android.app.ProgressDialog;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.widget.*;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import com.google.firebase.storage.FirebaseStorage;
import com.google.firebase.storage.StorageReference;
import com.google.firebase.firestore.FirebaseFirestore;

import java.util.HashMap;
import java.util.UUID;

public class UploadActivity extends AppCompatActivity {

    Button chooseBtn, uploadBtn;
    Spinner subjectSpinner;
    TextView fileNameText;
    Uri fileUri;

    FirebaseStorage storage;
    FirebaseFirestore firestore;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_upload);

        chooseBtn = findViewById(R.id.chooseBtn);
        uploadBtn = findViewById(R.id.uploadBtn);
        subjectSpinner = findViewById(R.id.subjectSpinner);
        fileNameText = findViewById(R.id.fileNameText);

        storage = FirebaseStorage.getInstance();
        firestore = FirebaseFirestore.getInstance();

        ArrayAdapter<String> adapter = new ArrayAdapter<>(this,
                android.R.layout.simple_spinner_item,
                new String[]{"Maths", "Science", "English"});
        subjectSpinner.setAdapter(adapter);

        chooseBtn.setOnClickListener(view -> {
            Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
            intent.setType("application/pdf");
            startActivityForResult(intent, 101);
        });

        uploadBtn.setOnClickListener(view -> {
            if (fileUri != null) {
                uploadPdf();
            } else {
                Toast.makeText(this, "कृपया कोई PDF चुनें", Toast.LENGTH_SHORT).show();
            }
        });
    }

    private void uploadPdf() {
        ProgressDialog dialog = new ProgressDialog(this);
        dialog.setTitle("Uploading...");
        dialog.show();

        String subject = subjectSpinner.getSelectedItem().toString();
        String fileName = UUID.randomUUID().toString() + ".pdf";
        StorageReference ref = storage.getReference().child("pdfs/" + fileName);

        ref.putFile(fileUri).addOnSuccessListener(taskSnapshot -> {
            ref.getDownloadUrl().addOnSuccessListener(uri -> {
                HashMap<String, Object> map = new HashMap<>();
                map.put("url", uri.toString());
                map.put("fileName", fileName);

                firestore.collection(subject).add(map).addOnSuccessListener(documentReference -> {
                    dialog.dismiss();
                    Toast.makeText(this, "Upload Successful", Toast.LENGTH_SHORT).show();
                });
            });
        }).addOnFailureListener(e -> {
            dialog.dismiss();
            Toast.makeText(this, "Upload Failed", Toast.LENGTH_SHORT).show();
        });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        if (requestCode == 101 && resultCode == RESULT_OK && data != null) {
            fileUri = data.getData();
            fileNameText.setText("Selected: " + fileUri.getLastPathSegment());
        }
    }
}

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp"
    android:gravity="center"
    android:background="#FFFFFF">

    <Spinner
        android:id="@+id/subjectSpinner"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <Button
        android:id="@+id/chooseBtn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Choose PDF"
        android:layout_marginTop="24dp"/>

    <TextView
        android:id="@+id/fileNameText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="No file chosen"
        android:layout_marginTop="16dp"/>

    <Button
        android:id="@+id/uploadBtn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Upload"
        android:layout_marginTop="24dp"/>
</LinearLayout>
package com.chahat.studyapp;

import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.widget.*;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.google.firebase.firestore.*;
import java.util.*;

public class StudentActivity extends AppCompatActivity {

    Spinner subjectSpinner;
    RecyclerView recyclerView;
    ArrayList<PDFModel> list;
    PDFAdapter adapter;

    FirebaseFirestore firestore;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_student);

        subjectSpinner = findViewById(R.id.subjectSpinner2);
        recyclerView = findViewById(R.id.recyclerView);

        firestore = FirebaseFirestore.getInstance();

        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        list = new ArrayList<>();
        adapter = new PDFAdapter(list);
        recyclerView.setAdapter(adapter);

        ArrayAdapter<String> arrayAdapter = new ArrayAdapter<>(this,
                android.R.layout.simple_spinner_item,
                new String[]{"Maths", "Science", "English"});
        subjectSpinner.setAdapter(arrayAdapter);

        subjectSpinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                fetchPDFs(subjectSpinner.getSelectedItem().toString());
            }

            @Override public void onNothingSelected(AdapterView<?> parent) {}
        });
    }

    private void fetchPDFs(String subject) {
        firestore.collection(subject).get().addOnSuccessListener(queryDocumentSnapshots -> {
            list.clear();
            for (DocumentSnapshot snapshot : queryDocumentSnapshots) {
                String url = snapshot.getString("url");
                String fileName = snapshot.getString("fileName");
                list.add(new PDFModel(fileName, url));
            }
            adapter.notifyDataSetChanged();
        });
    }

    // Adapter
    class PDFAdapter extends RecyclerView.Adapter<PDFAdapter.PDFViewHolder> {
        ArrayList<PDFModel> pdfList;

        public PDFAdapter(ArrayList<PDFModel> pdfList) {
            this.pdfList = pdfList;
        }

        @NonNull
        @Override
        public PDFViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
            View view = getLayoutInflater().inflate(android.R.layout.simple_list_item_1, parent, false);
            return new PDFViewHolder(view);
        }

        @Override
        public void onBindViewHolder(@NonNull PDFViewHolder holder, int position) {
            PDFModel model = pdfList.get(position);
            holder.textView.setText(model.getFileName());
            holder.itemView.setOnClickListener(v -> {
                Intent intent = new Intent(Intent.ACTION_VIEW);
                intent.setData(Uri.parse(model.getUrl()));
                startActivity(intent);
            });
        }

        @Override
        public int getItemCount() {
            return pdfList.size();
        }

        class PDFViewHolder extends RecyclerView.ViewHolder {
            TextView textView;

            public PDFViewHolder(@NonNull View itemView) {
                super(itemView);
                textView = itemView.findViewById(android.R.id.text1);
            }
        }
    }

    class PDFModel {
        String fileName, url;

        public PDFModel(String fileName, String url) {
            this.fileName = fileName;
            this.url = url;
        }

        public String getFileName() { return fileName; }

        public String getUrl() { return url; }
    }
}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="24dp"
    android:background="#FFFFFF">

    <Spinner
        android:id="@+id/subjectSpinner2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginTop="16dp"/>
</LinearLayout>
// Firestore Rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{subject}/{document=**} {
      allow read, write: if true;
    }
  }
}
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.chahat.studyapp">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="Study App"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.StudyApp">
        
        <activity android:name=".StudentActivity" />
        <activity android:name=".UploadActivity" />
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

    </application>

</manifest>
buildscript {
    dependencies {
        classpath 'com.google.gms:google-services:4.3.15'
    }
}
plugins {
    id 'com.android.application'
    id 'com.google.gms.google-services'
}

android {
    compileSdk 33

    defaultConfig {
        applicationId "com.chahat.studyapp"
        minSdk 21
        targetSdk 33
        versionCode 1
        versionName "1.0"
    }

    buildTypes {
        release {
            minifyEnabled false
        }
    }
}

dependencies {
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.firebase:firebase-storage:20.3.0'
    implementation 'com.google.firebase:firebase-firestore:24.7.1'
    implementation 'androidx.recyclerview:recyclerview:1.3.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
}
com.chahat.studyapp
├── MainActivity.java
├── UploadActivity.java
├── StudentActivity.java
├── PDFModel.java (inner class में है अभी)
├── res/
│   └── layout/
│       ├── activity_main.xml
│       ├── activity_upload.xml
│       └── activity_student.xml
├── manifest/
│   └── AndroidManifest.xml

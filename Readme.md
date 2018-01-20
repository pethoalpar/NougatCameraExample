<h1>How to take a picture in android nougat and oreo</h1>

<h3>Add file_paths.xml to define the provider</h3>

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <external-path
        name="external_files"
        path="."/>
</paths>
```

<h3>In the manifest file we must add the camera permission and the provider</h3>

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="pethoalpar.com.nougatcameraexample">

    <uses-feature android:name="android.hardware.camera" android:required="true"/>

    <application
        ...
        <provider
            android:authorities="${applicationId}.provider"
            android:name="android.support.v4.content.FileProvider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths">
            </meta-data>
        </provider>
		...
    </application>

</manifest>
```

<h3>Build the layout</h3>

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="pethoalpar.com.nougatcameraexample.MainActivity">

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="320dp"
        android:layout_height="320dp"
        android:layout_marginEnd="8dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:srcCompat="@mipmap/ic_launcher" />

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="8dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="20dp"
        android:text="Button"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/imageView" />
</android.support.constraint.ConstraintLayout>
```

<h3>Main java with algorithm</h3>

```java
public class MainActivity extends AppCompatActivity {

    private ImageView imageView;
    private Button button;

    private String mCurrentPhotoPath;

    private static final String TAG = MainActivity.class.getSimpleName();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        imageView = (ImageView) this.findViewById(R.id.imageView);
        button = (Button) this.findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                dispatchTakePictureIntent();
            }
        });
    }

    private void dispatchTakePictureIntent(){
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        if(takePictureIntent.resolveActivity(getPackageManager()) != null){
            File photoFile = null;
            try{
                photoFile = createImageFile();
            }catch (IOException ex){

            }
            if(photoFile != null){
                Uri photoURI = FileProvider.getUriForFile(this, BuildConfig.APPLICATION_ID+".provider", photoFile);
                takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoURI);
                startActivityForResult(takePictureIntent, 1024);
            }
        }
    }

    private File createImageFile() throws IOException{
        String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
        String imageFileName = "JPEG_"+timeStamp+"_";
        File storageDir = getExternalFilesDir(Environment.DIRECTORY_PICTURES);
        File image = File.createTempFile(imageFileName,".jpg", storageDir);
        mCurrentPhotoPath = image.getAbsolutePath();
        return image;
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if(requestCode == 1024){
            if(resultCode == Activity.RESULT_OK){
                getBitmap();
            }else if(resultCode == Activity.RESULT_CANCELED){
                Toast.makeText(getApplicationContext(), "Result canceled.", Toast.LENGTH_SHORT).show();
            }else{
                Toast.makeText(getApplicationContext(), "Activity result failed.", Toast.LENGTH_SHORT).show();
            }
        }
    }

    private void getBitmap(){
        try{
            BitmapFactory.Options options = new BitmapFactory.Options();
            options.inJustDecodeBounds = false;
            options.inSampleSize = 6;
            Bitmap bitmap = BitmapFactory.decodeFile(mCurrentPhotoPath, options);
            imageView.setImageBitmap(bitmap);
        } catch (Exception e){
            Log.e(TAG, e.getMessage());
        }
    }
}
```

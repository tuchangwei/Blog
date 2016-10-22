title: '[Android]导出app中的数据库'
date: 2016-08-15 00:21:14
tags: Database
categories: Android
---

最近做iOS项目，需要把Android端的数据库导出来比较一下，这个还挺麻烦。

如果用模拟器的话，可以直接用DDMS查看数据库的路径，然后导出，这样最方便。但我这个项目需要用到Google的服务， 模拟器上无法运行。

如果是真机，在不越狱的情况下是无法查看数据库路径的，没办法，只能用代码把数据库拷贝到SDCard上，然后用`File Manager`这个app把数据库文件导出来。

用代码拷贝文件到SDCard上，需要在`AndroidManifest.xml`文件中加入如下权限：
```
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

如果你的真机是6.0以上的系统，还需要在程序中向用户请求权限：

```
public void requestPermission() {
    if (ContextCompat.checkSelfPermission(this,
            android.Manifest.permission.WRITE_EXTERNAL_STORAGE)
            != PackageManager.PERMISSION_GRANTED) {

        // Should we show an explanation?
        if (ActivityCompat.shouldShowRequestPermissionRationale(this,
                android.Manifest.permission.WRITE_EXTERNAL_STORAGE)) {

            // Show an expanation to the user *asynchronously* -- don't block
            // this thread waiting for the user's response! After the user
            // sees the explanation, try again to request the permission.

        } else {

            // No explanation needed, we can request the permission.

            ActivityCompat.requestPermissions(this,
                    new String[]{android.Manifest.permission.WRITE_EXTERNAL_STORAGE},
                    0);

            // MY_PERMISSIONS_REQUEST_READ_CONTACTS is an
            // app-defined int constant. The callback method gets the
            // result of the request.
        }
    } else {
        copyDatabase();
    }
}

@Override
    public void onRequestPermissionsResult(int requestCode,
                                           String permissions[], int[] grantResults) {
        switch (requestCode) {
            case 0: {
                // If request is cancelled, the result arrays are empty.
                if (grantResults.length > 0
                        && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    copyDatabase();
                    // permission was granted, yay! Do the
                    // contacts-related task you need to do.

                } else {

                    // permission denied, boo! Disable the
                    // functionality that depends on this permission.
                }
                return;
            }

            // other 'case' lines to check for other
            // permissions this app might request
        }
    }
```
最后，将数据库拷贝到SDCard。
```
public void copyDatabase() {
        String dbPath = "/data/data/<packageName>/databases/" + DatabaseName;
        File file = new File(dbPath);
        if (file.exists()) {
            File dbCopy = new File(Environment.getExternalStorageDirectory().getAbsolutePath(), "copy.db");
            try {
                BufferedInputStream in = new BufferedInputStream(new FileInputStream(dbPath));
                BufferedOutputStream out = new BufferedOutputStream(new FileOutputStream(dbCopy));
                byte[] bb = new byte[1024];
                int n;
                while ((n = in.read(bb)) != -1) {
                    out.write(bb, 0, n);
                }
                out.close();
                in.close();
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

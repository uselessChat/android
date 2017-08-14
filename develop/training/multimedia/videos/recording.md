```java
addCameraAction(MediaStore.ACTION_VIDEO_CAPTURE, new ICamera() {
    @Override
    public void onProcess(Intent intent) {
        Uri videoUri = intent.getData();
        // viewVideo.setVideoURI(videoUri);
    }

    @Override
    public void onReject(Intent data) {
        // TODO:
    }
});
```

Android new UI toolkit Jetpack compose is in [beta now](https://android-developers.googleblog.com/2021/02/announcing-jetpack-compose-beta.html), which has all the features you need to build production-ready apps. [CameraX](https://developer.android.com/training/camerax) is another Jetpack support library, which let you control the camera easier.

As compose is still under development, lots of the views are still not available the compose way. At the moment there isn’t any official Composable function for CameraX so we have to inflate the legacy android view inside compose. In this post, I will describe a common way to use CameraX in Jetpack compose. To demo the state update and image analysis, a simple adaptive mesh camera view is built and you can find the project [here](https://github.com/pengj/compose-camerax).

![](https://miro.medium.com/max/218/1*vVeMtyU1I68wvuk4zB7Qjw.gif)

## Compose PreviewView

In the CameraX library, the [Preview](https://developer.android.com/reference/androidx/camera/core/Preview) is a use case that provides a camera preview stream for the camera preview, so the user can view the photo they will be taking. The class [PreviewView](https://developer.android.com/reference/androidx/camera/view/PreviewView) handles its view-related tasks. To preview the photo in the Jetpack compose, we just need to inflate the PreviewView inside a compose view. AndroidView is a composable that can be used to add Android Views inside of a @Composable function. In the AndroidView function, it requires three parameters:

- **ViewBlock**: This expects a function to create a class that extends an Android view. Here, we need to create and provide the PreviewView class.

- **Modifier**: This can be used to apply modifiers to the layout that will host the created View. Since we want to preview the phone in full screen, we will use Modifier._fillMaxSize()_.

- **Update**: This function is used to handle all the updates of the composition tree. Since we are streaming the Preview camera frame, we don’t need to do any view update here.

You can find the full composable function of the CameraPreview below

```
@Composable
fun SimpleCameraPreview() {
    val lifecycleOwner = LocalLifecycleOwner.current
    val context = LocalContext.current
    val cameraProviderFuture = remember { ProcessCameraProvider.getInstance(context) }

    AndroidView(
        factory = { ctx ->
            val previewView = PreviewView(ctx)
            val executor = ContextCompat.getMainExecutor(ctx)
            cameraProviderFuture.addListener({
                val cameraProvider = cameraProviderFuture.get()
                val preview = Preview.Builder().build().also {
                    it.setSurfaceProvider(previewView.surfaceProvider)
                }

                val cameraSelector = CameraSelector.Builder()
                    .requireLensFacing(CameraSelector.LENS_FACING_BACK)
                    .build()

                cameraProvider.unbindAll()
                cameraProvider.bindToLifecycle(
                    lifecycleOwner,
                    cameraSelector,
                    preview
                )
            }, executor)
            previewView
        },
        modifier = Modifier.fillMaxSize(),
    )
}
```

From the code snippet above we can see the following:

1. We create an instance of [ProcessCameraProvider](https://developer.android.com/reference/androidx/camera/lifecycle/ProcessCameraProvider) from the current local context, which will be used to bind the lifecycle of cameras and we use `remember{}` to persist for the lifetime of the process.

2. We create PreviewView with context and add Preview setup inside the runnable with `cameraProviderFuture`. You need to use `ContextCompat.getMainExecutor()` as it needs to be run on the main thread.

3. In the runnable, you can set up the surface view to initialize your Preview object and select a camera with `CameraSelected`. Before binding the lifecycle of the camera to the lifecycle owner, we call `unbindAll()` to make sure nothing is bound (clearing all the use cases).

4. After this setup, we need to remember to return the PreviewView.

If you add this composable function inside the `setContent()` method, you should see a full-screen camera preview. With the help of CameraX library, you won’t have to manage the camera session state or even dispose of images, binding to the local lifecycle is sufficient.

## Analysis Image with Palette

CameraX API is not just to make the preview and take pictures more easily, you can also apply image analysis in a very simple way. You can define a custom class that implements the _[ImageAnalysis.Analyzer](https://developer.android.com/reference/androidx/camera/core/ImageAnalysis.Analyzer)_ interface, which will be called with incoming camera frames. So you can receive image data and perform custom analysis processing. Here, I am using the palette library to check the dark vibrant colour of the camera frames.

```
val imageAnalysis = ImageAnalysis.Builder()
        .setBackpressureStrategy(ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST)
        .build()
        .apply {
            setAnalyzer(executor,analyzer)
        }

    cameraProvider.bindToLifecycle(
        lifecycleOwner,
        cameraSelector,
        imageAnalysis,
        preview
    )
```

The image analyzer can access the image proxy based on your config when creating an analyzer. The CameraX library produces image data in YUV_420_888 format (image format code 35 if you call `getFormat()`). The palette object gives you access to the colours in a Bitmap image while also providing six main colour profiles from the bitmap. To create a palette, the Palette build needs a bitmap when initializing the instance, so we need a way to convert the preview image data to bitmap. Luckily, there is a conversion function already post [here](https://stackoverflow.com/questions/56772967/converting-imageproxy-to-bitmap/56812799#56812799). More implementation details you can find in the [PaletteAnalyzer](https://github.com/pengj/compose-camerax/blob/main/app/src/main/java/me/pengj/arcompose/PaletteAnalyzer.kt). One thing that needs to remember is to close the image proxy once you finish the analysis.

## Update Mesh View with state

In the mesh view, a `MeshColor` is used to define the state and the `ViewModel` will host the state. Now, we have the image analyzer to check the palette of the camera frame. To update the mesh view state, a `onColorChanged` change event is exposed by `ViewModel` and will be fired every few seconds by the image analyzer. let's wire everything together to display a dynamic mesh view using unidirectional data flow and the state and event update loop that goes like this:

![](https://miro.medium.com/max/700/1*elT1scLq9A7XrhTl6MJHFA.png)

For MeshView composable, the state will be pass from the `ViewModel` and `onColorChanged` will be called by the ImgeAnalyzer in response to the camera frame changes.

## Compose on the way

I hope this article has shed some light on how to inflate the CameraX PreviewView inside compose and using image analysis. Of course, the journey doesn’t end here! You can see several duplicated logic that could move to the CameraX library to improve the Compose Preview experience. Please leave a comment if you have any other idea to use CameraX in a compose way.

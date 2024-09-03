---
title: "And she say's Just Not Interested (JNI)..."
categories: 
    - ML
    - WASM
    - Rust
tags: 
    - Rust
    - Federated learning
    - Burn
    - Android
header: 
sidebar:
    nav : hobby
---

Well, now that we know how to compile a rust library for android, we need to write it and interface it with the app written in Kotlin.  
If you have done some android development, you would know that we use a JNI bridge to connect to native methods/objects and call them in Java/Kotlin....

That's what we're going to do now, but first, let me talk about the silly (almost made me throw my lap) mistake one does, when they don't pay attention to the things they are reading : 
Okay, so the native function to be called receives **three** parameters from JNI - 
1. JNIEnv *env,        /* interface pointer */
2. jobject obj,        /* "this" pointer */
3. ... rest of the arguments

Guess what I did.... Yep, I forgot one of the default arguments - the `jobject` to be precise and was treating that as the arguments that I passed. 
And let me tell you, spending hours trying to decode the error while changing an object into an array because you used the wrong arguments is not fun, especially when no one else seemed to have that issue (cause people read the docs properly 🤦)

Oh well, stupidity aside, this could have been avoided if I used an interface generator like [flapigen-rs](https://github.com/Dushistov/flapigen-r) or some LLM to generate the interface.

So the basic workflow of the app is going to be like: 
1. **Image Input:** The user provides an image input through the app's interface.
2. **Image Processing:** The image is converted to a grayscale `byteArray` in Kotlin.
3. **JNI Bridge:** The grayscale `byteArray` is passed to a Rust function via JNI.
4. **Rust Processing:** The Rust function calls the `forward` method from the `burn` library, using a pretrained MNIST ONNX model to perform inference.
5. **Result Handling:** The result, an integer representing the predicted digit, is logged to the android console and returned from Rust to Kotlin.
6. **Output Display:** The predicted digit is displayed on the screen

Let's start!
We need to initialize the native library and write the prototype of the native function, before using it in kotlin 
```kotlin
  // You can write this in the init of your main class or wherever you need to load it
  init {
    System.loadLibrary("mnist_inference_android")
  }

  // Pay attention to the file you are writing this prototype in. You will need to use that name in the native function...
  external fun infer(inputImage: ByteArray): Int;

```
Next in the basic activity, I added a button which launched this launcher:
```kotlin
    val pickImageLauncher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.GetContent()
    ) { uri: Uri? ->
        uri?.let {
            val byteArray = uriToByteArray(context, it);
            if(byteArray != null){
                result = infer(byteArray)
            }
        }
    }
```
I planned to do the image processing part in kotlin because I am still not too familiar around rust (but getting there!), so the `urlToByteArray()` does the conversion of the image to a grayscale array.
We'll first downsize the bitmap, then convert it to a grayscale array of int's using the [NTSC formula](https://en.wikipedia.org/wiki/Grayscale#Luma_coding_in_video_systems) 
```kotlin
fun uriToByteArray(context: Context, uri: Uri): ByteArray? {
    val inputStream = context.contentResolver.openInputStream(uri) ?: return null
    val byteArray = inputStream.readBytes()
    val imageMap = BitmapFactory.decodeByteArray(byteArray, 0, byteArray.size)

    // The model takes 28x28 images as input so reduce size before grayscale conversion
    val reducedMap = Bitmap.createScaledBitmap(imageMap, 28, 28, false)

    val pixelArray = convertToGrayscaleArray(reducedMap)
    return pixelArray
}

fun convertToGrayscaleArray(bmp: Bitmap): ByteArray {
    // Create a mutable bitmap with the same dimensions as the original
    val width = bmp.width
    val height = bmp.height

    val grayscaleArray = ByteArray(width * height)
    // Iterate over each pixel in the original bitmap
    for (y in 0 until height) {
        for (x in 0 until width) {
            // Get the pixel color at (x, y)
            val pixel = bmp.getPixel(x, y)

            val r = Color.red(pixel)
            val g = Color.green(pixel)
            val b = Color.blue(pixel)
            // Converting to grayscale using the NTSC formula
            val gray = (0.299 * r + 0.587 * g + 0.114 * b).toInt()
            grayscaleArray[x + y * width] = gray.toByte() // Can also use the int array directly
        }
    }
    return grayscaleArray
}
```
There is no specific reason I used a byteArray to store the result. You can use whatever you like/whichever is optimum....

The app side is done!
Onto the Rust one!
My file name (where I wrote the prototype) is called `MnistInferPage` so my rust function will become:
```rust
pub extern "C" fn Java_com_example_mnistinferenceandroid_MnistInferPageKt_infer(
    env: JNIEnv,
    _: jobject,
    inputImage: JByteArray,
) -> jint
```
(the name of the function has `MnistInferPageKt` in it before the function name)

And the function definition is just to call the model with that input. 
```rust
    // Used to log to the android device's console since regular print statements won't show there, we use the android_logger package
    android_logger::init_once(Config::default().with_max_level(LevelFilter::Trace));

    let input = env
        .convert_byte_array(&inputImage)
        .expect("Error converting byteArray to Int vectors")
        .into_iter()
        .map(f32::from)
        .collect::<Vec<f32>>();

    // Just a POC, usually this would be done in an init function and not in each call
    type Backend = NdArray<f32>;
    let device = <Backend as burn::tensor::backend::Backend>::Device::default();

    let model: Model<Backend> = Model::default();

    // Reshape from the 1D array to 3d tensor [batch, height, width]
    let input =
        Tensor::<Backend, 1>::from_floats(input.as_slice(), &device).reshape([1, 1, 28, 28]);

    // Normalize input: make between [0,1] and make the mean=0 and std=1
    // values mean=0.1307,std=0.3081 were copied from Pytorch Mist Example
    // https://github.com/pytorch/examples/blob/54f4572509891883a947411fd7239237dd2a39c3/mnist/main.py#L122
    let input = ((input / 255) - 0.1307) / 0.3081;

    // Run the tensor input through the model
    let output: Tensor<Backend, 2> = model.forward(input);

    let res = match i32::try_from(output.argmax(1).into_scalar()) {
        Ok(val) => {
            debug!("The number is: {}", val);
            val
        }
        Err(_) => {
            debug!("Model output error!");
            -1
        }
    };
    res
```

Voila!! You got an mnist digit detecting app....

[Source code here](https://github.com/Scramjet911/burn/tree/mnist-android-example/examples/mnist-inference-android)
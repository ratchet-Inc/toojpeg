# A JPEG encoder in a single C++ file

*This is a mirror of my library hosted at* https://create.stephan-brumme.com/toojpeg/

TooJpeg is a compact baseline JPEG/JFIF writer, written in C++11 (but looks like C for the most part).  
Its interface has only one function: `writeJpeg()` - and that's it !

My library supports the most common JPEG output color spaces:
- YCbCr444,
- YCbCr420 (=2x2 downsampled) and
- Y (grayscale)

Saving NASA's huge 21600x10800px [blue marble image](https://eoimages.gsfc.nasa.gov/images/imagerecords/57000/57752/land_shallow_topo_21600.tif)
with quality=90 takes just 2.4 (YCbCr420) or 3.6 seconds (YCbCr444) on a standard x64 machine - faster than other small JPEG encoders.  
The compiled library enlarges your binary by about 7kb (CLang x64) or 12kb (GCC x64) in -O3 mode.  
Far more details can be found on the project homepage: https://create.stephan-brumme.com/toojpeg/

# How to use

1. create an image with any content you like, e.g. 1024x768, RGB (3 bytes per pixel)

```cpp
auto pixels = new unsigned char[1024*768*3];
```

2. define a callback that receives the compressed data byte-by-byte 

```cpp
// for example, write to disk (could be anything else, too, such as network transfer, in-memory storage, etc.)
void myOutput(unsigned char oneByte) { fputc(oneByte, myFileHandle); }
```

3. start JPEG compression

```cpp
TooJpeg::writeJpeg(myOutput, mypixels, 1024, 768);
// actually there are some optional parameters, too
//bool ok = TooJpeg::writeJpeg(myOutput, pixels, width, height, isRGB, quality, downSample, comment);
```
# How to use Concurrency/Threaded (Appended)

1. Initialize the new controller, or even your extended version of the controller

```cpp
// I recommend to use the new operator
TooJPEG_Controller *inst = new(std::nothrow) TooJPEG_Controller();
if(inst == nullptr || !inst->IsValid()){
	return 1;
}
```

2. Encode and get your encoding

```cpp
// same parameters as the original writeJpeg function
inst->Encode(pixels, width, height, isRGB, quality, downSample, comment);
// fetch the finished encoding, you can override the class function to receive byte by byte like writeJpeg
unsigned int len = 0;
unsigned char* ptr = inst->GetEncoded(len);
// example outputs
std::cout << ptr[0] << .... << ptr[len -1] << std::endl;
fileOut(ptr, len);	// fileOut = std::fstream
```

3. Release the instance

```cpp
// never forget to free your memory
delete inst;
```

4. Thread it

```cpp
// note this is an arbitrary algorithm, to paint a picture.
void Thread_Func(&workingData){
	step 1;
	step 2;
	step 3;
}
int main(void){
	pixels = [1280*720*3];
	pixelsArr[lim] = {pixels, ..., pixels};
	std::thread t[lim];
	for(index < lim){
		t[index] = std::thread(Thread_Func, pixelsArr[index]);
	}
	// encodings occur concurrently
	for(index < lim){
		t[index].join();
	}
	return 0;
}
```

# Note:

-My modication to the code were as minimal as possible to achieve instancing, for thread safety, while retaining the exact functionality of the original code.

-Due to my modifications, a slight degradation in performance of about %6.0-%10.0 is observed.
*Luckily, this is only for the first call of the funciton, due to modern CPU branch prediction, well at least on my AMD Ryzen CPU, all subsequent calls have a runtime that is equivalent to the unmodified code.

# Modifications Details:

-reduced the frame stack size of the encoding function for safety reasons, this was done my allocating
some variables dynamically, the new stack size is about 15MB, this is still alot but less than the original 22MB.
-added a structure and an optional parameter to prevent new allocation of memory when performing new encodings
-added a new class to handle encoding, which enables instancing.
-added a new typedef for member functions of the controller class
-modified the BitWritter struct by overloading the constructor to accept either a free function or
a member function with a class object and output to the valid pointer.


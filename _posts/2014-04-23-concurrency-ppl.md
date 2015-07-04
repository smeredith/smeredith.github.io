---
layout: post
title: Concurrency in C++ using Parallel Patterns Library
---

In this article, I present two variations of an algorithm to find the overall average color of an image using C++.
First I show a serial approach that uses a basic for loop.
Then I refactor the code in several steps to a multithreaded version that will take advantage of all available processors.

## Setup

For this code, I use C++ in Visual Studio 2013.
To decode jpg files into bitmaps, I use Microsoft's Windows Imaging Component (WIC) <http://msdn.microsoft.com/en-us/library/windows/desktop/ee719902(v=vs.85).aspx>.
For multithreading, I use Microsoft's Parallel Patterns Library (PPL) <http://msdn.microsoft.com/en-us/library/dd492418.aspx>.
I assume you know about the C++ standard library and how to use lambdas.

## Problem Statement

A bitmap is a series of pixels where each pixel is made up of a set of color component values that collectively describe the full color of the pixel.
For this exercise, I limit the input to bitmaps where each pixel is described by a red, green, and blue component and where each component is exactly one byte.
The "average color" is a single pixel whose red value is the average of all the red values in the bitmap, whose green value is the average of all the green values, and whose blue value is the average of all the blue values.

I use the term "scanline" to refer to a single horizontal line of pixels in an image.

## Simplest Approach

The simplest approach is to write a for loop to visit each pixel.
For each pixel add the red value to the grand total of the red values, add the green value to the grand total of the green values, and add the blue value to the grand total of the blue values.
Then divide each grand total by the number of pixels and you have three averages that together represent the average color.
See `AverageColorSerial()` at the end of the article for the full function.
The parallel version is there as well.

My buffer to hold the image data is a vector of bytes:

```cpp
// Make a buffer to hold the decoded image.
std::vector<BYTE> buffer(pixelCount * colorChannels);
```

where `colorChannels` is a constant with the value 3, representing the number of color components: red, green, and blue.
There is some WIC code early in the function that takes care of opening the file and loading the required decoder and decoding the bitmap.
I have eliminated that from the discussion for clarity, but you can look at the entire function at the end of the article to see how it fits in.
At this point, assume buffer contains a decoded bitmap.

I need to keep track of the sums for each color.
I chose to use an array with three elements for this with one element per color sum:

```cpp
// Store the totals needed to calculate an average, one for each channel.
std::array<ULONGLONG, colorChannels> totals = {0,0,0};
```

The array holds values of type `ULONGLONG`.
Each value needs to have enough capacity to store the sum of the 1-byte color values for every pixel.
`ULONGLONG` is big enough to process an image with 7,234,017,283,807,667 pixels.
That should suffice for the near future.

The next step is to create the for loop to visit each pixel and add the color values for that pixel to the totals:

```cpp
// Iterate through every color channel of every pixel and add to the total.
for (size_t i = 0; i < buffer.size();)
{
  for (size_t c = 0; c < colorChannels; ++c)
  {
    totals[c] += buffer[i++];  
  }
}
```

I chose for my function to return an `HRESULT` and use a `DWORD` reference parameter (`averageColor`) to return the average color.
At this point in the code there's nothing left that can fail so it's safe to modify the out parameter:

```cpp
averageColor = 0;

// Calculate the averages and create a single value representing the average color.
for (size_t i = 0; i < totals.size(); ++i)
{
  averageColor += (0xff & (totals[i] / pixelCount)) << (8 * i);
}
```

The line in this loop is taking a grand total for one of the colors and dividing it by the number of pixels to get an average.
Then it shifts the resulting average, a byte, into the correct position for combining the colors into a `DWORD`. Note that `totals[i] / pixelCount` is guaranteed to fit into a byte because it was created by summing `pixelCount` byte values.

That's it.
It works and it's pretty fast.
On my machine, it can compute the average color for a 100 million pixel image in about 450 ms.
However, it leaves 7 of my 8 processors idle while doing it.
This algorithm is a good candidate for parallel processing.

Microsoft has a library called _Parallel Patterns Library_ (<http://msdn.microsoft.com/en-us/library/dd492418.aspx>) that offers help in converting normal loops into parallel loops.
It has a `parallel_for()` that is similar to a normal for loop and a `parallel_for_each()` that operates on STL containers.
It will require a little extra refactoring but I chose the `parallel_for_each()` version for this exercise because it is a little faster.
So the first step is to refactor the hand-rolled for loop into a `std::for_each()` loop and make sure that works.
I'll switch to the parallel version later.

## Refactoring to Use std::for_each()

This step does nothing to make the function multi-threaded.
It's only a temporary step on the way and will not change the behavior or performance characteristics of the code.
Note that I created a set of unit tests for the previous version so I can quickly and easily refactor this code without fear of breaking the behavior.
The time spent creating those unit tests will pay off again and again as I continue to refactor the code towards a parallel version.

Since `std::for_each()` will visit each element in the buffer and each element is a byte, I need to keep track of whether or not I am visiting a red, green, or blue byte.
I could do this by incrementing a running count inside the loop but this won't work later when I want a parallel version.
So instead of a vector of bytes, I will create a vector of triple bytes, one for each color component.
Internally, the buffer is still a contiguous section of memory but now I can iterate over it one pixel (three bytes) at a time and the inner loop will have access to all three colors at once.

I define a new type to hold all three colors, which I call `PixelColors`:

```cpp
typedef std::array<BYTE, colorCount> PixelColors;
```

My buffer now holds elements of this type and contains one element per pixel:

```cpp
std::vector<PixelColors> buffer(pixelCount);
```

Here is the new version of the for loop:

```cpp
// Iterate through every color channel of every pixel and add to the total.
std::for_each(buffer.begin(), buffer.end(), 
  [&totals](PixelColors& colors)
  {
    for (size_t c = 0; c < totals.size(); ++c)
    {
      totals[c] += colors[c];
    }
  });
```

Note that I have to capture `totals` by reference since the lambda body is modifying it.

At this point, I could simply replace `std::for_each()` with `concurrency::parallel_for_each()` and replace `totals` with a concurrent_vector (or a combinable) and I would have a parallel version.
But it would be slower than the serial version.
This is because the unit of work inside the for loop is very small and the overhead of parallelizing the work is large in comparison.
In addition access to the totals vector has to be synchronized and there will contention on the lock.
What I need to do is break the work up into larger chunks and parallelize those.
A scanline is an obvious chunk that will serve this purpose.
The unit of work will be to calculate the color sums for a single scanline and I will parallelize those units.
Once all that work is complete, the code will sum the sums to get a grand total for each color and get the average as before.

## One Scanline at a Time

The next step is to convert the existing loop into a nested loop.
The outer loop will visit each scanline and the inner loop will calculate the color sums for a single scanline.
I won't introduce threading at this step.
I will refactor and make sure my unit tests still pass.

At this point, I define a new type to represent three color sums and a const to represent a sum's initial value:

```cpp
// PixelColorSum is a type to hold three different sums: one for each color.
typedef std::array<ULONGLONG, colorCount> PixelColorSum;
const PixelColorSum colorSumZero = {0,0,0};
```

I am going to keep the buffer as in the previous step but I'm going to create a typedef for it because I need to use its iterators belonging to that type later:

```cpp
typedef std::vector<PixelColors> PixelColorVector;
PixelColorVector buffer(pixelCount);
```

In addition, I am going to introduce a vector to represent the scanlines.
Each element will contain an iterator to the start of a scanline in the buffer, an iterator to the end of the scanline, and a value to hold the result of the operation to sum the colors of that scanline.
Each element is passed to `std::for_each()` to perform that calculation.
I could use a struct as the type of the elements in the scanline vector but I chose to use an `std::tuple` instead.
The first and second values are the iterators and the third value is the color sum:

```cpp
// Want to process scanlines in parallel, so create a list of each scanline's begin
// and end. The color sum for that line will be stored in this vector as well.
typedef std::tuple<PixelColorVector::iterator, PixelColorVector::iterator, PixelColorSum> ScanLine;
std::vector<ScanLine> scanLines(height);
```

Next, I initialize that vector of scanlines:

```cpp
for (size_t l = 0; l < height; ++l)
{
  PixelColorVector::iterator begin = buffer.begin() + (width * l);
  scanLines[l] = std::make_tuple(begin, begin + width, colorSumZero);
}
```

Then I need an outer loop to visit each scanline and an inner loop to process it.
There is also a tiny inner-inner loop to process the three colors:

```cpp
// Iterate through the scan lines.
std::for_each(scanLines.begin(), scanLines.end(),
  [](ScanLine& scanLine)
  {
    // Calculate the color sums for this line and save it.
    std::for_each(std::get<0>(scanLine), std::get<1>(scanLine),
      [&scanLine](PixelColors& pixelColors)
      {
        for (size_t c = 0; c < pixelColors.size(); ++c)
        {
          // Save it back into the scanline array.
          std::get<2>(scanLine)[c] += pixelColors[c];
        }
    });
  });
```

At this point, the scanlines array has a subtotal for each line in the third part of the tuple.
Sum those in a "reduce" or "merge" step:

```cpp
// Sum the sums of each scanline to get the grand total.
PixelColorSum grandTotal = std::accumulate(scanLines.begin(), scanLines.end(), colorSumZero,
  [](PixelColorSum colorSum, ScanLine& scanLine)
  {
    for (size_t c = 0; c < colorSum.size(); ++c)
    {
      colorSum[c] += std::get<2>(scanLine)[c];
    }
    return colorSum;
  });
```

Now I have a grand total for each color and I continue as before.
All unit tests still pass.

### Parallel Version

I've done all the hard work.
The last step is to replace the outer loop `std::for_each()` with `concurrency::parallel_for_each()`:

```cpp
// Iterate through the scan lines.
concurrency::parallel_for_each(scanLines.begin(), scanLines.end(),
  [](ScanLine& scanLine)
  {
    // Calculate the color sums for this line.
    PixelColorSum scanLineColorSum = {0,0,0};
    std::for_each(std::get<0>(scanLine), std::get<1>(scanLine),
      [&scanLineColorSum](PixelColors& pixelColors)
      {
        for (size_t c = 0; c < pixelColors.size(); ++c)
        {
          scanLineColorSum[c] += pixelColors[c];
        }
    });
 
    // Save it back into the scanline array. Note that multiple threads are
    // writing to the array but each one is writing to a different pre-allocated
    // element so it is thread safe.
    std::get<2>(scanLine) = scanLineColorSum;
  });
```

Note that multiple threads are accessing the scanlines array. That is safe because each thread only writes to the one element in the array that it is processing and the array itself is not being changed.

## Conclusion

I measured the time to process a 100 million pixel image using the original serial algorithm and the final parallel one.
The serial algorithm took around 450 ms to complete while the parallel version took around 125 ms.
This is on my 8 core dev machine and the times bounced around a little on different measurement runs.
I excluded the time to load the file from disk and decode it and only measured the time used by the code discussed here.

The PPL made the task relatively easy by abstracting away the threading details.
It did take a few hours to refactor the code and I still needed to think carefully about what I was doing.
But the resulting performance gain is substantial and the PPL will automatically scale to the number of processors on the machine running the program.
Now I just need a few thousand more cores.

I have included the full source of both the initial serial version and the parallel version of the function below.
For the full project, please see <http://www.github.com/smeredith/averagecolor>.
Look for the release tagged "blog" to get the exact version discussed here.
You will find other experiments in the project testing the performance of different concurrency techniques on this same problem.
The code has since been refactored so that it's not all one function but it should be close enough for you to understand the changes.

## Serial Version

```cpp
#include "stdafx.h"
#include <array>
#include <vector>
#include <algorithm>
#include <numeric>
#include <ppl.h>
#include <wincodec.h>

using namespace Microsoft::WRL;

HRESULT AverageColorSerial(PCWSTR filename, DWORD& averageColor)
{
    VERIFY_PARAM(filename);

    ComPtr<IWICImagingFactory> pWICFactory;
    IF_FAIL_RETURN(CoCreateInstance(
                CLSID_WICImagingFactory,
                nullptr,
                CLSCTX_INPROC_SERVER,
                IID_PPV_ARGS(&pWICFactory)
                ));

    ComPtr<IWICBitmapDecoder> pDecoder;
    IF_FAIL_RETURN(pWICFactory->CreateDecoderFromFilename(
                filename,
                nullptr,                         // Do not prefer a particular vendor
                GENERIC_READ,
                WICDecodeMetadataCacheOnDemand,  // Cache metadata when needed
                &pDecoder
                ));

    ComPtr<IWICBitmapFrameDecode> pFrame;
    IF_FAIL_RETURN(pDecoder->GetFrame(0, &pFrame));

    // The only pixel format info we can get from the frame is a GUID.
    WICPixelFormatGUID pixelFormatGuid;
    IF_FAIL_RETURN(pFrame->GetPixelFormat(&pixelFormatGuid));

    // Only need for handle this format for now. Code below assumes this format.
    IF_FALSE_RETURN((pixelFormatGuid == GUID_WICPixelFormat24bppBGR), E_UNEXPECTED);

    const UINT colorChannels = 3;
    const UINT bitsPerChannel = 8;

    UINT width;
    UINT height;
    IF_FAIL_RETURN(pFrame->GetSize(&width, &height));

    // Make a buffer to hold the decoded image.
    UINT pixelCount = width * height;
    std::vector<BYTE> buffer(pixelCount * colorChannels);

    // Decode the image into the buffer.
    IF_FAIL_RETURN(pFrame->CopyPixels(
                0, // entire bitmap
                width * colorChannels, // stride
                buffer.size(),
                buffer.data()
                ));

    // Store the totals we need to calculate an average, one for each channel.
    std::array<ULONGLONG, colorChannels> totals = {0,0,0};

    // Iterate through every color channel of every pixel and add to the total.
    for (size_t i = 0; i < buffer.size();)
    {
        for (size_t c = 0; c < colorChannels; ++c)
        {
            totals[c] += buffer[i++];
        }
    }

    averageColor = 0;

    // Calculate the averages and create a single value representing the average color.
    for (size_t i = 0; i < totals.size(); ++i)
    {
        averageColor += (0xff & (totals[i] / pixelCount)) << (8 * i);
    }

    return S_OK;
}
```

## Parallel Version

```cpp
#include "stdafx.h"
#include <array>
#include <vector>
#include <algorithm>
#include <numeric>
#include <ppl.h>
#include <wincodec.h>

using namespace Microsoft::WRL;

HRESULT AverageColorParallel(PCWSTR filename, DWORD& averageColor)
{
    VERIFY_PARAM(filename);

    ComPtr<IWICImagingFactory> pWICFactory;
    IF_FAIL_RETURN(CoCreateInstance(
                CLSID_WICImagingFactory,
                nullptr,
                CLSCTX_INPROC_SERVER,
                IID_PPV_ARGS(&pWICFactory)
                ));

    ComPtr<IWICBitmapDecoder> pDecoder;
    IF_FAIL_RETURN(pWICFactory->CreateDecoderFromFilename(
                filename,
                nullptr,                         // Do not prefer a particular vendor
                GENERIC_READ,
                WICDecodeMetadataCacheOnDemand,  // Cache metadata when needed
                &pDecoder
                ));

    ComPtr<IWICBitmapFrameDecode> pFrame;
    IF_FAIL_RETURN(pDecoder->GetFrame(0, &pFrame));

    // The only pixel format info we can get from the frame is a GUID.
    WICPixelFormatGUID pixelFormatGuid;
    IF_FAIL_RETURN(pFrame->GetPixelFormat(&pixelFormatGuid));

    // Only need for handle this format for now. Code below assumes this format.
    IF_FALSE_RETURN((pixelFormatGuid == GUID_WICPixelFormat24bppBGR), E_UNEXPECTED);

    const UINT colorCount = 3;
    const UINT bitsPerChannel = 8;

    UINT width;
    UINT height;
    IF_FAIL_RETURN(pFrame->GetSize(&width, &height));

    // Make a buffer to hold the bytes of the decoded image. This is a vector of 3-byte
    // arrays, where each byte in the array holds the RGB values for a single pixel. This
    // is so we can iterate through 1 pixel at a time instead of 1 byte at a time.
    typedef std::array<BYTE, colorCount> PixelColors;
    typedef std::vector<PixelColors> PixelColorVector;
    const UINT pixelCount = width * height;
    PixelColorVector buffer(pixelCount);

    // Decode the image into the buffer. The buffer appears as a single array of BYTES to
    // this function.
    IF_FAIL_RETURN(pFrame->CopyPixels(
                0, // entire bitmap
                width * colorCount, // stride
                pixelCount * colorCount, // buffer size
                reinterpret_cast<BYTE*>(buffer.data())
                ));

    // PixelColorSum is a type to hold three different sums: one for each color.
    typedef std::array<ULONGLONG, colorCount> PixelColorSum;
    const PixelColorSum colorSumZero = {0,0,0};

    // Want to process scanlines in parallel, so create a list of each scanline's begin
    // and end iterators. The color sum for that line will be stored in this vector as
    // well, as the third part of the tuple.
    typedef std::tuple<PixelColorVector::iterator, PixelColorVector::iterator, PixelColorSum> ScanLine;
    std::vector<ScanLine> scanLines(height);

    for (size_t l = 0; l < height; ++l)
    {
        PixelColorVector::iterator begin = buffer.begin() + (width * l);
        scanLines[l] = std::make_tuple(begin, begin + width, colorSumZero);
    }

    // Iterate through the scan lines in parallel.
    concurrency::parallel_for_each(scanLines.begin(), scanLines.end(),
        [](ScanLine& scanLine)
        {
            // Calculate the color sums for this line.
            PixelColorSum scanLineColorSum = { 0, 0, 0 };
            std::for_each(std::get<0>(scanLine), std::get<1>(scanLine),
                [&scanLineColorSum](PixelColors& pixelColors)
                {
                    for (size_t c = 0; c < pixelColors.size(); ++c)
                    {
                        scanLineColorSum[c] += pixelColors[c];
                    }
                });

            // Save it back into the scanline array. Note that multiple threads are
            // writing to the array, but each one is writing to a different pre-allocated
            // element, so it is thread safe.
            std::get<2>(scanLine) = scanLineColorSum;
        });

    // Sum the sums of each scanline to get the grand total.
    PixelColorSum grandTotal = std::accumulate(scanLines.begin(), scanLines.end(), colorSumZero,
        [](PixelColorSum colorSum, ScanLine& scanLine)
        {
            for (size_t c = 0; c < colorSum.size(); ++c)
            {
                colorSum[c] += std::get<2>(scanLine)[c];
            }
            return colorSum;
        });

    averageColor = 0;

    // Calculate the averages and create a single value representing the average color.
    for (size_t i = 0; i < colorCount; ++i)
    {
        averageColor += (0xff & (grandTotal[i] / pixelCount)) << (bitsPerChannel * i);
    }

    return S_OK;
```

## CSCI 5607 — HW2: Image Conversion & Processing

This program loads images and performs a sequence of operations from the command line. You can convert formats, adjust brightness, quantize, crop, extract a channel, and write PPMs at specific bit depths.

Build
g++ -fsanitize=address -std=c++14 main.cpp pixel.cpp image.cpp -o image


Requires a C++14 compiler.

Uses [stb_image.h] and [stb_image_write.h] (already included in this repo).

Usage

The program applies options in order:

./image -input <infile> [ops…] -output <outfile>


Examples:

-brightness <factor> (e.g., 1.4)

-quantize <nbits> (e.g., 2 → 4 uniform levels per channel)

-ppm_depth <nbits> (affects only PPM output header/maxval; values are scaled accordingly)

-crop <x y w h> (top-left + width/height)

-extractChannel <c> (0: Red, 1: Green, 2: Blue)

Get all options:

./image -help

Results (generated files)

All outputs below are in Results/ and were produced with the shown commands.

1) Baseline copy (sanity)

Command

./image -input SampleImages/goldy.ppm -output Results/goldy_copy.ppm


What to expect

Header lines identical except whitespace.

Data identical (Max=255).

2) Quantization (internal levels only)

Commands

./image -input SampleImages/goldy.ppm -quantize 2 -output Results/goldy_quant2.png
./image -input SampleImages/goldy.ppm -quantize 2 -output Results/goldy_quant2.ppm


What it does

Reduces the in-memory channel values to 4 equally spaced levels (2^2).

Output format can still be PNG/JPEG/PPM with normal 8-bit range; you’ll see visible banding.

3) PPM file depth (header maxval & value scaling)

Write as 2-bit (Max=3)

./image -input SampleImages/goldy.ppm -ppm_depth 2 -output Results/goldy_depth2.ppm


Write as 8-bit (Max=255)

./image -input SampleImages/goldy.ppm -ppm_depth 8 -output Results/goldy_depth8.ppm


What it does

Sets the PPM header’s third line to 2^nbits - 1 and rescales pixel values accordingly.

Uses round-to-nearest integer scaling:

Write (255 → Max): (v * Max + 255/2) / 255

Read (Max → 255): (v * 255 + Max/2) / Max

Example for 2-bit PPM (Max=3): channel values become {0, 1, 2, 3} instead of {0..255}.

4) Crop

Command

./image -input SampleImages/goldy.ppm -crop 250 50 100 150 -output Results/goldy_cropped.ppm


What it does

Extracts the rectangle at (x=250, y=50) with width=100, height=150.

The output PPM header’s width/height should be 100 150.

Implementation gist

Image* out = new Image(w,h);
for (int j = 0; j < h; ++j)
  for (int i = 0; i < w; ++i)
    out->SetPixel(i, j, GetPixel(x + i, y + j));
return out;

5) Extract a single channel

Green channel (1) after cropping

./image -input SampleImages/goldy.ppm -crop 250 50 100 150 -extractChannel 1 \
  -output Results/goldy_cropped_green.jpeg


What it does

Keeps only the green component (channel=1), zeros the others:

channel=0 → keep Red, zero G, B

channel=1 → keep Green, zero R, B

channel=2 → keep Blue, zero R, G

6) Pipeline (brightness → crop → extract)

Command

./image -input SampleImages/goldy.ppm -brightness 1.4 -crop 250 50 100 150 -extractChannel 0 \
  -output Results/goldy_cropped_bright_red.jpg


What it does

Scales intensity by 1.4 (clamped to 0..255), then crops, then keeps only red.


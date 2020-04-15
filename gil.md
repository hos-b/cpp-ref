# Boost General Image Library

## naming conventions in GIL
```cpp
_ColorSpace_ + _BitDepth_ + [f | s]+ [c] + [_planar] + [_step] + _ClassType_ + _t
bgr8_image_t             a;    // 8-bit interleaved BGR image
cmyk16_pixel_t           b;    // 16-bit CMYK pixel value;
cmyk16c_planar_ref_t     c(b); // const reference to a 16-bit planar CMYK pixel.
rgb32f_planar_step_ptr_t d;    // step pointer to a 32-bit planar RGB pixel.
```

## creating image view from raw data
```cpp
const unsigned char* src_pixels, ptrdiff_t src_row_bytes;
int w,h;
boost::gil::gray8c_view_t src = interleaved_view(w, h, (const gray8_pixel_t*)src_pixels,src_row_bytes);

```
where src_row_bytes is the amount of bytes per row
## Iteration
### pixel access
```cpp
src(x,y) = (src(x,y) + 2) / 2;
```
### iterators
faster than `src(x,y)`:
```cpp
for (int y = 0; y < src.height(); ++y)
{
    gray8c_view_t::x_iterator src_it = src.row_begin(y);
    for (int x = 0; x < src.width(); ++x)
        src_it[x] = (src_it[x] + 2) / 2
}
```
here the iterator type is raw C pointer and operator[] is a fast pointer indexing operator.
### iterating with GIL locators
an iterator that can move horizontally and vertically.
```cpp
// locator pointing to the first pixel of the second row
gray8c_view_t::xy_locator src_loc = src.xy_at(0,1);
// advance in x dimension
++src_loc.x()
// advance in y direction
++src_loc.y()
// neighbor below
src_loc(0, 1) = 0;
// neighbor above
src_loc(0, -1) = 1;
```
## changing view
```cpp
const rgb32fc_view_t src;
auto src_gray = color_converted_view<gray8_pixel_t>(src);
```
this is a shallow function and only changes format on pixel access. this makes it inefficient for use cases with many pixel access.
## changing format
change it alltogether
```cpp
const rgb32fc_view_t src;
gray8_image_t dest_ccv_image(src.dimensions());
copy_pixels(color_converted_view<gray8_pixel_t>(src), view(dest_ccv_image));
```

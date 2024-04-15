## Porting JuliaCon 2021 poster from Julia to Python

[images-as-matrices.ipynb](images-as-matrices.ipynb) - reproducing main matrix multiplication effects modulo some relatively mild differences probably due to suspected precision differences

Thanks to GPT-4 help while doing this:

https://chat.openai.com/share/9e12bb7c-90de-4e5c-91e3-924290f64332

https://chat.openai.com/share/215283b4-eefc-46c0-afe5-27e44deddbdc

https://chat.openai.com/share/9025d68d-c40e-4b20-a321-7b80dac62580

(Intermediate draft Python notebooks are available upon request)

---

Further investigation:

```python
# Function to normalize columns
def norm_columns(f, x):
    return f(x) / np.sum(f(x), axis=0)

# Function to normalize rows
def norm_rows(f, x):
    return f(x) / np.sum(f(x), axis=1)[:, np.newaxis] # it would be nice to understand this better
```

I asked GPT-4 to clarify the difference here:

https://chat.openai.com/share/d01e5b68-f1f5-403a-9976-1203ad4cd65c

---

I am going to introduce a new function for saving and displaying monochrome images:

```python
def save_show(monochrome_image, file_name):
    monochrome_image_uint8 = (monochrome_image * 255).astype(np.uint8)
    cv2.imwrite(file_name, monochrome_image_uint8)
    display(Image(filename=file_name))
```

[first-followup.ipynb](first-followup.ipynb) - a small clean notebook incorporating these changes and
investigating whether differences between Julia and Python are due to float32 and float64 difference.

In that sense, it turns out that float32 and float64 are indistinguishable, we still need to figure out
why matrix multiplication followed by `normalize_image` produces slightly different results
in Julia and in Python.

---

Ah, OK, this mystery has a very simple solution: obtained monochrome versions of `mandrill` are
somewhat different between Julia and Python.

This is less mysterious (so they have a slightly different conversions from color to monochrome for some
reason, that's not implausible).

I did verify that color images are the same. 

Julia documentation https://juliaimages.org/latest/function_reference/#Color-conversion says that

> `Gray` calculates a grayscale representation of a color image using the Rec 601 luma

and references https://en.wikipedia.org/wiki/Luma_%28video%29#Rec._601_luma_versus_Rec._709_luma_coefficients

So we just need to establish that `color.rgb2gray` in `skimage` uses the Rec. 709,
and the mystery will go away.

And, indeed, color.rgb2gray coefficients are not quite Rec. 709, but they are very close to that:
https://scikit-image.org/docs/stable/auto_examples/color_exposure/plot_rgb_to_gray.html

Yes, Python formula is exactly `Y = 0.2125 R + 0.7154 G + 0.0721 B` (which is not quite Rec. 709, but close):

https://github.com/scikit-image/scikit-image/blob/main/skimage/color/colorconv.py

Meanwhile, Julia code uses exactly Rec. 601, `0.299f0*red(x) + 0.587f0*green(x) + 0.114f0*blue(x)`, see the end of

https://github.com/JuliaGraphics/Colors.jl/blob/master/src/conversions.jl

**Conclusion:**: one can play with these coefficients if one feels like it.

---

[python-with-julia-monochrome.ipynb](python-with-julia-monochrome.ipynb) - reproduce Julia way of converting color to monochrome

---

This was continued by a collaborative effort here: https://github.com/Chernobilsky/Mishka-project

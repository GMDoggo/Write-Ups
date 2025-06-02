At the end of this lovely moment, n00psy gives you a picture of him. "I will never forget this moment, thank you for coming with me!", he says. "Also, sorry if the quality of this picture is a bit poor on the edges, this is an old picture that my friend Joseph took in 1807."


# Solve

Within the file appears to be data stored

![](Images/Pasted%20image%2020250601115002.png)

After digging down that rabbit hole too deep I started over. Looking at the hint `You have something, but can it be transformed ?`  that alongside "Joseph" in 1807 I finally put together this is supposed to be about a Fourier Transformation.

```
import numpy as np
import matplotlib.pyplot as plt
from PIL import Image

# Load image
img = Image.open('noopsy_selfie.png')
img_gray = img.convert('L')
img_array = np.array(img_gray)

# Apply 2D FFT
fft = np.fft.fft2(img_array)
fft_shift = np.fft.fftshift(fft)
magnitude = np.log(np.abs(fft_shift) + 1)

# Display
plt.figure(figsize=(12, 6))
plt.subplot(121)
plt.imshow(img_array, cmap='gray')
plt.title('Original Image')
plt.subplot(122)
plt.imshow(magnitude, cmap='gray')
plt.title('FFT Magnitude')
plt.show()
```

![](Images/Pasted%20image%2020250601130725.png)

```
N0PS{i_love_fourier}
```
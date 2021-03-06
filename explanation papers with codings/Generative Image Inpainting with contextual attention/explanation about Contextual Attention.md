# Image Inpainting with Contextual Attention
Convolutional neural networks process image features with local convolutional kernel layer by layer thus are not effective for borrowing features from distant spatial locations. To overcome the limitation, we consider attention
mechanism and introduce a novel contextual attention layer in the deep generative network. In this section, we first discuss details of the contextual attention layer, and then address how we integrate it into our unified inpainting network.
## Image Inpainting with Contextual Attention
The contextual attention layer learns where to borrow or copy feature information from known background patches to generate missing patches. It is differentiable, thus can be trained in deep models, and fully-convolutional, which allows testing on arbitrary resolutions.
![image](https://github.com/SherryCal/related-work-summary-and-tricks/blob/master/explanation%20papers%20with%20codings/Generative%20Image%20Inpainting%20with%20contextual%20attention/contextual%20attention%20structure.png)
### steps 
Illustration of the contextual attention layer.

1.we use convolution to compute matching score of foreground patches with background patches (as convolutional filters). 

2.we apply softmax to compare and get attention score for each pixel. 

3.we reconstruct foreground patches with background patches by performing deconvolution on attention score. The contextual attention layer is differentiable and fully-convolutional.
### Match and attend
We consider the problem where we want to match features of missing pixels (foreground) to surroundings (background).we first
extract patches (3 × 3) in background and reshape them as convolutional filters.
```
24    raw_w = tf.extract_image_patches(
25            b, [1,kernel,kernel,1], [1,rate*stride,rate*stride,1], [1,1,1,1], padding='SAME')
26        raw_w = tf.reshape(raw_w, [raw_int_bs[0], -1, kernel, kernel, raw_int_bs[3]])
27        raw_w = tf.transpose(raw_w, [0, 2, 3, 4, 1])  # transpose to b*k*k*c*hw
```
To match foreground patches {$f_{x,y}$}
with backgrounds ones {$b_{x′,y′}$}, we measure with normalized inner product (cosine similarity)
```
76    yi *=  mm  # mask
77            yi = tf.nn.softmax(yi*scale, 3)
78            yi *=  mm  # mask
79
80            offset = tf.argmax(yi, axis=3, output_type=tf.int32)
81            offset = tf.stack([offset // fs[2], offset % fs[2]], axis=-1)
82            # deconv for patch pasting
83            # 3.1 paste center
84            wi_center = raw_wi[0]
85            yi = tf.nn.conv2d_transpose(yi, wi_center, tf.concat([[1], raw_fs[1:]], axis=0), strides=[1,rate,rate,1]) / 4.
86            y.append(yi)
87            offsets.append(offset)
```
### Attention propagation
Attention propagation We further encourage coherency of attention by propagation (fusion). The idea of coherency is that a shift in foreground patch is likely corresponding to an equal shift in background patch for attention. To model and encourage coherency of attention maps, we do a left-right propagation followed by a top-down propagation with kernel size of k.

The propagation is efficiently implemented as convolution
with identity matrix as kernels. Attention propagation significantly improves inpainting results in testing and enriches
gradients in training
```
57    for xi, wi, raw_wi in zip(f_groups, w_groups, raw_w_groups):
58            # conv for compare
59            wi = wi[0]
60            wi_normed = wi / tf.maximum(tf.sqrt(tf.reduce_sum(tf.square(wi), axis=[0,1,2])), 1e-4)
61            yi = tf.nn.conv2d(xi, wi_normed, strides=[1,1,1,1], padding="SAME")
62
63            # conv implementation for fuse scores to encourage large patches
64            if fuse:
65                yi = tf.reshape(yi, [1, fs[1]*fs[2], bs[1]*bs[2], 1])
66                yi = tf.nn.conv2d(yi, fuse_weight, strides=[1,1,1,1], padding='SAME')
67                yi = tf.reshape(yi, [1, fs[1], fs[2], bs[1], bs[2]])
68                yi = tf.transpose(yi, [0, 2, 1, 4, 3])
69                yi = tf.reshape(yi, [1, fs[1]*fs[2], bs[1]*bs[2], 1])
70                yi = tf.nn.conv2d(yi, fuse_weight, strides=[1,1,1,1], padding='SAME')
71                yi = tf.reshape(yi, [1, fs[2], fs[1], bs[2], bs[1]])
72                yi = tf.transpose(yi, [0, 2, 1, 4, 3])
73            yi = tf.reshape(yi, [1, fs[1], fs[2], bs[1]*bs[2]])

```
### Memory efficiency
Assuming that a 64 × 64 region is missing in a 128 × 128 feature map, then the number of convolutional filters extracted from backgrounds is 12,288. This may cause memory overhead for GPUs. To overcome this issue, we introduce two options: 1) extracting background patches with strides to reduce the number of filters and 2) downscaling resolution of foreground inputs before convolution and upscaling attention map after propagation

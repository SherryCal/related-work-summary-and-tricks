# Image Inpainting with Contextual Attention
Convolutional neural networks process image features with local convolutional kernel layer by layer thus are not effective for borrowing features from distant spatial locations. To overcome the limitation, we consider attention
mechanism and introduce a novel contextual attention layer in the deep generative network. In this section, we first discuss details of the contextual attention layer, and then address how we integrate it into our unified inpainting network.
## Image Inpainting with Contextual Attention
The contextual attention layer learns where to borrow or copy feature information from known background patches to generate missing patches. It is differentiable, thus can be trained in deep models, and fully-convolutional, which allows testing on arbitrary resolutions.
![image](https://github.com/SherryCal/related-work-summary-and-tricks/blob/master/explanation%20papers%20with%20codings/Generative%20Image%20Inpainting%20with%20contextual%20attention/contextual%20attention%20structure.png)
### steps 
Illustration of the contextual attention layer.

1.  we use convolution to compute matching score of foreground patches with background patches (as convolutional filters). 

2.we apply softmax to compare and get attention score for each pixel. 

3.we reconstruct foreground patches with background patches by performing deconvolution on attention score. The contextual attention layer is differentiable and fully-convolutional.
### Match and attend
### Attention propagation
### Memory efficiency
Assuming that a 64 × 64 region is missing in a 128 × 128 feature map, then the number of convolutional filters extracted from backgrounds is 12,288. This may cause memory overhead for GPUs. To overcome this issue, we introduce two options: 1) extracting background patches with strides to reduce the number of filters and 2) downscaling resolution of foreground inputs before convolution and upscaling attention map after propagation
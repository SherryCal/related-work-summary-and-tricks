可以看出跟WGAN不同的主要有几处：1）用gradient penalty取代weight clipping；2）在生成图像上增加高斯噪声；3）优化器用Adam取代RMSProp。

这里需要注意的是，这个GP的引入，跟一般GAN、WGAN中通常需要加的Batch Normalization会起冲突。因为这个GP要求critic的一个输入对应一个输出，但是BN会将一个批次中的样本进行归一化，BN是一批输入对应一批输出，因而用BN后无法正确求出critic对于每个输入样本的梯度。

[return](/ai-pic/index)

```
在生成包含文字的图像方面，
目前表现最佳的Stable Diffusion版本是Stable Diffusion 3 Medium及其
后续的Stable Diffusion 3.5系列。

Stable Diffusion 3 Medium是一款多模态扩散变换器文本到图像模型，
它使用了三种固定的预训练文本编码器，包括OpenCLIP-ViT/G、CLIP-ViT/L和T5-xxl。

其包含CLIP功能的版本，如sd3_medium_incl_clips.safetensors，
以及进一步集成了T5-XXL-FP8模型的sd3_medium_incl_clips_t5xxlfp8.safetensors，
在文字生成能力上表现更为出色。
特别是sd3_medium_incl_clips_t5xxlfp8.safetensors，
它在保证一定图像生成质量的同时，对硬件资源的需求相对较低，只需12G显存。

Stable Diffusion 3.5系列则在Stable Diffusion 3的基础上进一步优化，
该系列中的Large版本（80亿参数）展现了卓越的提示词响应和细节表现，
在文本渲染能力上有显著提升，能够更准确地将文字融入图像中，生成视觉效果优异的内容。
其蒸馏版本Large Turbo则优化了生成速度，在生成时间和图像精细度之间达成平衡，
同时依旧保留了对提示词的高响应性和准确性，也适合需要生成包含文字图像的场景。
```
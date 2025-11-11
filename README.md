# Image Resize for ComfyUI

This repository is a **direct fork and continuation** of the archived project [`palant/image-resize-comfyui`](https://github.com/palant/image-resize-comfyui).  
- Original creator: **Wladimir Palant** (MIT License, archived on 2024-05-11).  
- Fork maintainer: **ussoewwin/image_resize_comfyui** — kept in sync with current ComfyUI releases, with packaging/docs refreshed so it keeps working today.

Like the upstream project, it delivers a powerful ComfyUI node for resizing images without distorting proportions. This fork focuses on keeping the code compatible with modern ComfyUI builds while preserving the original behavior. Optional mask handling remains fully supported, so masks stay aligned with the resized image.

![A ComfyUI node titled "Image resize" with inputs pixels and mask_optional, outputs IMAGE and MASK as well as a variety of widgets: action, smaller_side, larger_side, scale_factor, resize_mode, side_ratio, crop_pad_position, pad_feathering](image_resize.png)

## Features

- **Aspect Ratio Preservation**: Resize images while maintaining their original aspect ratio
- **Three Resize Modes**: Choose between resize-only, crop-to-ratio, or pad-to-ratio operations
- **Flexible Sizing Options**: Use smaller_side, larger_side, or scale_factor for intuitive control
- **Smart Resize Modes**: Reduce-only, increase-only, or unrestricted resizing
- **Mask Support**: Optional mask input is automatically resized and transformed alongside the image
- **Precision Cropping/Padding**: Fine-tune crop and pad positions with pixel-level control
- **Feathering Support**: Smooth mask transitions to prevent visible borders during inpainting

## Installation

Clone this repository into your `ComfyUI/custom_nodes` directory:

```bash
cd ComfyUI/custom_nodes
git clone https://github.com/ussoewwin/image-resize-comfyui
cd image-resize-comfyui
```

Then restart ComfyUI.

## Node Configuration

### `action`

Determines how the node processes the image after resizing:

- **`resize only`**: Image is resized while keeping its aspect ratio. The `side_ratio` setting is ignored.
- **`crop to ratio`**: After resizing, parts of the image are removed to match the target `side_ratio`. Useful for enforcing specific aspect ratios.
- **`pad to ratio`**: After resizing, transparent padding is added to match the target `side_ratio`. Original image content is always preserved.

### `smaller_side`, `larger_side`, `scale_factor`

Only one of these can be active (set to a non-zero value). They determine the target size:

#### `smaller_side` (Integer, 0-8192)
The smaller dimension of the image will be resized to this value.

**Example**: Original image 512×768 px, `smaller_side=1024` → Result: 1024×1536 px

#### `larger_side` (Integer, 0-8192)
The larger dimension of the image will be resized to this value.

**Example**: Original image 512×768 px, `larger_side=1024` → Result: 683×1024 px

#### `scale_factor` (Float, 0.0-10.0)
Direct scaling multiplier.

- `< 1.0`: Downscaling
- `> 1.0`: Upscaling
- `= 0.0`: No scaling, only crop/pad as needed

If all values are zero, the image is not resized but only cropped/padded as necessary.

### `resize_mode`

Controls which direction resizing is applied:

- **`reduce size only`**: Images already smaller than the target size will not be resized. Only downscaling occurs.
- **`increase size only`**: Images already larger than the target size will not be resized. Only upscaling occurs.
- **`any`**: Images are always resized to match the target, regardless of direction.

### `side_ratio` (String, default: "4:3")

Sets the target aspect ratio when using `crop to ratio` or `pad to ratio` actions.

**Format**: `width:height`

**Examples**:
- `4:3` – Standard aspect ratio
- `16:9` – Widescreen
- `1:1` – Square
- `512:768` – Explicit dimensions (when combined with `smaller_side=512`, always produces 512×768 px)

### `crop_pad_position` (Float, 0.0-1.0, default: 0.5)

Controls where cropping or padding occurs:

- `0.0`: All removal/addition on the right/bottom side
- `0.5`: Centered (even on both sides)
- `1.0`: All removal/addition on the left/top side
- `0.3`: 30% on left/top, 70% on right/bottom

**For Cropping**: Determines which parts are removed.
**For Padding**: Determines where transparent padding is inserted.

### `pad_feathering` (Integer, 0-8192, default: 20)

When padding is applied, this setting gradually expands mask transparency into the original image by the specified number of pixels. This helps avoid visible borders when the padded image is later used in inpainting operations.

- `0`: No feathering (hard edge)
- `20`: Smooth transition over 20 pixels (default)
- Higher values: Wider feathering range

## Inputs

- **`pixels`** (IMAGE): The input image to resize
- **`mask_optional`** (MASK, optional): An optional mask that will be resized alongside the image

## Outputs

- **`IMAGE`**: The resized (and optionally cropped/padded) image
- **`MASK`**: The resized mask (or generated mask if input was None)

## Usage Examples

### Example 1: Upscale to 1024px minimum dimension
```
action: resize only
smaller_side: 1024
resize_mode: any
side_ratio: 4:3 (ignored in resize only mode)
```

### Example 2: Create square images by cropping
```
action: crop to ratio
smaller_side: 512
side_ratio: 1:1
crop_pad_position: 0.5 (center crop)
resize_mode: any
```

### Example 3: Create 16:9 images by padding
```
action: pad to ratio
larger_side: 1024
side_ratio: 16:9
crop_pad_position: 0.5 (center padding)
pad_feathering: 30 (smooth edge transition)
resize_mode: any
```

### Example 4: Scale to exact size
```
action: resize only
smaller_side: 512
scale_factor: 0 (disabled)
resize_mode: any
side_ratio: 512:768 (stored for reference)
```
Then set `smaller_side=512` and `side_ratio=512:768` together for precise dimensions.

## Technical Details

- **Interpolation**: Uses bicubic interpolation with anti-aliasing for high-quality resizing
- **GPU Acceleration**: Fully compatible with CUDA and CPU fallback
- **Data Format**: Works with PyTorch tensors and maintains float32 precision
- **Mask Handling**: Masks are resized independently and clamped to [0.0, 1.0] range
- **Validation**: Comprehensive input validation with clear error messages

## License

MIT License - See LICENSE file for details

## Credits

Created by Wladimir Palant

## Changelog

### v1.0.0
- Initial release
- Support for three resize actions (resize, crop, pad)
- Flexible sizing options (smaller_side, larger_side, scale_factor)
- Three resize modes (reduce only, increase only, any)
- Mask support with feathering
- Comprehensive parameter validation

## Contributing

Contributions are welcome! Please feel free to submit issues and pull requests.

## Support

For issues, questions, or suggestions, please open an issue on the GitHub repository.


## 背景

tinygltf官方使用stb做图片的加载，但是stb支持的图片有限，截止2023年2月，只支持JPG, PNG, TGA, BMP, PSD, GIF, HDR, PIC图片的读取。在3dtiles的应用场景中，可能会接触其他格式，以达到极致的压缩，例如webp、ktx2等。

## TinyGltf's Custom Image decoder callback

### 函数申明

TinyGltf官方提供了自定义的图片解析回调函数。

```cpp
///
/// LoadImageDataFunction type. Signature for custom image loading callbacks.
///
///在ParseImage函数中调用
typedef bool (*LoadImageDataFunction)(Image *, const int, std::string *,
                                      std::string *, int, int,
                                      const unsigned char *, int,
                                      void *user_pointer);

///
/// WriteImageDataFunction type. Signature for custom image writing callbacks.
/// The out_uri parameter becomes the URI written to the gltf and may reference
/// a file or contain a data URI.
///
typedef bool (*WriteImageDataFunction)(const std::string *basepath,
                                       const std::string *filename,
                                       const Image *image, bool embedImages,
                                       std::string *out_uri, void *user_pointer);
```

### 如何使用？

在TinyGLTF（glTF解析器上下文）类中，可以设置或移除 **图片读写** 的回调函数

```cpp
///
/// glTF Parser/Serializer context.
///
class TinyGLTF
{
public:
  ///
  /// Set callback to use for loading image data
  ///
  void SetImageLoader(LoadImageDataFunction LoadImageData, void *user_data);

  ///
  /// Set callback to use for writing image data
  ///
  void SetImageWriter(WriteImageDataFunction WriteImageData, void *user_data);
};
```

### 默认的读写器
如果引入了stb_image库，在TinyGLTF（glTF解析器上下文）类中，就会提供默认值。此默认的读写器，正是基于stb_image提供的。

```cpp
class TinyGltf
{
private:
  LoadImageDataFunction LoadImageData =
#ifndef TINYGLTF_NO_STB_IMAGE
      &tinygltf::LoadImageData;
#else
      nullptr;
#endif
  void *load_image_user_data_{nullptr};
  bool user_image_loader_{false};

  WriteImageDataFunction WriteImageData =
#ifndef TINYGLTF_NO_STB_IMAGE_WRITE
      &tinygltf::WriteImageData;
#else
      nullptr;
#endif
  void *write_image_user_data_{nullptr};
}
```

#### LoadImageData

```cpp
#ifndef TINYGLTF_NO_STB_IMAGE
bool LoadImageData(Image *image, const int image_idx, 
				   std::string *err, std::string *warn, 
				   int req_width, int req_height,
                   const unsigned char *bytes, int size, 
                   void *user_data) {
  (void)warn;

  LoadImageDataOption option;
  if (user_data) {
    option = *reinterpret_cast<LoadImageDataOption *>(user_data);
  }

  int w = 0, h = 0, comp = 0, req_comp = 0;

  unsigned char *data = nullptr;

  // preserve_channels true: Use channels stored in the image file.
  // false: force 32-bit textures for common Vulkan compatibility. It appears
  // that some GPU drivers do not support 24-bit images for Vulkan
  req_comp = option.preserve_channels ? 0 : 4;
  int bits = 8;
  int pixel_type = TINYGLTF_COMPONENT_TYPE_UNSIGNED_BYTE;

  // It is possible that the image we want to load is a 16bit per channel image
  // We are going to attempt to load it as 16bit per channel, and if it worked,
  // set the image data accordingly. We are casting the returned pointer into
  // unsigned char, because we are representing "bytes". But we are updating
  // the Image metadata to signal that this image uses 2 bytes (16bits) per
  // channel:
  if (stbi_is_16_bit_from_memory(bytes, size)) {
    data = reinterpret_cast<unsigned char *>(
        stbi_load_16_from_memory(bytes, size, &w, &h, &comp, req_comp));
    if (data) {
      bits = 16;
      pixel_type = TINYGLTF_COMPONENT_TYPE_UNSIGNED_SHORT;
    }
  }

  // at this point, if data is still NULL, it means that the image wasn't
  // 16bit per channel, we are going to load it as a normal 8bit per channel
  // image as we used to do:
  // if image cannot be decoded, ignore parsing and keep it by its path
  // don't break in this case
  // FIXME we should only enter this function if the image is embedded. If
  // image->uri references
  // an image file, it should be left as it is. Image loading should not be
  // mandatory (to support other formats)
  if (!data) data = stbi_load_from_memory(bytes, size, &w, &h, &comp, req_comp);
  if (!data) {
    // NOTE: you can use `warn` instead of `err`
    if (err) {
      (*err) +=
          "Unknown image format. STB cannot decode image data for image[" +
          std::to_string(image_idx) + "] name = \"" + image->name + "\".\n";
    }
    return false;
  }

  if ((w < 1) || (h < 1)) {
    stbi_image_free(data);
    if (err) {
      (*err) += "Invalid image data for image[" + std::to_string(image_idx) +
                "] name = \"" + image->name + "\"\n";
    }
    return false;
  }

  if (req_width > 0) {
    if (req_width != w) {
      stbi_image_free(data);
      if (err) {
        (*err) += "Image width mismatch for image[" +
                  std::to_string(image_idx) + "] name = \"" + image->name +
                  "\"\n";
      }
      return false;
    }
  }

  if (req_height > 0) {
    if (req_height != h) {
      stbi_image_free(data);
      if (err) {
        (*err) += "Image height mismatch. for image[" +
                  std::to_string(image_idx) + "] name = \"" + image->name +
                  "\"\n";
      }
      return false;
    }
  }

  if (req_comp != 0) {
    // loaded data has `req_comp` channels(components)
    comp = req_comp;
  }

  image->width = w;
  image->height = h;
  image->component = comp;
  image->bits = bits;
  image->pixel_type = pixel_type;
  image->image.resize(static_cast<size_t>(w * h * comp) * size_t(bits / 8));
  std::copy(data, data + w * h * comp * (bits / 8), image->image.begin());
  stbi_image_free(data);

  return true;
}
#endif
```

#### WriteImageData
```cpp
#ifndef TINYGLTF_NO_STB_IMAGE_WRITE
static void WriteToMemory_stbi(void *context, void *data, int size) {
  std::vector<unsigned char> *buffer =
      reinterpret_cast<std::vector<unsigned char> *>(context);

  unsigned char *pData = reinterpret_cast<unsigned char *>(data);

  buffer->insert(buffer->end(), pData, pData + size);
}

bool WriteImageData(const std::string *basepath, const std::string *filename,
                    const Image *image, bool embedImages, std::string *out_uri,
                    void *fsPtr) {
  const std::string ext = GetFilePathExtension(*filename);

  // Write image to temporary buffer
  std::string header;
  std::vector<unsigned char> data;

  if (ext == "png") {
    if ((image->bits != 8) ||
        (image->pixel_type != TINYGLTF_COMPONENT_TYPE_UNSIGNED_BYTE)) {
      // Unsupported pixel format
      return false;
    }

    if (!stbi_write_png_to_func(WriteToMemory_stbi, &data, image->width,
                                image->height, image->component,
                                &image->image[0], 0)) {
      return false;
    }
    header = "data:image/png;base64,";
  } else if (ext == "jpg") {
    if (!stbi_write_jpg_to_func(WriteToMemory_stbi, &data, image->width,
                                image->height, image->component,
                                &image->image[0], 100)) {
      return false;
    }
    header = "data:image/jpeg;base64,";
  } else if (ext == "bmp") {
    if (!stbi_write_bmp_to_func(WriteToMemory_stbi, &data, image->width,
                                image->height, image->component,
                                &image->image[0])) {
      return false;
    }
    header = "data:image/bmp;base64,";
  } else if (!embedImages) {
    // Error: can't output requested format to file
    return false;
  }

  if (embedImages) {
    // Embed base64-encoded image into URI
    if (data.size()) {
      *out_uri = header +
                base64_encode(&data[0], static_cast<unsigned int>(data.size()));
    } else {
      // Throw error?
    }
  } else {
    // Write image to disc
    FsCallbacks *fs = reinterpret_cast<FsCallbacks *>(fsPtr);
    if ((fs != nullptr) && (fs->WriteWholeFile != nullptr)) {
      const std::string imagefilepath = JoinPath(*basepath, *filename);
      std::string writeError;
      if (!fs->WriteWholeFile(&writeError, imagefilepath, data,
                              fs->user_data)) {
        // Could not write image file to disc; Throw error ?
        return false;
      }
    } else {
      // Throw error?
    }
    *out_uri = *filename;
  }

  return true;
}
#endif
```

### LoadImageDataFunction中的user_data

在默认的读取器（`LoadImageData`）中，`user_data`即为`LoadImageDataOption`。具体逻辑可在`LoadFromString`中查看

- 其中，`load_image_user_data_`在`TinyGLTF::SetImageLoader`函数中设置。

```cpp
bool TinyGLTF::LoadFromString(Model *model, std::string *err, std::string *warn,
                              const char *json_str,
                              unsigned int json_str_length,
                              const std::string &base_dir,
                              unsigned int check_sections) 
{
  //...
  
  // 11. Parse Image
  void *load_image_user_data{nullptr}; //加载Image时的用户数据

  LoadImageDataOption load_image_option;

  if (user_image_loader_) {
	// Use user supplied pointer
	//如果是用户自定义图片加载器，使用用户提供的指针
	load_image_user_data = load_image_user_data_;
  } else {
    //如果是默认的加载器，则传入LoadImageDataOption
	load_image_option.preserve_channels = preserve_image_channels_;
	load_image_user_data = reinterpret_cast<void *>(&load_image_option);
  }

  //...
}
```

`user_data`不一定要设置，因为在默认加载器中，也会提供默认的`LoadImageDataOption`。

```cpp
bool LoadImageData(Image *image, const int image_idx, std::string *err,
                   std::string *warn, int req_width, int req_height,
                   const unsigned char *bytes, int size, void *user_data) {
  (void)warn;

  LoadImageDataOption option;
  if (user_data) {
    option = *reinterpret_cast<LoadImageDataOption *>(user_data);
  }

  //...
}
```

## 示例：集成webp


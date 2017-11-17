title: java代码判断base64字符串形式的图片格式、大小、高宽
author: 赵旭东
tags:
  - base64
categories:
  - 技术
date: 2017-11-17 17:20:00
---
有些插件将图片转换为base64字符串时，会带有`data:image/png;base64,`类似的前缀，而有些又没有这样的前缀。为了统一处理，可以先去掉前缀，然后做后续判断。

## 1.去掉base64前缀
```
 /**
     * 去掉base64图片前缀
     *
     * @param imgString
     * @return
     */
    private String replacePre(String imgString) {
        //允许的图片格式（可配置）
        String imgType = "jpg,png,jpeg";
        if (!StringUtils.isEmpty(imgType)) {
            String[] imgTypes = imgType.split(",");
            Pattern pattern;
            Matcher matcher;
            String regex;
            for (String v : imgTypes) {
                regex = MessageFormat.format("data:image/{0};base64,", v);
                pattern = Pattern.compile(regex, Pattern.CASE_INSENSITIVE);
                matcher = pattern.matcher(imgString);
                if (matcher.lookingAt()) {
                    return matcher.replaceFirst("");
                }
            }
        }
        return imgString;
    }
```
<!--more-->

## 2.校验图片格式
> 代码里面向外抛出的异常可以替换成自己想要的任何操作

```
    /**
     * 校验格式是否符合要求
     *
     * @param imgString
     */
    private boolean checkFormat(String imgString) {
        //允许的图片格式（可配置）
        String imgType = "jpg,png,jpeg";
        try {
            imgString = replacePre(imgString);
            byte[] bytes = new BASE64Decoder().decodeBuffer(imgString);
            //判断文件大小是否符合要求
            if (!checkSize(bytes)) {
                throw new BizException(ErrorMessage.ERROR_FILE_FORMAT.getCode(), "文件大小不符合要求");
            }
          
            //不带类似data:image/jpg;base64,前缀的解析
            ImageInputStream imageInputstream = new MemoryCacheImageInputStream(new ByteArrayInputStream(
                    bytes));
            //不使用磁盘缓存
            ImageIO.setUseCache(false);
            Iterator<ImageReader> it = ImageIO.getImageReaders(imageInputstream);
            if (it.hasNext()) {
                ImageReader imageReader = it.next();
                // 设置解码器的输入流
                imageReader.setInput(imageInputstream, true, true);
                
                // 图像文件格式后缀
                String suffix = imageReader.getFormatName().trim().toLowerCase();
                  int height = imageReader.getHeight(0);
                int width = imageReader.getWidth(0);
                imageInputstream.close();
                //校验宽和高是否符合要求
                if (!checkWidthAndHeight(width, height)) {
                    throw new BizException(ErrorMessage.ERROR_FILE_FORMAT.getCode(), "文件宽高不符合要求");
                }
                String[] imgTypes = imgType.split(",");
                for (String type : imgTypes) {
                    if (type.equalsIgnoreCase(suffix)) {
                        return true;
                    }
                }
            }

        } catch (IOException e) {
            e.printStackTrace();
            throw new BizException(ErrorMessage.SYSTEM_ERROR.getCode(), ErrorMessage.SYSTEM_ERROR.getMessage());
        }

        return false;
    }
```
## 3. 校验图片大小

```
    /**
     * 校验文件大小
     *
     * @param
     * @return
     */
    private boolean checkSize(byte[] bytes) {
        //符合条件的照片大小（可配置） 单位：M
        double imgSize = 2.0;
        //图片转base64字符串一般会大，这个变量就是设置偏移量。可配置在文件中，随时修改。目前配的是0。后续看情况适当做修改
        double deviation = 0.0;
        int length = bytes.length;
        //原照片大小
        double size = (double) length / 1024 / 1024 * (1 - deviation);
        logger.info("照片大小为：" + size + "M");
        return size <= imgSize;
    }
```

## 4.校验图片宽高

```
    /**
     * 校验宽高
     *
     * @param 
     * @return
     */
    private boolean checkWidthAndHeight(int width, int height) {
        //宽高最小值（可配置） 单位px
        int imgRangeLow = 8;
        //宽高最大值（可配置） 单位px
        int imgRangeHigh = 40;
        logger.info("照片宽为：" + width + "px");
        logger.info("照片高为：" + height + "px");

        if (width > imgRangeLow && width <= imgRangeHigh &&
                height > imgRangeLow && height <= imgRangeHigh) {
            return true;
        }

        return false;
    }
```
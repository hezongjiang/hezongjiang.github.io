---
title: SpringBoot 上传图片
catalog: true
date: 2019-04-23 19:47:05
subtitle:
header-img: cat.jpeg
tags: 服务端开发
---
**思路**

一般情况下都是将用户上传的图片放到服务器的某个文件夹中，然后将图片在服务器中的路径存入数据库。本Demo也是这样做的。

由于用户自己保存的图片文件名可能跟其他用户同名造成冲突，因此本Demo选择了使用时间戳来生成随机的文件名解决冲突。

本Demo不涉及任何有关数据库的操作。

创建idea默认的springboot项目，版本是2.X

**第一步：**

新建一个`FileController`:
```
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpServletRequest;
import java.io.File;
import java.io.IOException;

@RestController
public class FileController {

    @PostMapping(value = "/fileUpload")
    public ResultVo fileUpload(@RequestParam(value = "file") MultipartFile file, Model model, HttpServletRequest request) {
        if (file.isEmpty()) {
            System.out.println("文件为空空");
            return ResultVo.success("文件为空空");
        }
        String fileName = file.getOriginalFilename();  // 文件名
        String suffixName = fileName.substring(fileName.lastIndexOf("."));  // 后缀名
        String filePath = "/Users/terry-jri/Desktop/temp/"; // 上传后的路径
        fileName = System.currentTimeMillis() + suffixName; // 新文件名
        File dest = new File(filePath + fileName);
        if (!dest.getParentFile().exists()) {
            dest.getParentFile().mkdirs();
        }
        try {
            file.transferTo(dest);
        } catch (IOException e) {
            e.printStackTrace();
        }
        String filename = "/images/" + fileName;
        model.addAttribute("filename", filename);

        return ResultVo.success(filename);
    }
}
```

**第二步：**
资源映射路径
```
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MyWebAppConfigurer implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
       /**
		 * 资源映射路径
		 * addResourceHandler：访问映射路径
		 * addResourceLocations：资源绝对路径
		 */
        registry.addResourceHandler("/images/**").addResourceLocations("file:/Users/terry-jri/Desktop/temp/");
    }
}
```
现在已经可以上传了，除了`ResultVo `是自定义的一个返回对象的包装类，其他都是`Spring`的类。访问图片使用 `http://localhost:8080/images/时间戳.png`
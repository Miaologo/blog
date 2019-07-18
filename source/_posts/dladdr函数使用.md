---
title: dladdr函数使用
date: 2019-07-18 17:30:16
tags: dladdr, 动态库
---

## 1.获取APP全部自定义类名

最近需要有个需求，涉及到runtime打印所有的自定义类名，那么如何区分自己的类和系统定义的类呢？查了些资料发现可用dladdr来实现，在寒神的XXShield里面也有类似使用；dladdr可获得一个函数所在模块，名称以及地址。
 引入头文件  #import <dlfcn.h>

获取自定义类名：

```objective-c
    int numClasses;
    Class * classes = NULL;
    classes = NULL;
    numClasses = objc_getClassList(NULL, 0);
    if (numClasses > 0 )
    {
        static struct dl_info app_info;
        if (app_info.dli_saddr == NULL) {
            dladdr((__bridge void *)[UIApplication.sharedApplication.delegate class], &app_info);
        }
        classes = (__unsafe_unretained Class *)malloc(sizeof(Class) * numClasses);
        numClasses = objc_getClassList(classes, numClasses);
        for (int i = 0; i < numClasses; i++) {
            Class c = classes[i];
            
            struct dl_info self_info = {0};
            dladdr((__bridge void *)c, &self_info);
            
            // 忽略系统函数
            if (self_info.dli_fname == NULL || strcmp(app_info.dli_fname, self_info.dli_fname)) {
            }else{
            //自定义函数
            NSLog(@"%s", class_getName(c));
            }
        }
        free(classes);
    }
```

当dladdr((__bridge void *)[self class], &self_info)中的self为我们自定义的一个UIViewController的时候。
 self_info.dli_fname打印出的为模块路径：

```
/Users/ganvinalix/Library/Developer/CoreSimulator/Devices/13BD3F3B-2C8C-40BB-8CC1-96C71FD0CBBF/data/Containers/Bundle/Application/DF26258E-2F6F-418F-80C3-751D03FD1F21/XXShield_Example.app/XXShield_Example
```

当dladdr((__bridge void *)[NSObject class], &self_info);NSObject是系统SDK函数。
 self_info.dli_fname打印出的为模块路径：

```
"/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/CoreSimulator/Profiles/Runtimes/iOS.simruntime/Contents/Resources/RuntimeRoot/usr/lib/libobjc.A.dylib"
```

app_info.dli_fname打印出的为模块路径：

```
"/Users/ganvinalix/Library/Developer/CoreSimulator/Devices/13BD3F3B-2C8C-40BB-8CC1-96C71FD0CBBF/data/Containers/Bundle/Application/3D34D2D9-4556-4E0B-93F4-08034E9975A4/XXShield_Example.app/XXShield_Example"
```

总之就是要比较的类的dli_fname，如果和app的sharedApplication.delegate类的dli_fname相同就为自定义类，否则为系统SDK类

当然dladdr还可以做一些安全验证方面的事情。推荐庆哥早期文章，iOS安全–验证函数地址，检测是否被替换，反注；
 http://www.blogfshare.com/ioss-validate-address.html

## 2.打印APP加载的所有动态链接库的名称与大小等相关信息

```objective-c
#include <mach-o/getsect.h>
#include <mach-o/loader.h>
#include <mach-o/dyld.h>
#include <dlfcn.h>
#import <objc/runtime.h>
#import <objc/message.h>
#include <mach-o/ldsyms.h>

NSArray<NSString *>* KGReadConfiguration(char *sectionName,const struct mach_header *mhp);

static uint32_t _image_header_size(const struct mach_header *mh)
{
    bool is_header_64_bit = (mh->magic == MH_MAGIC_64 || mh->magic == MH_CIGAM_64);
    return (is_header_64_bit ? sizeof(struct mach_header_64) : sizeof(struct mach_header));
}

static void _image_visit_load_commands(const struct mach_header *mh, void (^visitor)(struct load_command *lc, bool *stop))
{
    assert(visitor != NULL);
    
    uintptr_t lc_cursor = (uintptr_t)mh + _image_header_size(mh);
    
    for (uint32_t idx = 0; idx < mh->ncmds; idx++) {
        struct load_command *lc = (struct load_command *)lc_cursor;
        
        bool stop = false;
        visitor(lc, &stop);
        
        if (stop) {
            return;
        }
        
        lc_cursor += lc->cmdsize;
    }
}

static uint64_t _image_text_segment_size(const struct mach_header *mh)
{
    static const char *text_segment_name = "__TEXT";
    
    __block uint64_t text_size = 0;
    
    _image_visit_load_commands(mh, ^ (struct load_command *lc, bool *stop) {
        if (lc->cmdsize == 0) {
            return;
        }
        if (lc->cmd == LC_SEGMENT) {
            struct segment_command *seg_cmd = (struct segment_command *)lc;
            if (strcmp(seg_cmd->segname, text_segment_name) == 0) {
                text_size = seg_cmd->vmsize;
                *stop = true;
                return;
            }
        }
        if (lc->cmd == LC_SEGMENT_64) {
            struct segment_command_64 *seg_cmd = (struct segment_command_64 *)lc;
            if (strcmp(seg_cmd->segname, text_segment_name) == 0) {
                text_size = seg_cmd->vmsize;
                *stop = true;
                return;
            }
        }
    });
    
    return text_size;
}

static const uuid_t *_image_retrieve_uuid(const struct mach_header *mh)
{
    __block const struct uuid_command *uuid_cmd = NULL;
    
    _image_visit_load_commands(mh, ^ (struct load_command *lc, bool *stop) {
        if (lc->cmdsize == 0) {
            return;
        }
        if (lc->cmd == LC_UUID) {
            uuid_cmd = (const struct uuid_command *)lc;
            *stop = true;
        }
    });
    
    if (uuid_cmd == NULL) {
        return NULL;
    }
    
    return &uuid_cmd->uuid;
}

static void _print_image(const struct mach_header *mh, bool added)
{
    Dl_info image_info;
    int result = dladdr(mh, &image_info);
    
    if (result == 0) {
        printf("Could not print info for mach_header: %p\n\n", mh);
        return;
    }
    
    const char *image_name = image_info.dli_fname;
    
    const intptr_t image_base_address = (intptr_t)image_info.dli_fbase;
    const uint64_t image_text_size = _image_text_segment_size(mh);
    
    char image_uuid[37];
    const uuid_t *image_uuid_bytes = _image_retrieve_uuid(mh);
    uuid_unparse(*image_uuid_bytes, image_uuid);
    
    const char *log = added ? "Added" : "Removed";
    printf("%s: 0x%02lx (0x%02llx) %s <%s>\n\n", log, image_base_address, image_text_size, image_name, image_uuid);
}

static void dyld_callback(const struct mach_header *mhp, intptr_t vmaddr_slide)
{
    _print_image(mhp, true);
}

//注册main之前的析构函数,析构函数仅爱周注解才能生效
__attribute__((constructor))
void initProphet() {
    //动态链接库加载的时候的hook，可能会回调次数比较多，可能不建议
    _dyld_register_func_for_add_image(dyld_callback);
}
```


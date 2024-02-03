# [Martinspace.top](https://martinspace.top)
使用**hugo&papaermod**框架开发的个人博客
# 目录介绍
```
|
|--archetypes
|--assets
|--content 使用markdown写博客
|--data 空
|--layouts 自定义的页面配置在这里
|--public 打包的文件在这里
|--resources 空
|--static 存储一些图片等静态资源，
|--themes 官方的默认配置在这里
|--config.yaml 在这个文件中配置项目基本信息
```
```shift+.```打开github在线编辑器 
or
```git clone```

# 启动项目
输入```hugo server -D```

按住```ctrl```键点击```localhost:1313```就可以在浏览器中预览项目了

# 打包项目
```hugo -F --cleanDestinationDir```，这样打包会把之前的public清空，生成新的public，防止产生冲突

将public文件夹挂在到服务器就可以正常访问了

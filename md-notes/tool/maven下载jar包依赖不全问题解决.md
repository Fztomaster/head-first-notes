# maven下载jar包依赖不全问题解决

### 1.删除本地maven仓库的不全的依赖包

### 2.找一份完整的依赖包拷贝到maven仓库里

### 3.修改maven的settings.xml

之前有配置其他镜像的，可以删掉，直接从中央仓库下载

### 4.重新把依赖添加到pom.xml文件中
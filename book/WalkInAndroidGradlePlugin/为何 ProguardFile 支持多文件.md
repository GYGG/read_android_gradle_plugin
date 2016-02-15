# 为何 ProguardFile 支持多文件

当开始学习如何在 Gradle 中使用代码混淆——也就是原来 Ant 构建中已经用的很熟的 Proguard 工具时，一般会先学会在 Android DSL 中增加下面这段配置来使 Proguard 生效：

``` 
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
```

这里比较玄妙的是 proguardFiles 这个配置项后面可以通过逗号分隔来连续配置多个 proguard 的规则声明文件，并且让他们同时生效。

一起来探究一下这个问题。

## 找到位置

在源码中「/gradle/LibraryPlugin.groovy」能够看到：

``` 
// merge consumer proguard files from different build types and flavors
MergeFileTask mergeProGuardFileTask = project.tasks.create(
        "merge${variantData.variantConfiguration.fullName.capitalize()}ProguardFiles",
        MergeFileTask)
mergeProGuardFileTask.conventionMapping.inputFiles = {
    project.files(variantConfig.getConsumerProguardFiles()).files }
mergeProGuardFileTask.conventionMapping.outputFile = {
    project.file(
            "$project.buildDir/$DIR_BUNDLES/${variantData.variantConfiguration.dirName}/$LibraryBundle.FN_PROGUARD_TXT")
}
```

这段就是进行多个 ProguardFile 合并的代码。

再找到其中用到的 MergeFileTask类，对应路径在：「/gradle/internal/tasks/MergeFileTask.groovy」

从这个类的下面这段核心实现可以看出，其实最上面提到的多个逗号分隔开的 Proguard 规则声明就是进行简单的文件连接最终合并成一个文件的。

``` 
// otherwise put the all the files together
for (File file : files) {
    String content = Files.toString(file, Charsets.UTF_8);
    Files.append(content, output, Charsets.UTF_8);
    Files.append("\n", output, Charsets.UTF_8);
}
```

这么看就没什么技术含量了。


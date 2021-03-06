| Markdown版本笔记 | 我的GitHub首页 | 我的博客 | 我的微信 | 我的邮箱 |
| :------------: | :------------: | :------------: | :------------: | :------------: |
| [MyAndroidBlogs][Markdown] | [baiqiantao][GitHub] | [baiqiantao][博客] | bqt20094 | baiqiantao@sina.com |

[Markdown]:https://github.com/baiqiantao/MyAndroidBlogs
[GitHub]:https://github.com/baiqiantao
[博客]:http://www.cnblogs.com/baiqiantao/

文件 File 递归复制 修改后缀名 生成Markdown目录 字符串解析 MD
***

# Markdown 工具类
工具类的作用：生成`GitHub`上厂库[MyAndroidBlogs](https://github.com/baiqiantao/MyAndroidBlogs)的`REANME.MD`中博客的目录。

步骤：
- 通过为知笔记导出指定目录下的所有文件，导出格式为纯文本
- 通过 `modifyFileSuffix` 修改后缀名
- 通过 `getFormatFilesNames` 获取目录
- 将获取的内容粘贴到`REANME.MD`中就可以了

```java
modifyFileSuffix(new File(DIR));
getFormatFilesNames(new File(DIR));
```

## 一些常量
```java
private static final String DIR = "D:/为知笔记导出/MyAndroidBlogs";
private static final String WEBSITE = "https://github.com/baiqiantao/MyAndroidBlogs/blob/master/";
private static final String FILTER = "MD";
private static final String SUFFIX_MD_TXT = ".md.txt";
private static final String SUFFIX_TXT = ".txt";
private static final String SUFFIX_MD = ".md";
private static final String SPACE = " ";
private static final String SPACE_FORMAT = "%20";
private static final String[] EXCLUDE_FILES = new String[]{".git", "README.md"};
```

## 递归修改文件后缀名
```java
/**
 * 递归修改文件后缀名
 */
public static void modifyFileSuffix(File from) {
    if (from == null || !from.exists()) {
        throw new RuntimeException("源目录或文件不存在");
    }
    if (from.isFile()) { //只处理带指定标识的文件
        if (from.getName().contains(FILTER)) {
            copyFile(from, new File(from.getParent(), getNewName(from)));//首先复制文件
        }
        from.delete();//然后删除源文件
        return;
    }

    File[] listFiles = from.listFiles();
    if (listFiles == null || listFiles.length == 0) {
        System.out.println("----------------------" + from.getName() + "中不存在任何文件");
        from.delete();//删除空文件夹
        return;
    }

    for (File file : listFiles) {
        if (file.isDirectory()) {
            modifyFileSuffix(file);//递归
        } else if (file.isFile()) {
            if (file.getName().contains(FILTER)) {//只处理带指定标识的文件
                copyFile(file, new File(file.getParent(), getNewName(file)));
            }
            file.delete();
        }
    }

    listFiles = from.listFiles();
    if (listFiles == null || listFiles.length == 0) {
        System.out.println("----------------------" + from.getName() + "的文件全部不符合要求");
        from.delete();//删除空文件夹
    }
}
```

```java
/**
 * 获取重命名的文件名
 */
private static String getNewName(File file) {
    String name = file.getName();
    if (name.endsWith(SUFFIX_MD_TXT)) { //处理指定后缀的文件
        name = name.substring(0, name.indexOf(SUFFIX_MD_TXT) + SUFFIX_MD.length());
    } else if (name.endsWith(SUFFIX_TXT)) {
        name = name.substring(0, name.indexOf(SUFFIX_TXT)) + SUFFIX_MD;
    }
    return name;
}
```

## 递归获取格式化后的文件名
```java
/**
 * 获取指定目录及其子目录下的文件的文件名，并对文件名进行格式化，并存储到一个指定的集合中
 */
public static List<String> getFormatFilesNames(File from) {
    List<String> filePathList = new ArrayList<>();
    getDirFormatFilesNames(filePathList, from, 0);
    for (String string : filePathList) {
        System.out.print(string);
    }
    return filePathList;
}
```

```java
/**
 * 递归获取指定目录及其子目录下的文件的文件名，并对文件名进行格式化，并存储到一个指定的集合中
 *
 * @param filePathList 将结果保存到指定的集合中
 * @param from         要遍历的目录
 * @param curLeval     记录当前递归所在层级
 */
private static void getDirFormatFilesNames(List<String> filePathList, File from, int curLeval) {
    if (from == null || !from.exists()) {
        throw new RuntimeException("源目录或文件不存在");
    } else if (isExcludeFile(from.getName())) {
        System.out.println("----------------------" + "忽略文件" + from.getName());
        return;
    } else {
        filePathList = filePathList == null ? new ArrayList<String>() : filePathList;
        filePathList.add(getTitle(curLeval, from));
    }
    
    curLeval++;
    File[] files = from.listFiles();
    if (files == null || files.length == 0) {
        System.out.println("----------------------" + from.getName() + "中不存在任何文件");
        return;
    }
    
    for (File file : files) {
        if (file.isDirectory()) {
            getDirFormatFilesNames(filePathList, file, curLeval);//递归
        } else if (file.isFile() && !isExcludeFile(file.getName())) {
            filePathList.add(getTitle(curLeval, file));
        }
    }
}
```

```java
/**
 * 是否是忽略的文件
 */
private static boolean isExcludeFile(String fileName) {
    for (String name : EXCLUDE_FILES) {
        if (name.equals(fileName)) {
            return true;
        }
    }
    return false;
}
```

## 递归复制文件
```java
/**
 * 递归复制目录下的所有文件
 */
public static void copyFiles(File from, File to) {
    if (from == null || !from.exists() || to == null) {
        throw new RuntimeException("源文件或目标文件不存在");
    } else if (from.equals(to)) {
        throw new RuntimeException("源文件和目标文件是同一个");
    }
    
    if (from.isDirectory()) {
        if (to.isFile()) {
            throw new RuntimeException("目录不能复制为文件");
        }
        if (!to.exists()) {
            to.mkdirs();//创建目录
        }
    } else {
        if (to.isDirectory()) {
            throw new RuntimeException("文件不能复制为目录");
        }
        copyFile(from, to);//文件的话直接复制
        return;
    }
    
    File[] files = from.listFiles();
    if (files == null || files.length == 0) {
        System.out.println(from.getName() + "中不存在任何文件");
        return;
    }
    
    for (File file : files) {
        if (file.isDirectory()) {
            File copyDir = new File(to, file.getName());
            if (!copyDir.exists()) {
                copyDir.mkdirs();
                System.out.println("创建子目录\t\t" + copyDir.getAbsolutePath());
            }
            copyFiles(file, copyDir);//递归
        } else if (file.getName().contains(FILTER)) {
            copyFile(file, new File(to, file.getName()));
        }
    }
}
```

## 复制一个文件目录
```java
/**
 * 复制一个文件目录(不会复制目录下的文件)
 */
public static void copyFile(File from, File to) {
    if (from == null || !from.exists() || to == null) {
        throw new RuntimeException("源文件或目标文件不存在");
    } else if (from.equals(to)) {
        throw new RuntimeException("源文件和目标文件是同一个");
    }
    
    if (from.isDirectory()) {
        if (to.isFile()) {
            throw new RuntimeException("目录不能复制为文件");
        }
        if (!to.exists()) {
            to.mkdirs();//创建目录
        }
        return; //没有下面的刘操作
    } else {
        if (to.isDirectory()) {
            throw new RuntimeException("文件不能复制为目录");
        }
        if (!to.getParentFile().exists()) {
            to.getParentFile().mkdirs(); //复制父目录
        }
    }
    
    try {
        BufferedInputStream bufis = new BufferedInputStream(new FileInputStream(from));
        BufferedOutputStream bufos = new BufferedOutputStream(new FileOutputStream(to));
        int ch;
        while ((ch = bufis.read()) != -1) {
            bufos.write(ch);
        }
        bufis.close();
        bufos.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

## 生成格式化的 Markdown 目录
```java
/***
 * 生成格式化的 Markdown 目录
 */
private static String getTitle(int level, File file) {
    StringBuilder sb = new StringBuilder();
    StringBuilder parentPath = new StringBuilder();
    File parent;
    for (int x = 1; x < level; x++) {
        sb.append("\t");
        parent = file.getParentFile();//逐级获取父文件
        parentPath.insert(0, parent.getName() + "/");//在前面插入父文件的文件名
    }
    
    sb.append("-").append(" ")//无序列表
        .append("[")//超链接显示的字符
        .append(file.getName().endsWith(SUFFIX_MD) ? file.getName().substring(0, file.getName().lastIndexOf(SUFFIX_MD)) : file.getName())//
        .append("]")//
        .append("(")//拼接超链接
        .append(WEBSITE)//前缀
        .append(parentPath.toString())//父目录
        .append(file.getName().replaceAll(SPACE, SPACE_FORMAT))//文件名
        .append(")")//
        .append("\n");
    return sb.toString();
}
```
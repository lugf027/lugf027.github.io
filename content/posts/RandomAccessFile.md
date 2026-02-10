---
title: " SweetPotato内存优化-RandomAccessFile 读取速度比较记录 "
date: 2022-08-08
draft: false
tags: ["日志工具", "内存优化"]
categories: ["技术"]
---

> Java IO 包提供有RandomAccessFile，但是经过测试读取性能较差。记录一下。

RandomAccessFile 读取速度比较
### 零、背景

最近遇到日志过滤工具内存性能优化问题。	

日志过滤工具会将日志文件读到内存，提供搜索、过滤等功能。但是目前内存占用较高，亟待优化。	

初步方案是不在内存中记录所有的日志信息，改为记录缓存+位置。缓存中没有时读文件，定期清理缓存。

### 一、数据结构

主干日志最小单元结构：
![图片](/images/1/7fbe9300-86dd-45d0-88ea-f82a215fb9ab.png)

初步优化后结构：
![图片](/images/1/7ba5d811-bd09-49f5-ab1b-c02a3d060a68.png)

主要删除了时区zoneOffset、整行信息rawLog、日志msg在整行中的索引messageRange；新增了文件名filePath、日志msg在文件中的位置msgPointerPos。

这样在messagepart为空（没有缓存时），直接从文件读取这部分内容。

优化后整体内存下降达85%。

### 二、随机读取文件

Java IO 包提供有RandomAccessFile，但是经过测试读取性能较差。	

以读取110,271,326 字节日志文件测试数据为例：直接BufferedReader读，时间在**550ms**左右；直接用RandomAccessFile读，时间在**70.3s**左右。性能差了两个数量级。	

经确认后者主要开销在read上(接近70s)，原因是每个字节的read都会进行一次io操作。故这里考虑下对RandomAccessFile加buffer，或直接内存映射下看能否提升效率。

经测试：对RandomAccessFile使用内存映射，时间在**6s**左右；**对RandomAccessFile加Buffer读，时间在600ms左右，基本与BufferedReader一致。**

故此处先选用BufferedRandomAccessFile。本人这里直接搬运了[apache cassandra](https://github.com/facebookarchive/cassandra/tree/master/src/org/apache/cassandra) 包里的 [BufferedRandomAccessFile](https://github.com/facebookarchive/cassandra/blob/master/src/org/apache/cassandra/io/BufferedRandomAccessFile.java)。

具体数据：

```shell
* testCurBufferedFile time: 547 
​
testRawRandomAccessFile time seek:296
testRawRandomAccessFile time read:69977
testRawRandomAccessFile time rest:28
* testRawRandomAccessFile time: 70302 
​
testBufferedRandomAccessFile time seek:17
testBufferedRandomAccessFile time read:574
testBufferedRandomAccessFile time rest:24
* testBufferedRandomAccessFile time: 615 
​
testMappedByteBuffer time char:2946
testMappedByteBuffer time rest:3111
* testMappedByteBuffer time: 6057
```
#### 1、 testCurBufferedFile


```kotlin
private fun testCurBufferedFile(path: String): Long {
    val file = File(path)
    val strList = mutableListOf<String>()

    val timeStart = System.currentTimeMillis()
    if (file.exists()) {
        file.bufferedReader().use {
            while (true) {
                val line = it.readLine() ?: break
                strList.add(line)
            }
        }
    }
    val timeEnd = System.currentTimeMillis()

    return timeEnd - timeStart
}
```

#### 2、testRawRandomAccessFile


```kotlin
private fun testRawRandomAccessFile(path: String): Long {
    val file = RandomAccessFile(path, "r")
    val strList = mutableListOf<String>()
    var curStep = 0L
    var seekTime = 0L
    var readLineTime = 0L

    val timeStart = System.currentTimeMillis()
    while (true) {
        val timeSeekStart = System.nanoTime()
        file.seek(curStep)
        val timeSeekEndAndReadLineStart = System.nanoTime()
        val line = file.readLine() ?: break
        val timeReadLineEnd = System.nanoTime()

        seekTime += (timeSeekEndAndReadLineStart - timeSeekStart)
        readLineTime += (timeReadLineEnd - timeSeekEndAndReadLineStart)

        curStep += (line.length + 1)
        strList.add(line)
    }
    val timeEnd = System.currentTimeMillis()

    val totalTime = timeEnd - timeStart
    println("testRawRandomAccessFile time seek:${seekTime / 1000_000}")
    println("testRawRandomAccessFile time read:${readLineTime / 1000_000}")
    println("testRawRandomAccessFile time rest:${totalTime - (seekTime + readLineTime) / 1000_000}")
    return totalTime
}
```

#### 3、testBufferedRandomAccessFile


```kotlin
private fun testBufferedRandomAccessFile(path: String): Long {
    val file = BufferedRandomAccessFile(path, "r")
    val strList = mutableListOf<String>()
    var curStep = 0L
    var seekTime = 0L
    var readLineTime = 0L

    val timeStart = System.currentTimeMillis()
    while (true) {
        val timeSeekStart = System.nanoTime()
        file.seek(curStep)
        val timeSeekEndAndReadLineStart = System.nanoTime()
        val line = file.readLine() ?: break
        val timeReadLineEnd = System.nanoTime()

        seekTime += (timeSeekEndAndReadLineStart - timeSeekStart)
        readLineTime += (timeReadLineEnd - timeSeekEndAndReadLineStart)

        curStep += (line.length + 1)
        strList.add(line)
    }
    val timeEnd = System.currentTimeMillis()

    val totalTime = timeEnd - timeStart
    println("testBufferedRandomAccessFile time seek:${seekTime / 1000_000}")
    println("testBufferedRandomAccessFile time read:${readLineTime / 1000_000}")
    println("testBufferedRandomAccessFile time rest:${totalTime - (seekTime + readLineTime) / 1000_000}")
    return totalTime
}
```

#### 4、testMappedByteBuffer


```kotlin
private fun testMappedByteBuffer(path: String): Long {
    val file = RandomAccessFile(path, "r")
    val fileChannel = file.channel
    val mbbFile = fileChannel.map(FileChannel.MapMode.READ_ONLY, 0, fileChannel.size())
    val strList = mutableListOf<String>()
    var chatToStringTime = 0L

    val timeStart = System.currentTimeMillis()
    val line = StringBuilder("")
    for (i in 0 until fileChannel.size()) {
        val char = mbbFile.get(i.toInt()).toInt().toChar()
        if (char == '\n') {
            strList.add(line.toString())
            line.clear()
        }

        val timeCharToStringStart = System.nanoTime()
        line.append(char)
        val timeCharToStringEnd = System.nanoTime()
        chatToStringTime += (timeCharToStringEnd - timeCharToStringStart)
    }
    val timeEnd = System.currentTimeMillis()

    val totalTime = timeEnd - timeStart
    println("testMappedByteBuffer time char:${chatToStringTime / 1000_000}")
    println("testMappedByteBuffer time rest:${totalTime - chatToStringTime / 1000_000}")
    return totalTime
}
```
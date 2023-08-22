---
title: 文件md5和文本压缩字节流上传
author: 陈龙
date: 2021-12-30 22:21:16
tags: [md5]
categories: [md5]
---

## 文件 md5

在开发文件上传的时候，文件服务器需要提前知道文件的 md5，来判断文件在服务器是否存在，可以通过以下方法来获取文件 md5 的值

```ts
import SparkMD5 from 'spark-md5';

function getMd5(file: File): Promise<string> {
  return new Promise((resovle, reject) => {
    const blobSlice = File.prototype.slice,
      chunkSize = 2097152,
      chunks = Math.ceil(file.size / chunkSize),
      spark = new SparkMD5.ArrayBuffer(),
      fileReader = new FileReader();
    let currentChunk = 0;
    fileReader.onload = function (e) {
      console.log('read chunk nr', currentChunk + 1, 'of', chunks);
      spark.append(e.target.result as ArrayBuffer);
      currentChunk++;
      if (currentChunk < chunks) {
        loadNext();
      } else {
        resovle(spark.end());
      }
    };
    fileReader.onerror = function (e) {
      console.warn('oops, something went wrong.');
      reject(e);
    };
    function loadNext() {
      const start = currentChunk * chunkSize,
        end = start + chunkSize >= file.size ? file.size : start + chunkSize;
      fileReader.readAsArrayBuffer(blobSlice.call(file, start, end));
    }
    loadNext();
  });
}
```

## 文本压缩字节流上传

```ts
import JSZip from 'jszip'
import axios from 'axios'

// 获取字符串字节数组
function string2Uint8Array(s: string) {
    const arr = []
    for (let i = 0, j = s.length; i < j; i++) {
        arr.push(s.charCodeAt(i))
    }
    const res = new Uint8Array(arr)
    return res
}

// 获取文件字节流
readFileAsUnit8(file: File): Promise<Uint8Array> {
    return new Promise((resolve, reject) => {
        const reader = new FileReader()
        reader.readAsArrayBuffer(file)
        reader.onloadend = function () {
            if (reader.error) {
                reject(reader.error)
            } else {
                resolve(new Uint8Array(reader.result as ArrayBuffer))
            }
        }
    })
}

const memoryLog = '假设这是一串非常长的日志文本'
const zip = new JSZip()

zip.file('text.log', memoryLog)
// 得到压缩过的文件内容，格式为 blob
const logContent = await zip.generateAsync({ type: "blob", compression: "DEFLATE", compressionOptions: { level: 9 } })
// 传给服务端的格式为 data=logContent，其中logContent是文件的具体内容
const prefix = 'data='
const prefixArray = string2Uint8Array(prefix)
const filesArray = await readFileAsUnit8(data)
// 初始化出一个包含前缀和内容的总长度的无符号整型数组
const mergedArray = new Uint8Array(prefixArray.length + filesArray.length)
mergedArray.set(prefixArray)
mergedArray.set(files, prefixArray.length)

// 通过节流的方式上传到服务端
await axios.post(url, mergedArray, {
    headers: { 'Content-Type': 'application/octet-stream' }
})
```

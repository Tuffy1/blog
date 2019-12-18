---
title: 'File, base64, blob 与 canvas'
date: 2019-11-07 22:26:20
tags:
---

## 预览图片

### 使用 URL 对象

URL.createObjectURL 方法可以生成一个指向某个 File 对象，Blob 对象或者 MediaSource 对象的 URL。

```js
document.getElementById("input").addEventListener("change", function(e) {
  const target = e.target;
  const [file] = target.files;
  console.log(file);
  if (URL) {
    const url = URL.createObjectURL(file); // blob:xxx/xxxxxxx 生成一个url，可以直接通过这个url访问到图片。
    console.log(url);
    $$("preview").setAttribute("src", url);
  }
});
```

URL.createObjectURL 的参数也可以是一个 blob 对象，所以通过此方法也可以将 blob 格式图片显示出来。
当不需要这个 URL 对象时，通过 URL.revokeObjectURL() 方法释放。

### 使用 FileReader 对象

FileReader.readAsDataURL 方法接收一个 File 对象或 Blob 对象为参数，触发 onload 事件，此时 FileReader.result 为对应的 base64 编码。

```js
document.getElementById("input").addEventListener("change", function(e) {
  const target = e.target;
  const [file] = target.files;
  console.log(file);
  // file to base64 使用对象FileReader
  const fr = new FileReader();
  fr.onload = function() {
    console.log(fr.result); // data:image/xxx;base64,/....
    document.getElementById("preview").setAttribute("src", fr.result); // 可以直接将img的src赋值为base64，直接显示图片
  };
  fr.readAsDataURL(file);
});
```

fr.readAsDataURL 的参数也可以是一个 blob 对象，所以通过此方法也可以将 blob 格式图片显示出来。**也可将 blob 转为 base64**

## File 转为 base64, File 转为 Blob

### 使用 Canvas

将 File 通过 URL.createObjectURL 方法得到指向图片的 url，创建一个 Image 对象存放该 url 图片，再绘制在 canvas 上，通过 canvas.toDataURL 方法得到图片的 base64。（常利用 Canvas 对图片进行一定的裁剪）

```js
document.getElementById("input").addEventListener("change", function(e) {
  const target = e.target;
  const [file] = target.files;
  console.log(file);
  if (URL) {
    const url = URL.createObjectURL(file); // blob:xxx/xxxxxxx 生成一个url，可以直接通过这个url访问到图片。
    console.log(url);
    const img = new Image();
    img.src = url;
    img.onload = function() {
      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d");
      ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
      const base64Url = canvas.toDataURL("image/jpeg", 0.5); // 转为base64（可指定图片格式与质量）
      console.log(base64Url); // data:image/jpeg;base64,....
      canvas.toBlob(
        function(blob) {
          // 转为Blob（可指定图片格式与质量）
          console.log(blob); // Blob { size: xxx, type: 'image/jpeg' }
        },
        "image/jpeg",
        0.5
      );
    };
  }
});
```

### 使用 FileReader 对象

```js
document.getElementById("input").addEventListener("change", function(e) {
  const target = e.target;
  const [file] = target.files;
  console.log(file);
  const fr = new FileReader();
  fr.onload = function() {
    console.log(fr.result); // data:image/xxx;base64,/....
  };
  fr.readAsDataURL(file);
});
```

FileReader.readAsArrayBuffer 在 onload 中 result 对应 ArrayBuffer 数据；
FileReader.readAsBinaryString 在 onload 中 result 对应 blob 数据（非标准特性）。

## base64 转为 Blob

FileReader 有直接的标准方法将 Blob 转为 base64。此处写方法将 base64 转为 Blob。

```js
function dataURIToBlob(uri) {
  // 如uri：// data:image/jpeg;base64,/....
  const mimeString = uri
    .split(",")[0]
    .split(":")[1]
    .split(";")[0]; // 例子中得到的是："image/jpeg"
  const byteString = atob(uri.split(",")[1]); // 例子中"/..."部分进行解码
  const arrayBuffer = new ArrayBuffer(byteString.length);
  const intArray = new Uint8Array(arrayBuffer);
  for (let i = 0; i < byteString.length; i += 1) {
    intArray[i] = byteString.charCodeAt(i);
  }
  const blob = new Blob([intArray], { type: mimeString });
  return blob;
}
```

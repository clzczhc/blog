---
title: nodejs批量修改mp3文件名
categories: 前端
date: 2022-09-07 13:53:38
tags:
    - nodejs
    - 小技能
---

# nodejs批量修改mp3文件名

## 前言

最近发现以前的SD卡里很多音乐文件出问题了，在LOST.DIR文件夹里，而且文件名变成了一堆数字，还没有后缀。上网查的数据修复的方法都没用，所以决定自食其力，自己修改。批量修改当然就得先弄个办法使用脚本来实现啦。

## 批量修改后缀

批量，所以我们需要想办法获取文件夹的所有文件。所以需要先使用`fs.readdir()`获取文件夹中所有文件。

> `fs.readdir(path, options, callback)`：
> 
> * `path`：文件夹路径
> 
> * `options`：可选参数，可以设置编码方式等。
>   
>   * `encoding`：默认'utf8'`
>   
>   * `withFileTypes` ：默认 `false`
> 
> * `callback`：回调函数，两个参数。
>   
>   * `err`：如果操作失败，将引发此错误
>   
>   * `files`：文件夹中的文件数组

```js
const fs = require('fs');

fs.readdir('./', function (err, files) {
  if (err) {
    throw err;
  }

  console.log(files);
})
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208281058888.png)

重命名：`fs.rename()`

> `fs.rename(oldPath, newPath, callback`：
> 
> * `oldPath`：旧文件夹路径
> 
> * `newPath`：新文件夹路径
> 
> * `callback`：回调函数，一个参数
>   
>   * `err`：如果操作失败，将引发错误

```js
const fs = require('fs');

fs.readdir('./', function (err, files) {
  if (err) {
    throw err;
  }

  files.forEach(file => {
    if (!file.includes('.js')) {  // 避免修改现在的脚本
      fs.rename(`${file}`, `${file}.mp3`, function (err) {
        if (err) {
          throw err;
        }
      })
    }
  })
})
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208281110198.png)

### 另一个好方法

这个方法是学其他东西时偶然看到的。

1. 新建一个`txt`文件

2. 输入`ren * *.mp3`（如果需要修改mp4后缀为mp3，则是`ren *.mp4 *.mp3`）

3. 修改后缀为`bat`

4. 之后双击这个批处理文件，转换就完成了

## 使用`node-id3`库修改文件名

从上面的图片还是可以发现文件名和歌名、歌手名还是很大区别的，但是mp3文件可能会有歌手、歌名信息。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208281131400.png)

介绍：[node-id3](https://www.npmjs.com/package/node-id3)

主要通过`NodeID3.read()`方法获取mp3的歌手、歌名等。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208281125535.png)

```js
const NodeID3 = require('node-id3');
const fs = require('fs');

fs.readdir('./', function (err, files) {
  if (err) {
    throw err;
  }

  files.forEach(file => {
    if (!file.includes('.js')) {  // 避免修改现在的脚本

      NodeID3.read(file, (err, tags) => {
        if (err) {
          throw err;
        }

        console.log(tags);
      });
    }
  });
})
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208281133809.png)

所以我们只需要修改文件名的时候，用歌手、歌名来命名即可。

```js
const NodeID3 = require('node-id3');
const fs = require('fs');

fs.readdir('./', function (err, files) {
  if (err) {
    throw err;
  }

  files.forEach(file => {
    if (!file.includes('.js')) {  // 避免修改现在的脚本

      NodeID3.read(file, (err, tags) => {
        if (err) {
          throw err;
        }

        let artist = tags.artist;
        let title = tags.title;

        if (artist && title) {

          fs.rename(file, `${artist} - ${title}.mp3`, function (err) {
            if (err) {
              throw err;
            }
          });
        }
      });
    }
  });
})
```

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208281140575.png)

可以发现，还有一些没有修改，看一下报错信息。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208281142138.png)

也就是说文件名还是会有限制的，不能有`/`，刚好这个是歌手，而且有一些歌会有很多歌手，所以可以采用只使用第一个歌手名来命名。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208281146784.png)

但是，这样子还是会有一些不能成功，因为文件名并不只是有不能有`/`的限制而已。

## 使用正则表达式修改限制字符

首先得先知道文件名的具体限制，使用上面的`/`重命名文件，查看提示：

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208281151603.png)

我们可以使用正则表达式将限制字符修改成另外的字符。

这里推荐一个正则表达式可视化网站：[JavaScript Regular Expression Visualizer](https://jex.im/regulex/)

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208281154139.png)

这里没有`\`字符，这是因为字符串不会存在`\`字符，因为它会把后面的字符转义。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208281201721.png)

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208281208820.png)

完美解决。不过，还有一些小问题，有一些mp3没有歌名、歌手那些信息，这些漏网之鱼可以通过音乐软件的听歌识曲来获取。

![](https://raw.githubusercontent.com/13535944743/CLZ_img/master/images/202208281208746.png)

## 完整代码

```js
const NodeID3 = require('node-id3');
const fs = require('fs');

const fileReg = /\/|:|\*|\?|"|<|>|\|/g;

fs.readdir('./', function (err, files) {
  if (err) {
    throw err;
  }

  files.forEach(file => {
    if (!file.includes('.js') && file !== "node_modules") {  // 避免修改现在的脚本

      NodeID3.read(file, (err, tags) => {
        if (err) {
          throw err;
        }

        let artist = tags.artist;
        let title = tags.title;

        if (artist) {
          artist = artist.split('/')[0].replace(fileReg, '-');
        }

        if (title) {
          title = title.replace(fileReg, '-');
        }

        if (artist && title) {

          fs.rename(file, `${artist} - ${title}.mp3`, function (err) {
            if (err) {
              throw err;
            }
          });
        }
      });
    }
  });
})
```

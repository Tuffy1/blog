---
title: js异步操作
date: 2019-06-03 22:24:06
tags:
---

## 并行进行多个异步请求

循环进行异步请求，并打印所有请求结束后所得的数组。
以爬取豆瓣电影 top250 的电影标题为例，不同的方法如何实现异步请求读取并将最终结果放在数组 movies 中。

1. 回调函数处理异步操作

```js

let count = 0; // 设置一个变量，每个请求成功后都加1，作为所有请求是否都成功了的判断标志
function requestMovies(url) {
  request(url, (err, response, body) => {
    if (err === null && response.statusCode === 200) {
      const e = cheerio.load(body); 
      const movieList = e('.item');
      for (let i = 0; i < movieList.length; i += 1) {
        let movieItem = movieList[i];
        let movieInfo = takeMovie(movieItem);
        movies.push(movieInfo);
      }
      count += 1;
      if (count === 10) {
        console.log(movies);
      }
    }
  });
}

for (let i = 0; i < 10; i += 1) {
  const url = `https://movie.douban.com/top250?start=${i * 25}`;
  requestMovies(url);
}

```

2. promise 实现异步操作

```js

function requestMovies(url) {
  const promise = new Promise(function (resolve, reject) {
    // 此处request是一个import进来的方法，故依然使用其支持的回调的方法
    request(url, (err, response, body) => {
      if (err === null && response.statusCode === 200) {
        const e = cheerio.load(body); 
        const movieList = e('.item');
        for (let i = 0; i < movieList.length; i += 1) {
          let movieItem = movieList[i];
          let movieInfo = takeMovie(movieItem);
          movies.push(movieInfo);
        }
        return resolve();
      }
      return reject(err);
    });
  });
  return promise;
}

const promises = [];
for (let i = 0; i < 10; i += 1) {
  const url = `https://movie.douban.com/top250?start=${i * 25}`;
  promises.push(requestMovies(url)); 
}
// 每个请求都返回一个promise，当所有promise都是完成状态时打印movies数组
Promise.all(promises).then(() => {
  console.log(movies);
})

```

3. await/async 异步处理

```js

function requestMovies(url) {
  const promise = new Promise(function (resolve, reject) {
    request(url, (err, response, body) => {
      if (err === null && response.statusCode === 200) {
        const e = cheerio.load(body); 
        const movieList = e('.item');
        for (let i = 0; i < movieList.length; i += 1) {
          let movieItem = movieList[i];
          let movieInfo = takeMovie(movieItem);
          movies.push(movieInfo);
        }
        return resolve(movies);
      }
      return reject(err);
    });
  });
  return promise;
}

const promises = [];
async function collectMovies() { // 异步函数
  for (let i = 0; i < 10; i += 1) {
    const url = `https://movie.douban.com/top250?start=${i * 25}`;
    promises.push(requestMovies(url));
  }
  await Promise.all(promises); // 只有所有请求完成时，才接着往下执行
  console.log(movies);
}
collectMovies();

```

## 多层回调

进行多个异步操作，并且下一个异步操作依赖上一个异步操作的结果。
如果我们想要读取豆瓣top250的电影，并且按顺序读取并输出，即读完第一页再读第二页，读完第二页再读第三页...

1. 回调函数处理异步操作

```js

let count = 0; // 设置全局变量，即是用于获取下一个请求的链接，也是判断目前请求的页数
function requestMovies() {
  const url = `https://movie.douban.com/top250?start=${count * 25}`;
  request(url, (err, response, body) => {
    if (err === null && response.statusCode === 200) {
      const e = cheerio.load(body); 
      const movieList = e('.item');
      for (let i = 0; i < movieList.length; i += 1) {
        let movieItem = movieList[i];
        let movieInfo = takeMovie(movieItem);
        console.log(movieInfo);
      }
      count += 1;
      if (count === 10) {
        return;
      } else {
        requestMovies(); // 请求成功后，进行下一个请求，此处需利用递归实现
      }
    }
  });
}
requestMovies();

```

2. await/async 异步处理

```js

function requestMovies(url) {
  const promise = new Promise(function (resolve, reject) {
    request(url, (err, response, body) => {
      if (err === null && response.statusCode === 200) {
        const e = cheerio.load(body); 
        const movieList = e('.item');
        for (let i = 0; i < movieList.length; i += 1) {
          let movieItem = movieList[i];
          let movieInfo = takeMovie(movieItem);
          console.log(movieInfo);
        }
        return resolve(movies);
      }
      return reject(err);
    });
  });
  return promise;
}

async function collectMovies() {
  for (let i = 0; i < 10; i += 1) {
    const url = `https://movie.douban.com/top250?start=${i * 25}`;
    await requestMovies(url); // 每次只有当请求得到反馈后才进行下一步请求
  }
}
collectMovies();


```
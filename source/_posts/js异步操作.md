---
title: js异步操作
date: 2019-06-03 22:24:06
tags:
---

## 并行进行多个异步请求

循环进行异步请求，并打印所有请求结束后所得的数组。
以爬取豆瓣电影 top250 的电影标题为例，不同的方法如何实现异步请求读取并将最终结果放在数组 movies 中。

1. 回调函数处理异步操作

2. promise 实现异步操作

```

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
        return resolve();
      }
      return reject(err);
    });
  });
  return promise;
}

let count = 0;
const promises = [];
for (let i = 0; i < 10; i += 1) {
  const url = `https://movie.douban.com/top250?start=${i * 25}`;
  promises.push(requestMovies(url));
}
Promise.all(promises).then(() => {
  console.log(movies);
})

```

3. await/async 异步处理

```

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
async function collectMovies() {
  for (let i = 0; i < 10; i += 1) {
    const url = `https://movie.douban.com/top250?start=${i * 25}`;
    promises.push(requestMovies(url));
  }
  await Promise.all(promises); 
  console.log(movies);
}
collectMovies();

```

## 多层回调
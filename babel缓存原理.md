# 原理
webpack 中的 loader 其实就是对文本字符串的处理，给指定的 loader 一个 input，loader 还你一个 output，有点函数式编程的内味了。因此所谓的 babel-loader 其实就是把一段 JavaScript 字符串转换成另一端 JavaScript 字符串。基于以上的前提，因此只要在转换的时候，记录下转换前的文件和转换后的文件，然后对比文件是否有改动，如果文件没有改动那就继续拿上次转换之后的文件，所以就跳过这一次转换的过程，大大提高了速度

```
/ https://github.com/babel/babel-loader/blob/master/src/cache.js#L63
const filename = function(source, identifier, options) {
  const hash = crypto.createHash("md4");

  const contents = JSON.stringify({ source, options, identifier });

  hash.update(contents);

  return hash.digest("hex") + ".json";
};
```
1基于md4 哈希来验证
2.source 对应文件
  options 配置
  identifier 默认使用 @babel/core 的版本号 防止babel-loader升级

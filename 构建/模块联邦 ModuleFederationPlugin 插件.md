webpack提供了一个 ModuleFederationPlugin 插件，能够实现模块的发布和引入， ModuleFederationPlugin 插件常用的配置有以下几个
> name ： 必须，唯一的库名，使用者通过 ［name］/［exposes］使用
> filename ：必须，暴露的文件名，在使用者通过 remotes 来引入
> library ：可选， umd的名称，类似于jQuery的$，lodash的_
> exposes ： 可选，配置暴露指定的模块，供其他人使用
> remotes ： 可选，表示使用其他远程的模块
> shared ：可选， 配置共享库，例如 shared: ['react', 'react-dom'] 意思是宿主和引用依赖的模块共享配置的库，如果host有该库则不需要再次下载，直接使用host已有的库

```
module.exports = {
  // 其他webpack配置...
  plugins: [
    new ModuleFederationPlugin({
        name: 'empBase',
        library: { type: 'var', name: 'empBase' },
        filename: 'emp.js',
        remotes: {
          app_two: "app_two_remote",
          app_three: "app_three_remote"
        },
        exposes: {
          './Component1': 'src/components/Component1',
          './Component2': 'src/components/Component2',
        },
        shared: ["react", "react-dom","react-router-dom"]
      })
  ]
}
```
```
var moduleMap = {
	"./components/Comonpnent1": function() {
		return Promise.all([__webpack_require__.e("webpack_sharing_consume_default_react_react"), __webpack_require__.e("src_components_Close_index_tsx")]).then(function() { return function() { return (__webpack_require__(16499)); }; });
	},
};
var get = function(module, getScope) {
	__webpack_require__.R = getScope;
	getScope = (
		__webpack_require__.o(moduleMap, module)
			? moduleMap[module]()
			: Promise.resolve().then(function() {
				throw new Error('Module "' + module + '" does not exist in container.');
			})
	);
	__webpack_require__.R = undefined;
	return getScope;
};
var init = function(shareScope, initScope) {
	if (!__webpack_require__.S) return;
	var oldScope = __webpack_require__.S["default"];
	var name = "default"
	if(oldScope && oldScope !== shareScope) throw new Error("Container initialization failed as it has already been initialized with a different share scope");
	__webpack_require__.S[name] = shareScope;
	return __webpack_require__.I(name, initScope);
}
```

代码中包括三个部分：

moduleMap：通过exposes生成的模块集合
get: host通过该函数，可以拿到remote中的组件
init：host通过该函数将依赖注入remote中
总结

首先，mf会让webpack以filename作为文件名生成文件
其次，文件中以var的形式暴露了一个名为name的全局变量，其中包含了exposes以及shared中配置的内容
最后，作为host时，先通过remote的init方法将自身shared写入remote中，再通过get获取remote中expose的组件，而作为remote时，判断host中是否有可用的共享依赖，若有，则加载host的这部分依赖，若无，则加载自身依赖。
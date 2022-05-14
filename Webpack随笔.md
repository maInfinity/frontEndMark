# 1.Webpack的理解

①Webpack是一个**模块打包工具**，可以使用它**管理项目中的模块依赖**，并编译输出模块所需的静态文件。

②它可以很好地**管理、打包开发中所用到的HTML,CSS,JavaScript和静态文件（图片，字体）**等，让开发更高效。

③对于不同类型的依赖，Webpack有对应的模块加载器，而且会分析模块间的依赖关系，最后合并生成优化的静态资源。

# 2.常见的loader

1> raw-loader

2> file-loader

3> url-loader

4> image-loader

5> css-loader

6> sass-loader

7> less-loader

8> style-loader

9> vue-loader

10> eslint-loader

11> tslint-loader

12> json-loader

13> babel-loader

14> svg-inline-loader

# 3.常见的plugin

1>define-plugin

2>ignore-plugin

3>uglifyjs-webpack-plugin

4>terser-webpack-plugin

5>mini-css-extract-plugin

6>html-webpack-plugin

7>web-webpack-plugin

8>clean-webpack-plugin

9>serviceworker-webpack-plugin

10>webpack-parallel-uglify-plugin

# 3.Loader和Plugin的区别

- loader 加载器

>`Webpack` 将一切文件视为模块，但是 `webpack` 原生是**只能解析 `js` 文件**. `Loader` 的作用是让 `webpack` 拥有了**加载和解析非 `JavaScript` 文件**的能力
>
>在 `module.rules` 中配置，也就是说他作为模块的解析规则而存在，类型为数组

- Plugin 插件

> 扩展 `webpack` 的功能，让 `webpack` 具有更多的灵活性
>
> 在 `plugins` 中单独配置。类型为数组，每一项是一个 `plugin` 的实例，参数都通过构造函数传入


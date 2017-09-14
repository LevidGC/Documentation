IdentityServer3 文档（中文）
============================================

> 个人翻译，不代表官方意图。

站点：https://identityserver.github.io/Documentation/

## 如何构建

假设你已经安装了 Ruby 和 [Bundler](http://bundler.io/) 。

* 导航到项目源码目录
* 运行 `bundle install` 来安装必备的模块
* 修改 _config.yml 文件，将 URL 更改为 `url: http://localhost:4000`
 * 这将确保相对链接按预期工作
 * **注意:** 不要提交任何更改
* 运行 `bundle exec jekyll serve` 启动 Web 站点
# PHP 7 兼容性检查器（php7cc）

English documentation: [README.md](README.md)

## 简介

php7cc 是一个命令行工具，帮助将 PHP 5.3–5.6 项目迁移到 PHP 7。它会扫描代码中可能产生兼容性问题的语句，并报告文件名、行号和问题描述。php7cc 只负责检查，不会自动修改代码。

它报告两类问题：

1. **错误**：在 PHP 7 中可能导致致命错误、语法错误、提示等问题，以红色显示。
2. **警告**：代码在 PHP 5 和 PHP 7 中都合法，但行为可能发生变化，进而引起逻辑问题，以黄色显示。

可能触发报告的语句可参考 [php-src 的 PHP 7 升级说明](https://github.com/php/php-src/blob/PHP-7.0/UPGRADING)。静态检查无法保证发现所有问题，迁移后仍应运行完整的项目测试。

## 环境要求

- PHP 5.3.3 或更高版本；支持 PHP 7。
- 安装 php7cc 时可能需要 [Composer](https://getcomposer.org/)。

如果待检查文件存在语法错误（例如无效数字字面量或无效的 UTF-8 码点转义），工具无法继续分析该文件，只会报告遇到的第一个语法错误。

## 安装

### Phar 包

从 [GitHub Releases](https://github.com/sstalle/php7cc/releases) 下载稳定版本的 Phar 包。

### 使用 Composer 全局安装

```bash
composer global require afei/php7cc
```

建议将 Composer 全局可执行文件目录加入 `PATH`。目录位置因操作系统而异，可参考 [Composer 文档](https://getcomposer.org/doc/03-cli.md#composer-home)。部分类 Unix 系统可使用：

```bash
export PATH="$PATH:$HOME/.config/composer/vendor/bin"
```

之后即可直接运行 `php7cc`。

### 在项目中本地安装

在项目目录中运行：

```bash
composer require afei/php7cc --dev
```

本地安装的可执行文件位于 `vendor/bin/php7cc`。

### Docker 镜像

Docker 镜像由贡献者 [ypereirareis](https://github.com/ypereirareis) 维护，可在 [Docker Hub](https://hub.docker.com/r/ypereirareis/php7cc/) 获取。

## 使用方法

查看所有命令行选项：

```bash
php7cc --help
```

### 检查文件或目录

将文件或目录作为参数传入。目录会递归检查；默认只处理 `.php` 文件。

```bash
php7cc /path/to/file.php
php7cc /path/to/project/
```

### 指定要检查的扩展名

使用逗号分隔扩展名：

```bash
php7cc --extensions=php,inc,lib /path/to/project/
```

### 排除文件或目录

`--except` 可重复指定要排除的绝对路径，或相对于待检查目录的路径：

```bash
php7cc --except=vendor --except=/path/to/project/test /path/to/project/
```

上例会跳过项目中的 `vendor` 目录、指定的 `test` 目录及其内容。

### 设置最低问题级别

`--level` 可设为 `info`、`warning` 或 `error`。`info` 当前与 `warning` 等效。

```bash
php7cc --level=error /path/to/project/
```

上例只显示错误，不显示警告。

### 指定输出格式

支持 `plain`（默认）和 `json` 两种格式。可使用 `--output-format` 或简写 `-o`：

```bash
php7cc -o json /path/to/project/ | json_pp
```

JSON 输出包含检查文件列表、每个文件的问题及检查摘要，例如：

```json
{
  "summary": {
    "elapsedTime": 0.006,
    "checkedFiles": 3
  },
  "files": [
    {
      "name": "/path/to/project/example.php",
      "errors": [],
      "warnings": [
        {
          "text": "String containing number in hexadecimal notation",
          "line": 13
        }
      ]
    }
  ]
}
```

### 其他选项

- `--relative-paths`（`-r`）：输出相对于待检查目录的文件路径。
- `--integer-size`：指定目标系统整数位数，用于位移运算检查。

## 故障排查

### 达到最大函数嵌套层级

如果出现 `Maximum function nesting level ... reached`，可在 PHP 或 Xdebug 配置中提高嵌套层级，例如：

```ini
xdebug.max_nesting_level = 1000
```

### PHP 内存耗尽

可提高 CLI PHP 可用内存，或关闭内存限制。例如：

```bash
php -d memory_limit=-1 php7cc.php /path/to/project
```

## 其他

- 贡献指南：[CONTRIBUTING.md](CONTRIBUTING.md)
- 贡献者列表：[GitHub Contributors](https://github.com/afei/php7cc/graphs/contributors)

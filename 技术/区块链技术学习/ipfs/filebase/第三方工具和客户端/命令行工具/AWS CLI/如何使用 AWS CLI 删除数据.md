# 如何使用 AWS CLI 删除数据
	了解如何使用 AWS CLI 从 Filebase 存储桶中删除大量对象或特定对象组
Filebase Web 仪表板支持一次删除多达 1,000 个对象。在某些情况下，一次删除 1,000 个对象可能既乏味又耗时。Web 仪表板还支持清空整个桶。虽然这在某些情况下可能是最快的解决方案，但其他情况下可能只需要删除某些文件，而不仅仅是存储桶中的所有内容。

本指南将展示如何使用 AWS CLI 更有效地从命令行删除存储桶中的所有对象，以及如何删除大量特定对象。

- 先决条件：
	- 注册一个免费的 Filebase 帐户。
	- 安装和配置 AWS CLI。有关如何执行此操作的步骤，请参阅 AWS CLI 指南。

## 删除桶中的所有对象
要删除存储桶中的所有对象，请使用命令

	aws --endpoint https://s3.filebase.com s3 rm --recursive s3://bucket_name/。
- 例如，要删除存储桶“filebase-bucket”中的所有对象：

		aws --endpoint https://s3.filebase.com s3 rm --recursive s3://filebase-bucket/

## 删除存储桶中的特定对象组
如果要删除符合特定命名约定的对象，可以使用

	aws ——endpoint https://s3.filebase.com s3 rm s3://bucket_name/
命令中的 `——exclude` 或 `——include` 标志。这些标志后面是对象名称需要匹配的模式，以便从 delete 命令中包含或排除。

支持以下字符模式与 `——exclude` 和 `——include` 标志一起使用。

- `*`: 匹配所有内容
- `？`: 匹配任何单个字符
- `[sequence]` ：匹配序列中的任意字符
- `[!sequence]` ：匹配任何不按顺序的字符

例如，要删除所有以“.jpg”结尾的文件，可以使用命令：

	aws --endpoint https://s3.filebase.com s3 rm --include “*.jpg” s3://filebase-bucket
要删除除以“.txt”结尾的文件之外的所有文件，可以使用命令：

	aws --endpoint https://s3.filebase.com s3 rm --exclude “*.txt” s3://filebase-bucket

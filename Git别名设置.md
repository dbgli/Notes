#Git 

一般我不喜欢设置太多的命令别名，因为原教旨主义，避免对自定义配置的依赖。不过查看提交记录时还是有必要别名一下的。方便起见，在`~/.gitconfig`的alias段中加入下面内容，分别表示查看记录和查看完整记录：
```bash
lg = log --color --graph --pretty=format:'%C(red)%h%C(reset) -%C(yellow)%d%C(reset) %s %C(green)(%cd)%C(reset) %C(blue bold)<%an>%C(reset)' --date=format:'%Y-%m-%d %H:%M:%S'
lgf = log --format=fuller
```

默认的log命令只显示Author和AuthorDate，如果遇到rebase，或者花费时间处理合并冲突，再或者amend修改提交，会导致CommitDate和AuthorDate不一致，有必要将两者区分。lg别名只选取Author和CommitDate，看似不一致，但是GitKraken在提交列表中也是这么做的。lgf别名则完整显示作者，提交者以及对应的日期。
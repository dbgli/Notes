#Ubuntu #异体字 #fontconfig

安装完Ubuntu后，如果系统语言不是简体中文的话，对于某些中文字符，会显示为奇怪的异体字，比如：“门”、“直”、“靠”、“将”等等。这是因为对于CJK字体，配置文件里并没有把简体中文排在日韩前面，所以显示了日语里的汉字字形。

对于Ubuntu24.04，编辑`/etc/fonts/conf.d/64-language-selector-cjk-prefer.conf`文件，这是一个软链接，指向`/usr/share/fontconfig/conf.avail/64-language-selector-cjk-prefer.conf`，内容如下：
```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
    <alias>
        <family>sans-serif</family>
        <prefer>
            <family>Noto Sans CJK JP</family>
            <family>Noto Sans CJK KR</family>
            <family>Noto Sans CJK SC</family>
            <family>Noto Sans CJK TC</family>
            <family>Noto Sans CJK HK</family>
        </prefer>
    </alias>
    <alias>
        <family>serif</family>
        <prefer>
            <family>Noto Serif CJK JP</family>
            <family>Noto Serif CJK KR</family>
            <family>Noto Serif CJK SC</family>
            <family>Noto Serif CJK TC</family>
        </prefer>
    </alias>
    <alias>
        <family>monospace</family>
        <prefer>
            <family>Noto Sans Mono CJK JP</family>
            <family>Noto Sans Mono CJK KR</family>
            <family>Noto Sans Mono CJK SC</family>
            <family>Noto Sans Mono CJK TC</family>
            <family>Noto Sans Mono CJK HK</family>
        </prefer>
    </alias>
</fontconfig>
```

简单地将SC简体中文、TC台湾繁体、HK香港繁体挪到前面，重启电脑即可。
```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
    <alias>
        <family>sans-serif</family>
        <prefer>
            <family>Noto Sans CJK SC</family>
            <family>Noto Sans CJK TC</family>
            <family>Noto Sans CJK HK</family>
            <family>Noto Sans CJK JP</family>
            <family>Noto Sans CJK KR</family>
        </prefer>
    </alias>
    <alias>
        <family>serif</family>
        <prefer>
            <family>Noto Serif CJK SC</family>
            <family>Noto Serif CJK TC</family>
            <family>Noto Serif CJK JP</family>
            <family>Noto Serif CJK KR</family>
        </prefer>
    </alias>
    <alias>
        <family>monospace</family>
        <prefer>
            <family>Noto Sans Mono CJK SC</family>
            <family>Noto Sans Mono CJK TC</family>
            <family>Noto Sans Mono CJK HK</family>
            <family>Noto Sans Mono CJK JP</family>
            <family>Noto Sans Mono CJK KR</family>
        </prefer>
    </alias>
</fontconfig>
```
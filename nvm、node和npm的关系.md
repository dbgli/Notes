#nodejs #nvm #npm #TypeScript

一开始写js放浏览器运行，后来需要独立运行，于是有了node.js运行时。
后来又需要一个包管理器，于是有了npm。
再后来需要多版本管理，又有了nvm来管理node和npm。

后来写js不得劲，要写ts，于是通过npm安装typescript。
有了tsc每次编译成js很麻烦，于是再通过npm安装ts-node直接运行ts。
但是ts-node需要先`tsc init`下生成tsconfig.json才行。
于是`node demo.js`和`ts-node demo.ts`运行代码。

还有其它的包管理器和版本管理工具，以后再说。
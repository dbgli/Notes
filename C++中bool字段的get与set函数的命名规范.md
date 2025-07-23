#CPP #命名规范 

一般情况下数据成员以名词命名，取值和设值时直接加get和set前缀即可，例如字段`name_`，对应的取值和设值函数为`getName()`和`setName()`。但是当数据成员是bool类型时，变量名已经包含了动词前缀，因此需要特殊对待。

## bool字段的命名风格

按使用频率从高到低排，主要有**isAdj/hasNoun/canDo**这三类，Java中主推is作为前缀，但我们这里允许灵活选择。当然，如果愿意的话，后两种几乎总是可以转换成第一种形式，例如`hasAccess_`可以写成`isAccessible_`，`hasLicense_`可以写成`isLicensed_`，`canRead_`可以写成`isReadable_`。

## bool字段的get取值函数

直接以字段名作为函数名，不需要添加get前缀。因为**isAdj/hasNoun/canDo**比get更直观，且在if语句中更符合自然语言，例如字段`hasLicense_`，使用取值函数时`if app.hasLicense()`。

## bool字段的set设值函数

如果字段为**isAdj**的形式，应命名为**setAdj**，例如字段`isBusy_`，对应的设值函数为`setBusy()`，如果用`setIsBusy()`则太冗长了。

如果字段为**hasNoun/canDo**的形式，应命名为**setHasNoun/setCanDo**，例如字段`hasAccess_`和`canRead_`，对应的设值函数为`setHasAccess()`和`setCanRead()`，这时就没有什么好的既通用又简洁的模式了。

如果字段为**isAdjNoun**的形式，例如`isValidSalary_`，应遵循**hasNoun**的规则，即设值函数为`setIsValidSalary()`。不过这种情况建议转换为其它形式，如果属于`Salary`类，直接写`isValid_`，如果属于更上层的例如`Employee`类，应该命名为`hasValidSalary_`，再套用各自的规则。

## 另一种简洁方案

除了bool类型的get函数直接以字段名作为函数名，其余所有get/set函数均在原字段名前加get/set。

## 灵活性

最后，也不一定强制要求get/set函数必须和字段名一致，比如标准库中的`resize()`，就相当于`setSize()`函数，只要语义明确且以动词开头即可。
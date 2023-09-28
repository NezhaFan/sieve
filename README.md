
### 介绍
- 使用DFA算法实现关键词/敏感词检测。有问题可以联系我 QQ:`772532526`

- 优点：
	- 快速。复杂度O(1)
	- 忽略英文大小写。
	- 忽略常见符号。例如设置关键字`傻逼`，则`傻-逼`、`傻、逼`也能被识别到。
	- 支持通配符*。一个通配符*必定匹配一个非符号字符。 所以`fu*k`不能识别`fuk`、`fu.k`、`fucck`
	- 打标签。对关键词分类，更好地定制化处理。例如人工分类多个种类的关键词：政治、色情、辱骂、广告 ...
	- 选择性替换。检测得到但是不替换文本。例如你希望对疑似营销、广告的关键词不进行替换，仅灰字提示消息接受者注意。

- 其他说明
	- 关键词的录入会忽略符号。 通配符*不允许放在第一位。
	- `傻逼`的等价词很多`傻b`、`傻x`、`傻bi`等，使用`傻*`固然可以全杀此类，但这不是一个好方式，`傻子`不应该被禁。主要还是它过短及可能性太多。  `迷*药	`就不太存在误杀的可能。 
	- 中文很麻烦的一点是前后词的联动，例如`操你*`你觉得应该禁掉，但是它会误杀`广播体操你会吗`。 还有`色情`、`白色情人节`。 特例很少，但通配符的使用还是要很慎重。如果确定只存在两三种情况，建议都列出来添加。
	- 本项目不包含敏感词库。如果你没有可以参考 `https://github.com/fwwdn/sensitive-stop-words`


### 函数说明
- [x] `New() *Sieve` 创建新的实例
- [x] `(*Sieve) Add(words []string) (fail []string)` 添加。返回失败的词
- [x] `(*Sieve) AddByFile(filename string, tag uint8, autoReplace bool) (fails []string, err error)` 从文件中添加。设置标签及替换与否。（支持远程文件，但不建议）
- [x] `(*Sieve) Search(text string) (string, uint8)` 搜索。返回第一个匹配到的关键词和其标签。
- [x] `(*Sieve) Replace(text string) (string, map[uint8][]string)` 替换。返回替换后的文本和其包含的关键词情况。


### 效果
```sh
===== 基础用法：添加、移除、搜索、替换
添加: 苹果 西红柿 葡萄
移除: 葡萄
测试: 我想吃葡萄和西红柿，苹果也不错
搜索关键词: 西红柿
替换后: 我想吃葡萄和***，**也不错

===== 进阶用法：通配符、忽略大小写、符号干扰无效
添加: fuck 操你*
测试: FUCK!我操你x、操你🐎、操你&x
替换后: ****!我***、***、****

===== 特殊功能：打标签
添加词典: ./keyword 设置标签: 1 自动替换
测试: 你是傻b么？这么傻呢！
替换后: 你是**么？这么傻呢！
包含敏感词:  map[1:[傻b]]

===== 特殊功能：不替换 (仅发现，另行处理) =====
添加词典: ./keyword 设置标签: 2 不替换
测试: 二手房怎么样
替换后: 二手房怎么样
包含敏感词:  map[2:[二手]]
```

1. 引用
```sh
go get -u github.com/tomatocuke/sieve@latest
```

2. 使用
```go
package main

import (
	"fmt"
	"strings"
	"testing"
)

var (
	filter = New()

	myDemoKeywords = "./keyword"
)

func main() {
	// ===== 基础用法：添加、移除、搜索、替换 =====
	demo1()

	// ===== 进阶用法：通配符、忽略大小写、符号干扰无效 =====
	demo2()

	// ===== 特殊功能：打标签 (用于区分敏感词类型) =====
	demo3()

	// ===== 特殊功能：不替换 (仅发现) =====
	demo4()
}

func demo1() {
	// 添加
	filter.Add([]string{"苹果", "西红柿", "葡萄"})
	// 移除
	filter.Remove([]string{"葡萄"})
	const text = "我想吃葡萄和西红柿，苹果也不错"
	// 搜索 (第一个关键词)
	searchKeyword, _ := filter.Search(text)
	// 替换
	replaceText, _ := filter.Replace(text)

	fmt.Println("\n===== 基础用法：添加、移除、搜索、替换")
	fmt.Println("添加:", "苹果", "西红柿", "葡萄")
	fmt.Println("移除:", "葡萄")
	fmt.Println("测试:", text)
	fmt.Println("搜索关键词:", searchKeyword)
	fmt.Println("替换后:", replaceText)
}

func demo2() {
	const text = "FUCK!我操你x、操你🐎、操你&x"
	filter.Add([]string{"fuck", "操你*"})
	replaceText, _ := filter.Replace(text)

	fmt.Println("\n===== 进阶用法：通配符、忽略大小写、符号干扰无效")
	fmt.Println("添加:", "fuck", "操你*")
	fmt.Println("测试:", text)
	fmt.Println("替换后:", replaceText)
}

// 设置分类标签
const (
	TagDefault = iota
	TagInsult
	TagTrade
)

func demo3() {
	const text = "你是傻b么？这么傻呢！"
	fails, err := filter.AddByFile(myDemoKeywords, TagInsult, true)
	if err != nil {
		panic(err)
	}
	replaceText, keywords := filter.Replace(text)

	fmt.Println("\n===== 特殊功能：打标签")
	fmt.Printf("添加词典: %s 设置标签: %d 自动替换\n", myDemoKeywords, TagInsult)
	if len(fails) > 0 {
		fmt.Println("添加失败:", fails)
	}
	fmt.Println("测试:", text)
	fmt.Println("替换后:", replaceText)
	fmt.Println("包含敏感词: ", keywords)
}

func demo4() {
	const text = "二手房怎么样"
	filter.AddByFile(myDemoKeywords, TagTrade, false)
	replaceText, keywords := filter.Replace(text)

	fmt.Println("\n===== 特殊功能：不替换 (仅发现，另行处理) =====")
	fmt.Printf("添加词典: %s 设置标签: %d 不替换\n", myDemoKeywords, TagTrade)
	fmt.Println("测试:", text)
	fmt.Println("替换后:", replaceText)
	fmt.Println("包含敏感词: ", keywords)
}

var longText = strings.Repeat("哦😯哈HA", 20) // 100字符

func BenchmarkReplace(b *testing.B) {
	for i := 0; i < b.N; i++ {
		filter.Replace(longText)
	}
}

func BenchmarkSearch(b *testing.B) {
	for i := 0; i < b.N; i++ {
		filter.Search(longText)
	}
}
```


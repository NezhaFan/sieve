## 完全自己写的，喜欢的同学给个🌟，我需要你

### 介绍
- 使用DFA算法实现关键词/敏感词检测
- 特点。
	1. 快速。这是基本要求，本项目对字符串最多完整便利一次，复杂度O(1)
	2. 并发安全。 可动态增删查询
	3. 自动忽略英文大小写和无意义字符。
	4. 支持通配符*。杀伤范围巨大。这点在别的项目没看到
	5. 支持标签分类。 根据分类可以更有好地自定义处理方式
	6. 支持选择性替换。 如「涉嫌交易」，我希望检查出来，但是不需要替换。
- 敏感词库。网上有，几万条，我觉得太混乱了，不准确且矫枉过正。我整理了一些[敏感词库](https://github.com/kikiakikia/keyword)，供下载到本地读取。
- 有问题请务必联系我 QQ:`772532526`

### 使用

```sh
go get -u github.com/tomatocuke/sieve
```

```go
package main

import (
	"fmt"

	"github.com/tomatocuke/sieve"
)

var (
	filter = sieve.New()
)

func main() {

	// ======== 简单用法 =========
	text := "我有香水、苹果4S手机、苹果派和红苹-果"

	// 添加 (可重复添加，苹果掺杂的符号会过滤掉只添加「苹果」。数字和字母是有效的)
	filter.Add([]string{"香蕉", "苹,。果", "苹果派", "苹果4s手机"})
	// 移除
	filter.Remove([]string{"苹果派"})

	s, _ := filter.Search(text)
	// 搜索: 苹果4S手机 （返回第一个匹配到的关键词的最大长度)
	fmt.Println("搜索:", s)

	// 替换: 我有香水、******、**派和红*** 没有苹果手机
	fmt.Println("替换:", filter.Replace(text), filter.Replace("没有苹果手机"))






	// ======== 高阶用法 =========

	// 1. 模糊匹配。
	// 使用通配符*，但是不能处于第一个位置。一个*最多通配一个字符。
	// 此类关键词设置需要谨慎，尤其是单个字接*，防止误杀。

	// 举例，「草你**」明显是骂人的。
	// 但是「我操**」就不一定，可能是「我操作很快」。
	filter.Add([]string{"草你**"})

	// 打印：通配符模式替换: 我草！我****的，我****
	fmt.Println(
		"通配符模式替换:",
		filter.Replace("我草！我草你大爷的，我草你X"), // 最多覆盖2个字
	)

	// 2. 标签分类 和 默认替换
	// 需要自定义标签
	const (
		tag1  = iota + 1 // 水果，需要被替换
		tag2             // 交易，存在风险，不替换，给接收者提示
	)
	// 给苹果和桃子打标签1，设置为可被替换。
	filter.AddWithTag([]string{"苹果", "桃子"}, tag1, true)
	// 涉嫌推广交易。  设置为不替换文本，自己另行处理
	filter.AddWithTag([]string{"多少钱"}, tag2, false)

	// 分别查找是否有分类1和分类2
	text = "有苹果和桃子吗，多少钱"
	s, has := filter.ReplaceAndCheckTags(text, []uint8{tag2})

	// 打印：替换并检查是否涉嫌交易: 有**和**吗，多少钱 true
	fmt.Println("替换并检查是否涉嫌交易:", s, has)
}

```
打印结果
```sh
搜索: 苹果4S手机
替换: 我有香水、******、**派和红*** 没有****
通配符模式替换: 我草！我****的，我***
替换并检查是否涉嫌交易: 有**和**吗，多少钱 true
```

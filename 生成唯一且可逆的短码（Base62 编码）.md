生成**唯一且可逆的短码（Base62 编码）**，本质上是把一个**整数 ID** 用 **62 进制**表示，从而得到更短、更适合 URL 的字符串。这在短链接、订单号、邀请码等场景非常常见。

---

## 一、什么是 Base62

Base62 使用 62 个字符：

```
0-9  → 10
a-z  → 26
A-Z  → 26
```

共 **62 个字符**，相比十进制或十六进制，同样的数字可以用更短的字符串表示。

---

## 二、核心思路（保证唯一 & 可逆）

**前提：**

* 你有一个 **唯一的整数 ID**（如数据库自增 ID、Snowflake ID）

**流程：**

```
ID（int） ⇄ Base62 编码字符串
```

* **唯一性**：ID 唯一 → 编码结果唯一
* **可逆性**：Base62 是数学进制转换，可 100% 解码回原 ID

⚠️ Base62 **不是加密**，只是编码

---

## 三、Base62 编码与解码算法

### 字符表（顺序必须固定）

```text
0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ
```

---

## 四、Go 语言实现示例（推荐）

### 1️⃣ 编码：int → Base62

```go
const base62Chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"

func EncodeBase62(num int64) string {
	if num == 0 {
		return "0"
	}

	result := make([]byte, 0)
	base := int64(len(base62Chars))

	for num > 0 {
		remainder := num % base
		result = append([]byte{base62Chars[remainder]}, result...)
		num /= base
	}
	return string(result)
}
```

---

### 2️⃣ 解码：Base62 → int

```go
func DecodeBase62(str string) int64 {
	var num int64
	base := int64(len(base62Chars))

	index := make(map[rune]int64)
	for i, c := range base62Chars {
		index[c] = int64(i)
	}

	for _, c := range str {
		num = num*base + index[c]
	}
	return num
}
```

---

### 3️⃣ 使用示例

```go
id := int64(123456789)

code := EncodeBase62(id)
fmt.Println(code) // e.g. 8m0Kx

original := DecodeBase62(code)
fmt.Println(original) // 123456789
```

✅ **完全可逆**
✅ **无碰撞**
✅ **URL 安全**

---

## 五、长度对比

| 数字        | 十进制长度 | Base62 |
| --------- | ----- | ------ |
| 999999    | 6     | 4      |
| 123456789 | 9     | 5      |
| 10¹²      | 13    | 7      |

---

## 六、实际项目常见增强方案

### 1️⃣ 固定长度（补零）

```go
fmt.Sprintf("%06s", EncodeBase62(id))
```

---

### 2️⃣ 防止被猜（加盐 / 混淆）

> Base62 本身可逆，很容易被枚举

**方案：**

* ID ⊕ secret
* Hashids（推荐）
* 雪花 ID + Base62

```go
encoded := EncodeBase62(id ^ secret)
```

---

### 3️⃣ 适用场景

| 场景       | 是否合适 |
| -------- | ---- |
| 短链接      | ✅    |
| 邀请码      | ✅    |
| 订单号      | ✅    |
| 安全 Token | ❌    |

---

## 七、总结

✔ Base62 = **进制转换，不是加密**
✔ 唯一性来自 **原始 ID**
✔ 可逆性来自 **固定字符表**
✔ 适合：短、可读、URL 友好 ID

---

如果你愿意，我可以：

* 🔐 帮你改成 **Hashids 安全版**
* ⚡ 帮你适配 **Snowflake ID**
* 📦 封装成一个 **Go 工具包**

你更打算用在哪种业务场景？

# MicroPython GT20L16S1Y 字库驱动

适用于 GT20L16S1Y 字库芯片的 MicroPython SPI 驱动。它可以按地址读取字模，也封装了 GB2312 中文和多种 ASCII 字体的点阵访问接口，适合配合 OLED、LCD 等屏幕显示中文及字符。

![GT20L16S1Y 字库模块](img.jpg)

## 特性

- 纯 MicroPython 实现，无第三方依赖。
- 支持 GB2312 16x16 中文字模。
- 支持 GB2312 国标扩展字符字模。
- 支持 5x7、7x8、8x16、8x16 粗体 ASCII 字模。
- 支持 16 点阵 Arial、Times New Roman 不等宽 ASCII 字模。
- 附带多个 `ssd1306` 显示示例，便于接入实际项目。

> API 方法名中的 `ascll` 是历史拼写，为保持兼容保留；对应内容为 ASCII 字模。

## 目录结构

```text
.
├── gt20l.py          # 驱动源码
├── example/          # 使用示例
├── help/             # 字模排列和接线参考图片
└── img.jpg           # 字库模块图片
```

## 安装

1. 将 `gt20l.py` 复制到 MicroPython 设备文件系统的根目录或当前程序目录。
2. 确保 `gt20l.py` 与调用它的脚本处于同一导入路径中。
3. 在程序中导入并创建驱动对象：

```python
from machine import Pin, SPI
import gt20l

cs = Pin(15, Pin.OUT)
cs.value(1)

# SPI 的编号、引脚和参数需要根据你的开发板修改。
spi = SPI(
    -1,
    mosi=Pin(13),
    miso=Pin(12),
    sck=Pin(14),
)

font = gt20l.gt20l(spi, cs)
```

## 接线参考

以下为仓库示例使用的引脚，仅供参考，实际请以所用开发板和字库模块为准：

| 字库模块 | 开发板示例引脚 | 说明 |
| --- | --- | --- |
| CS | GPIO 15 | 片选，低电平有效，由驱动控制 |
| SCK | GPIO 14 | SPI 时钟 |
| MOSI | GPIO 13 | 主机发送到字库 |
| MISO | GPIO 12 | 字库返回到主机 |
| VCC | 3.3V | 电平必须是 3.3V |
| GND | GND | 必须共地 |

接线参考见 [`help/电路连接参考.jpg`](help/电路连接参考.jpg)。如果读取结果始终为 `0x00` 或 `0xff`，应先检查供电、共地、SPI 引脚和片选信号。

## 快速开始

### 读取 8x16 ASCII 字模

```python
from machine import Pin, SPI
import gt20l

cs = Pin(15, Pin.OUT)
cs.value(1)
spi = SPI(-1, mosi=Pin(13), miso=Pin(12), sck=Pin(14))

font = gt20l.gt20l(spi, cs)
data = font.get_816ascll('a')

print(data)
```

返回结果是一个十六进制字符串列表，例如：

```python
['0x00', '0x00', '0x7c', ...]
```

如果后续需要按字节处理，可以转换为整数：

```python
font_data = [int(value, 16) for value in data]
```

### 读取 GB2312 中文字模

`get_gb2312_font()` 接收 `[高字节, 低字节]` 形式的两字节国标码。下例读取“中”字：

```python
data = font.get_gb2312_font([0xD6, 0xD0])
print(data)
```

这里 `0xD6` 是“中”的 GB2312 高字节，`0xD0` 是低字节。一个 16x16 中文字模通常返回 32 个字节。

## API 说明

### `gt20l(spi, cs)`

创建字库对象。

- `spi`：已经初始化好的 `machine.SPI` 对象。
- `cs`：字库片选引脚，类型为 `machine.Pin`，建议配置为输出模式。

### `get_font(addr, byte)`

按字库内部地址读取原始字模数据。

- `addr`：24 位内部地址。
- `byte`：读取字节数。
- 返回：指定长度的十六进制字符串列表。

这是底层接口，通常在实现新字库区段时使用。

### `get_gb2312_font(gb)`

读取 GB2312 中文字模。

- `gb`：`[高字节, 低字节]` 列表，例如 `[0xD6, 0xD0]`。
- 返回：通常为 32 字节的十六进制字符串列表。

### `get_gb2312_efont(gb)`

读取 GB2312 国标扩展字符字模。

- `gb`：合并后的国标内码整数，例如 `0xAAA1`。
- 当前源码支持的范围为 `0xAAA1` 到 `0xAAFE`，以及 `0xABA1` 到 `0xABC0`。
- 返回：通常为 16 字节的十六进制字符串列表。

### `get_57ascll(let)`

读取 5x7 ASCII 字模。

- `let`：单个 ASCII 字符字符串，例如 `'A'`。
- 返回：8 个十六进制字符串。

### `get_78ascll(let)`

读取 7x8 ASCII 字模。

- `let`：单个 ASCII 字符字符串。
- 返回：8 个十六进制字符串。

### `get_816ascll(let)`

读取 8x16 ASCII 字模。

- `let`：单个 ASCII 字符字符串。
- 返回：16 个十六进制字符串。

### `get_16Arial_ascll(let)`

读取 16 点阵 Arial 不等宽 ASCII 字模。

- `let`：单个 ASCII 字符字符串。
- 返回：34 个十六进制字符串。

### `get_816bold_ascll(let)`

读取 8x16 粗体 ASCII 字模。

- `let`：单个 ASCII 字符字符串。
- 返回：16 个十六进制字符串。

### `get_TimesNewRoman_ascll(let)`

读取 16 点阵 Times New Roman 不等宽 ASCII 字模。

- `let`：单个 ASCII 字符字符串。
- 返回：34 个十六进制字符串。

## 绘制字模

驱动只负责返回字模数据，不负责直接驱动屏幕。不同屏幕的像素排列、颜色值和坐标方向并不相同，通常需要以下步骤：

1. 将十六进制字符串转换为整数或位数据。
2. 按屏幕要求逐位设置像素。
3. 调用屏幕对象的 `show()` 或等价方法刷新显示。

仓库中的示例可以作为起点：

- [`example/example.py`](example/example.py)：读取 GB2312 中文字模。
- [`example/example2.py`](example/example2.py)：在 SSD1306 上显示 8x16 ASCII 字符串。
- [`example/example3.py`](example/example3.py)：在 SSD1306 上显示一个 GB2312 汉字。
- [`example/example4.py`](example/example4.py)：封装汉字绘制函数。
- [`help/`](help)：字模排列和接线参考图片。

## 常见问题

### 返回数据全是 `0x00` 或 `0xff`

通常是硬件或通信问题：

- 检查字库模块是否正常供电，并确保使用 3.3V 逻辑电平。
- 检查开发板与字库是否共地。
- 检查 CS、SCK、MOSI、MISO 是否接对。
- 检查 `SPI` 对象的引脚编号、模式和速率是否符合开发板及字库要求。

### 抛出 `NameError: local variable referenced before assignment`

当前驱动没有做完整的参数校验。当输入字符不在支持的 ASCII 范围，或传入的内容不是函数要求的类型时，内部地址变量可能不会被赋值，从而触发该错误。

调用前应确认：

- ASCII 接口传入的是单个字符字符串。
- 字符编码在 ASCII `0x20` 到 `0x7E` 范围内。
- GB2312 接口传入的是正确的字节列表或合并内码。

### 字符显示方向或点阵顺序不对

先确认屏幕驱动的像素方向，再检查字符的位顺序和字节排列。可参考 [`help/`](help) 中的字模使用说明。

## 注意事项

- `get_font()` 及各封装接口返回的是 `['0x00', ...]` 形式的列表，不是 `bytes` 对象。
- 驱动不会自动初始化或重新配置 SPI，需要调用者传入可用的 `SPI` 对象。
- `get_gb2312_font()` 支持 `gb[0] == 0xA9` 的符号区码，并通过低字节 `gb[1]` 计算字模地址。
- 16x16 中文通常为 32 字节；Arial 和 Times New Roman 为不等宽字体，驱动固定读取 34 字节，实际字形宽度应由字模数据处理。

## 反馈

如果发现兼容性问题或字模地址错误，请附上开发板型号、MicroPython 版本、接线方式、调用代码和实际返回值，便于定位问题。

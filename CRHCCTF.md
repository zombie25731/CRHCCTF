---
title: CRHCCTF

---

# CRHCCTF

# Rev
## hahaha
click the .exe in the vm and saw a shut down word

![image](https://hackmd.io/_uploads/HJDhpeGYll.png)

but somehow i found this stuff in `.rdata`
i dont know what is `UTF-16LE` so i google it
- UTF-16 = a way to store Unicode text (all kinds of characters, not just ASCII).
- 16 = each character is stored in 16 bits (2 bytes).
- LE = Little Endian, which means the low byte comes first in memory, then the high byte.
```python
0x66 0x02 => 0x0266
U+0266 = "ɦ"
```

![image](https://hackmd.io/_uploads/HkISCeGYgg.png)

```python=
import unicodedata

# .rdata bytes
data = [
    0x43,0x00, 0x52,0x00, 0x48,0x00, 0x43,0x00, 0x7B,0x00, # 'CRHC{'
    0x66,0x02, 0x51,0x02, 0x66,0x02, 0x51,0x02, 0x66,0x02, 0x51,0x02,
    0x74,0x00, 0x66,0x02, 0x56,0x04, 0x55,0x04, 0x56,0x04, 0x55,0x04,
    0x66,0x00, 0x51,0x02, 0x38,0x01, 0x35,0x04, 0x55,0x04, 0xBB,0x04,
    0x75,0x00, 0x74,0x00, 0x64,0x00, 0xBF,0x03, 0x77,0x00, 0x3F,0x04,
    0x7D,0x00
]

# Convert to bytes
raw = bytes(data)

# Decode as UTF-16LE
decoded = raw.decode("utf-16le")
print("Decoded UTF-16LE:")
print(decoded)

# Normalize to ASCII
homoglyph_map = {
    'ɦ': 'h', 'ɑ': 'a', 'і': 'i', 'ѕ': 's',
    'ĸ': 'k', 'е': 'e', 'һ': 'h', 'ο': 'o', 'п': 'n'
}

ascii_flag = ''.join(homoglyph_map.get(c, c) for c in decoded)
print(ascii_flag)

```
Decoded UTF-16LE :  CRHC{ɦɑɦɑɦɑtɦіѕіѕfɑĸеѕһutdοwп}
ASCII :  CRHC{hahahathisisfakeshutdown}

## Spot the Anomaly
No matter what i enter,it always pop out `Nope.`
![image](https://hackmd.io/_uploads/H1wvezMtgg.png)

trace the code and find out the main function process:
show `Enter code:` -> `sys_read` -> Delete `\n` -> `sys_write('Nope.')` ->`sys_exit`

it do not compare with your `Enter code`
so i found out another usefull place.It is after the `LINUX - sys_exit`
and as u can see lea(load effective address) to rsi,then rsi `xor` with 0x23
![image](https://hackmd.io/_uploads/B1yBfGfKgl.png)

there is the script
```python=
list_bytes = [
    0x60,  # db 60h
    # dq 51427C5658606B71h
    0x71, 0x6B, 0x60, 0x58, 0x56, 0x7C, 0x42, 0x51,
    # dq 12487C6167647C10h
    0x10, 0x7C, 0x64, 0x67, 0x61, 0x61, 0x48, 0x12,
    # dq 73647C51137C444Dh
    0x4D, 0x44, 0x7C, 0x13, 0x51, 0x7C, 0x64, 0x73,
    # dq 1C1C444D4F487C77h
    0x77, 0x7C, 0x48, 0x4F, 0x4D, 0x44, 0x1C, 0x1C
]


flag = ''.join(chr(i ^ 0x23) for i in list_bytes)
print(flag)
```
CRHC{u_ar3_GDBBk1ng_0r_GPT_klng??}
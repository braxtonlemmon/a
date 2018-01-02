# Preface
- serialize
见/py/py-serial.md

# String
like js, double quotes and single quotes is same

	print "a\nb" ;# The character here "\n" is new line
	print 'a\nb'
	print '\x30\x31'; 01
	print '\xb8'; b'\xc2\xb8' 其实是按utf8 解释

## multi line

	s=("str1"
		"str2" )

## string number
	str.isdigit()
	.isnumeric()
	.isdecimal()

	.isalpha
	.isalnum()

## format
https://pyformat.info/

old

	>>> print 'This is (%r) (%s)' % ("Hilojack\"", "Blog")
	This is ('Hilojack"') (Blog)
	>>> print '''This is (%r) (%s)''' % ("Hilojack\"", "Blog")
	This is ('Hilojack"') (Blog)

> %r displays the "raw" data


### value type
```
'{:s}'.format('str')
'{:d}'.format(123)
'{:f}'.format(123.0)
```

### value conversion
```
class Data0(object):
    def __repr__(self):
        return 'räpr'
class Data1(object):
    def __str__(self):
        return 'str'

//Old
'%s %r %a' % (Data0(), Data1())
//New
'{1!s} {0!r} {0!a}'.format(Data0(), Data1())
```
Output
```
str räpr r\xe4pr
```

### Padding and aligning strings
```
# align left
'%-10s' % ('test')
'{:10}'.format('test')
'{:<10}'.format('test')

# align center
'{:^10}'.format('test')

# align right
'%10s' % ('test',)
'{:>10}'.format('test')
      test

```
string default left, number/float default right:
```
>>> '{:5s}'.format('a')
'a    '
>>> '{:5d}'.format(3)
'    3'
```
padding with _
```
'{:_<10}'.format('test')
'{:_^10}'.format('test')
'{:_>10}'.format('test')
```
padding with zero
```
>>> '%06.2f' % (3.141592653589793,)
'003.14'
>>> '{:06.2f}'.format (3.141592653589793,)
'003.14'
```
padding with signed number
```
>>> '{:+06.2f}'.format (+3.141592653589793,)
'+03.14'
>>> '{:=+06.2f}'.format (+3.141592653589793,)
'+03.14'
```
padding with signed number position:
```
>>> '{:+6d}'.format (3)
'    +3'
>>> '{:=+6d}'.format (3)
'+    3'
```

### Truncating long strings
```
>>> '%-.2s' % '12345'
>>> '{:.2}'.format('12345')
'12'
>>> '%-3.2s' % '12345'
>>> '{:3.2}'.format('12345')
'12 '
```
Truncating float
```
'{:.6f}'.format(123.0)
'123.000000'
```

combining truncating and padding
```
'{:12.2f}'.format(123.0)
'      123.00'
```

### named placeholder
```
'hello, {name}'.format(name='Wang')
'hello, {0}-{1}'.format('Wang', 'Kang')
'{p[first]} {p[last]}'.format(p={'first':1, 'last':2})
```
with object:
```
'{p.type}'.format(p=Plant())
```
### datetime
```
from datetime import datetime
'{:%Y-%m-%d %H:%M}'.format(datetime(2001, 2, 3, 4, 5))
```
### parametrized formats
```
'{:^10}'.format('test')
'{:{align}{width}}'.format('test', align='^', width='10')

'%2,2f' % ( 2.7182)
'%*.*f' % (5, 2, 2.7182)
'{:{width}.{prec}f}'.format(2.7182, width=5, prec=2)

'{:{dfmt} {tfmt}}'.format(dt, dfmt='%Y-%m-%d', tfmt='%H:%M')
```

### custom objects
```
class HAL9000(object):
    def __format__(self, format):
        if (format == 'open-the-pod-bay-doors'):
            return "I'm afraid I can't do that."
        return 'HAL 9000'
'{:open-the-pod-bay-doors}'.format(HAL9000())
```

### f-str
```
x=1; user={'name':'ahui'};
f'x={x}, user["name"]={user["name"]}, id(user)={id(user)}'

'x=1, user["name"]=ahui, id(user)=4500547264'
```

### string Template

    from string import Template
    Template('$who likes $what').substitute(who='Tom', what='cat')
    Template('$who').substitute({'who':1})

## encoding
python 内存中string 全部是以unicode编码的，而bytes则可以任何特定的编码

### unicode
以下三个等价, 且都是 class str

	>>> '中'
    >>> "\u4e2d"
	>>> u'中'
    >>> u"\u4e2d"
    '中'

    >>> ord('A')
    65
    >>> ord('中') \x4e2d = 20013
    20013
    >>> chr(66)
    'B'
    >>> chr(25991)
    '文'

### Bytes
bytes对象b'\xa420'只是一堆比特位而已。define bytes

    >>> '\x00\x01\x61'.encode()
    >>> bytes([0, 1, 0x61])
    b'\x00\x01a'

	>>> type(b'abc')
	<class 'bytes'>
	>>> type('abc')
	<class 'str'>

    >>> 'abc'[0]
    'a'
    >>> 'abc'.encode()[0]
    97

对于字符串操作，我们并不关心它们内部编码。除非需要字节包用于传输时
1. str 是无关编码的unicode: utf8,gbk, 是纯字符串表示: python3 采用unicode
1. bytes 是关编码: utf8,gbk, 是纯字符经过编码过后的二进制数据

#### string to bytes:
ascii 不变, 其它则用16进制表示

	'abc'.encode('ascii')
		b'abc'
	'中国'.encode() # 默认utf8
		b'\xe4\xb8\xad'
	'中国'.encode('GB2312')
		b'\xd6\xd0'

	b'\xd6\xd0'.decode('GB2312')
		'中'

detect bytes encoding:

    >>> chardet.detect('中国人民'.encode())
    {'encoding': 'utf-8', 'confidence': 0.938125, 'language': ''}

#### utf8声明源码
1. 第二行注释是为了告诉Python解释器，按照UTF-8编码读取源代码
2. 编辑器要without BOM

    #!/usr/bin/env python3
    # -*- coding: utf-8 -*-

### raw string
string literals: r'...', r'''...''', r"...", r"""...""" are all byte strings,

    >>> r'a\n\t'
    'a\\n\\t'
    >>> r'''start
     end'''

## len
python3:

	len('中'); 1
	len('中'.encode('utf8')); 3
	len('ab'); 2
	len('ab'.encode('utf8')); 2


## ord/chr

	ord('A'); 65
	chr(65); 'A'

## Access String
like list

	'hilojack'[4:]
	'hilo' in 'hilojack'

## string func

	## substring count
	'hilo hilo'.count('hilo')

	## capitalize
	'hilo'.capitalize()
	'hilo'.upper()
	'hilo'.lower()

	## len
	len('abc')

## split
'ab cd'.split(' ') # ['ab', 'cd']
'word1 word2 word3'.split(' ', 1) # 'word2 word3'

	## sorted
	sorted('a zx') # [' ', 'a', 'x', 'z']

## search replace

	'hilojack'.find('jack');//4
	'hilo' in 'hilojack'
	str.replace(needle, word, 1); //replace the first needle with word

### startwith
    aString.startswith("hello")

### endswith

    'abc'.endswith('bc')
    'abc'.endswith(('bd', 'bc')) # tuple only

### find and index
### rfind and rindex

    str.find(substr, start=0, end=len(string))
        Index if found and -1 otherwise.
         S.index(sub[, start[, end]]) -> int
         Like S.find() but raise ValueError when the substring is not found.

## pad- zfill

	$ print '1'.zfill(2);
	01

## repeat
```
	'pad'*2
	'.'.join(['pad']*2)
```

## trim
包括\n, ' ', '\t\r'

	'a\n  '.strip() + ',end'
    s.lstrip()

## Concat String

	>>> print 'a'+'b'+'c'	# with no space
	abc	# "abc\n"
	>>> print 'a' 'b' 	'c'   # with no space
	>>> print 'a''b''c'   # with no space
	abc # "abc\n"
	>>> print var1 var2   # syntax error
	>>> print '-'*6
	------
	>>> print '-' * 6
	------

long delimiter `"""` and `'''`(same): `\n` is still transfered by python

	print """
	a\nbc
	"""
	print """ab\nc"""
	print '''ab\nc'''

### Escape Sequences

	\n	ASCII linefeed (LF)
	\N{name}	Character named name in the Unicode database (Unicode only)
	\r ASCII	Carriage Return (CR)
	\uxxxx	Character with 16-bit hex value xxxx (Unicode only)
	\Uxxxxxxxx	Character with 32-bit hex value xxxxxxxx (Unicode only)
	\v	ASCII vertical tab (VT)
	\ooo	Character with octal value ooo
	\xhh	Character with hex value hh

#### not Escape
use repr encode

	>>> string = "abc\ndef"
	>>> print (repr(string))
	>>> 'abc\ndef'

	>>> print(repr("\n"))
	'\n'
	>>> print(r"\n")
    \n
    repr(obj, /)
        Return the canonical string representation of the object.

### print

	>>> print('a', 'b', 'c') # with space and new line
	a b c	# "a b c\n"
	>>> print('.', end='') # with space only, no new line

	# If you are having trouble with buffering, or with: sys.stdout.flush()
	>>> print('.', end='', flush=True) 

with no space and new line:

	>> sys.stdout.write('string')
	string

## string like list

	print "abc"[-1]
	for in "abc"

## hex

    >>> hex(16)
    '0x10'

## chunk
利用range+step:
```
    def ChunkStr1(string, length):
        return (string[0+i:length+i] for i in range(0, len(string), length))

    # 利用regex:
    import re
    def ChunkStr2(string, length):
    	return re.findall('.{1,'+str(length)+'}',string)

    list(ChunkStr1(s,10))
```
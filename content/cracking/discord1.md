+++
title = 'Discord Cracking Challenge (easy)'
date = 2023-09-24T18:19:45-04:00
tags = ["cracking", "easy"]
description = "Easy cracking challenge on a discord server i recently found."
draft = false
+++
This binary came from a discord server, I won't mention which exactly, so this won't be indexed.

The binary was really easy, we see a while loop and the xor key `\xda\xd0\x23\x73`.

I solved the challenge using binary ninja python api, took me about 10 seconds.
```python
>>> output = bv.read(0x140009000, 0x20)
>>> xor = Transform['XOR']
>>> xor.decode(output, {'key' : b'\xda\xd0\x23\x73'})
b'FLAG{ReversE_EngineerinG_IS_FuN}'
```


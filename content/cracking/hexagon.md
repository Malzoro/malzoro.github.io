+++
title = 'Hexagon'
date = 2023-09-23T16:26:10-04:00
tags = ["cracking", "easy"]
description = "Super easy C++ reverse engineering challenge, very good intro to reversing for newbies !"
draft = false
+++
Super easy reverse-engineering challenge, very good intro to reversing C++ binaries.

You'll learn how `std::cout`, `std::cin` works at a lower level.

For example it's the first time i notice that `std::cout` and `std::cin` are stored in the `.bss` segment.

{{< image src="/code.png" alt="Hello Friend" position="center" >}}

As you can see the challenge is relatively easy, we're checking that the input is between 7 and 11 bytes. Next we're doing the following checks.
- byte 0 needs to be J (R - 0x8)
- byte 2 needs to be 0
- byte 4 needs to be -
- byte 3 and 7 needs to be equal

If we match all these conditions then we won the challenge, we can come up with the answer.
```
JA0A-AAA
```

Testing this key, we successfully win the challenge.

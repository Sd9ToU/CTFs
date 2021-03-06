# Time's Up, Again!
> Previously you solved things fast. Now you've got to go faster. Much faster. Can you solve *this one* before time runs out? times-up-again, located in the directory at /problems/time-s-up--again-_2_55710a2388cfe35ec1afa8221b3f1ded.

## Summary
* signal
* /dev/urandom

## Analysis
``` python
from ctypes import *
from os import urandom
from sys import byteorder

libc = CDLL("libc.so.6")

guess = c_int()
result = c_int()

def init_randomness():
    libc.srand(libc.time(None))

def get_random():
    return c_int(int.from_bytes(urandom(8), byteorder))

def get_random_op():
    v2 = 2764075
    v0 = libc.rand()
    return c_uint(v2 >> (v0 - 3 * ((0xAAAAAAAAAAAAAAAB * v0 >> 64) >> 1)) * 8 & 0xff)

def do_op(a, b, c):
    if a == 43:
        return c_int(b + c)
    if a == 45:
        return c_int(b - c)
    if a != 42:
        exit(1)
    return c_int(c * b)

def maybe_decrease(n):
    return c_int(n) if libc.rand() % 50 <= 0 else c_int(n - 1)

def gen_expr(n):
    if n:
        v3 = maybe_decrease(n).value
        v4 = maybe_decrease(n).value
        v5 = get_random_op().value
        print('(', end='')
        v6 = gen_expr(v3).value
        print(" %c " % chr(v5), end='')
        v7 = gen_expr(v4).value
        print(')', end='')
        return do_op(v5, v6, v7)
    else:
        v1 = get_random().value
        print("(%d)" % v1, end='')
        return c_int(v1)

def generate_challenge():
    global result
    result = gen_expr(4)


init_randomness()
print("Challenge: ", end='')
generate_challenge()
print(chr(10), end='', flush=True)
print("Setting alarm...", flush=True)
libc.ualarm(200, 0)
print("Solution? ", end='')
libc.scanf(b"%d", pointer(guess))

if guess.value == result.value:
    print("Congrats! Here is the flag.txt!")
    libc.system("/bin/cat flag.txt")
else:
    print("Nope!")
```

## Solve
``` python
import subprocess

while True:
    p = subprocess.Popen("./times-up-again", stdin=subprocess.PIPE, stdout=subprocess.PIPE)
    expr = p.stdout.readline()[10:-1]
    try:
        p.stdin.write(str(eval(expr))+'\n')
    except IOError:
        continue
    flag = p.stdout.read()
    if "pico" in flag:
        print(flag)
        break
```
times-up-again은 챌린지를 시작하고 0.2초 후에 시그널을 발생시키기 때문에 성공 확률이 적다. pwntools는 실패하면 통신이 끊겨버리므로 사용하기 어렵다. 따라서 위와 같이 subprocess를 사용하되 성공할 때까지 수행하도록 하면 된다.  

flag: `picoCTF{Hasten. Hurry. Ferrociously Speedy. #3230cac7}`

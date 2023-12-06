官网地址：[GitHub - hellman/libnum: Working with numbers (primes, modular, etc.)](https://github.com/hellman/libnum)
### 一、Coverting
1. libnum.s2n(s)：字符串转换为数字。
```python
import libnum
s = "ab12"
print(libnum.s2n(s))
result: 1633825074
```
2. libnum.s2n(n)：数字转换为字符串。
```python
import libnum
n = 1633825074
print(libnum.n2s(n))
result: ab12
```
3. libnum.s2b(s)：字符串转换为二进制字符串。
```python
import libnum
s = "ab12"
print(libnum.s2b(s))
result: 01100001011000100011000100110010
```
4. libnum.b2s(b)：二进制字符串转换为字符串。
```python
import libnum
b = "01100001011000100011000100110010"
print(libnum.b2s(b))
result: ab12
```
### 二、Primes
1. libnum.primes(n)：返回不大于n的素数列表。
```python
import libnum
print(libnum.primes(19))
result: [2, 3, 5, 7, 11, 13, 17, 19]
```
2. libnum.generate_prime(n)：产生长度为n位的伪素数。
```python
import libnum
print(libnum.generate_prime(10))
result: 1021
```
### 三、其它
1. libnum.factorize(n)：返回n的所有素因子及每个素因子的个数。
```python
import libnum
print(libnum.factorize(60))
result: {2: 2, 3: 1, 5: 1}
```
2. libnum.modular.invmod(e,m)：返回e模m的逆元。
```python
import libnum
print(libnum.modular.invmod(5,24))
result: 5
```

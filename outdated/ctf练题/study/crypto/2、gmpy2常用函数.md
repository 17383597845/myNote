## gmpy2常见函数使用

### **1.初始化大整数**

```python
import gmpy2
gmpy2.mpz(909090)

result:mpz(909090)
```

### **2.求大整数a,b的最大公因数**

```python
import gmpy2
gmpy2.gcd(6,18)

result:mpz(6)
```

### **3.求大整数x模m的逆元y**

```python
import gmpy2
#4*6 ≡ 1 mod 23
gmpy2.invert(4,23)

result:mpz(6)
```

### **4.检验大整数是否为偶数**

```python
import gmpy2
gmpy2.is_even(6)

result:True

-----------
import gmpy2
gmpy2.is_even(7)

result:False
```

### **5.检验大整数是否为奇数**

```python
import gmpy2
gmpy2.is_odd(6)

result:False

-----------
import gmpy2
gmpy2.is_odd(7)

result:True
```

### **6.检验大整数是否为素数**

```python
import gmpy2
gmpy2.is_prime(5)

result:True
```

### **7.求大整数x开n次根**
```python
import gmpy2
gmpy2.iroot(81,2)

result:(mpz(9),True)
```

### **8.求大整数x的y次幂模m取余**
```python
import gmpy2
#2^4 mod 5 
gmpy2.powmod(2,4,15)

result:mpz(1)
```
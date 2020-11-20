# md5加密

by: 铁乐猫

date: 2020-11-21



例：加盐

```python
import hashlib

def md5hex(password,salt):
    """md5加密算法
    :param password:密码明文 salt: 加盐
    :return 32位hash值
    """
    password = password.encode('utf-8')
    salt = salt.encode('utf-8')
    md5 = hashlib.md5()
    md5.update(salt+password)
    return md5.hexdigest()

password = md5hex('1234567a','01a5eb')
print(password)
```


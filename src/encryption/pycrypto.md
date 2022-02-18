- [crypto](#crypto)
  - [老版本AES的ECB模型加密示例](#老版本aes的ecb模型加密示例)
  - [使用cryptography进行AES-256-ECB加密](#使用cryptography进行aes-256-ecb加密)

# crypto

> **Python 密码学工具包 (pycrypto)**
>
> pycrypto pypi: <https://pypi.org/project/pycrypto/>
>
> 这是安全散列函数（如 SHA256 和 RIPEMD160）和各种加密算法（AES、DES、RSA、ElGamal 等）的集合。
>
> > **注意**: 该软件不再维护。参考:  <https://www.pycrypto.org/>
> >
> >　PyCrypto 2.x 未维护、已过时且包含安全漏洞。
> >
> >　请选择以下选项之一：
> >
> > - Cryptography: <https://cryptography.io/>
> > - PyCryptodome: <https://www.pycryptodome.org/>

## 老版本AES的ECB模型加密示例

```python
import base64
from Crypto.Cipher import AES

class AesEncry(object):
    key = "U%56#o#u$jk0ffds".encode('utf-8')  # aes秘钥
    mode = AES.MODE_ECB
    cryptos = AES.new(key, mode)

    def decrypt(self, text):
        decode_base64 = base64.decodebytes(text.encode("utf-8"))
        plain_text = self.cryptos.decrypt(decode_base64)
        return self.uncode_chars(plain_text.decode("utf-8"))

    @staticmethod
    def uncode_chars(data):
        nCount = ord(data[-1])
        for i in data[-nCount:]:
            if ord(i) != nCount:
                return data
        return data[:-nCount]

    def encrypt(self, data):
        BLOCK_SIZE = 16
        pad = lambda s: (
            s
            + (BLOCK_SIZE - len(s) % BLOCK_SIZE)
            * chr(BLOCK_SIZE - len(s) % BLOCK_SIZE)
        )
        text = pad(str(data))
        cipher_text = self.cryptos.encrypt(text.encode("utf-8"))
        # 因为AES加密后的字符串不一定是ascii字符集的，输出保存可能存在问题，所以这里转为base64进制字符串
        return base64.encodebytes(cipher_text).decode("utf-8")

```

## 使用cryptography进行AES-256-ECB加密

原文: <https://gist.github.com/tcitry/df5ee377ad112d7637fe7b9211e6bc83>

```python
import base64
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.primitives import padding
from cryptography.hazmat.backends import default_backend
from django.utils.encoding import force_bytes, force_text

SECRET_KEY = "hellomotherfucker"
value = force_bytes("12345678901234567890")

backend = default_backend()
key = force_bytes(base64.urlsafe_b64encode(force_bytes(SECRET_KEY))[:32])


class Crypto:

    def __init__(self):
        self.encryptor = Cipher(algorithms.AES(key), modes.ECB(), backend).encryptor()
        self.decryptor = Cipher(algorithms.AES(key), modes.ECB(), backend).decryptor()

    def encrypt(self):
        padder = padding.PKCS7(algorithms.AES(key).block_size).padder()
        padded_data = padder.update(value) + padder.finalize()
        encrypted_text = self.encryptor.update(padded_data) + self.encryptor.finalize()
        return encrypted_text

    def decrypt(self, value):
        padder = padding.PKCS7(algorithms.AES(key).block_size).unpadder()
        decrypted_data = self.decryptor.update(value)
        unpadded = padder.update(decrypted_data) + padder.finalize()
        return unpadded


if __name__ == '__main__':
    print('>>>>>>>>>>>')
    crypto = Crypto()
    text = force_text(base64.urlsafe_b64encode(crypto.encrypt()))
    print(text)
    print('<<<<<<<<<<<<<')
    text = force_text(crypto.decrypt(base64.urlsafe_b64decode(text)))
    print(text)
```

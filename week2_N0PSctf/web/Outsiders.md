# Đề bài:
![Screenshot (512)](https://github.com/ductohno/ehc-adward/assets/152991010/1afcc8f1-b20f-487d-bac1-43369411895e)

# Giao diện trang web:
![Screenshot (513)](https://github.com/ductohno/ehc-adward/assets/152991010/67e2626e-4047-41f4-a4a2-86de147ed28a)

# Hint:
- Thật sự thì bài này mình không hiểu đầu bài muốn bảo làm gì mà thử referer không được, vậy nên mình đã đi xin hint

![Screenshot (514)](https://github.com/ductohno/ehc-adward/assets/152991010/2782deed-7244-4c87-9a3b-41b14dbe33fa)

- Vậy thì bài này sẽ phải sử dụng ```X-Forwarded-For: 127:0:0:1```

# Exploit:
- Bài có đúng 1 bước thôi, đó là thêm header ```X-Forwarded-For: 127.0.0.1```

![Screenshot (490)](https://github.com/ductohno/ehc-adward/assets/152991010/ef718ec2-d4de-4a04-b3c8-61456ea62be2)

# Flag:
N0PS{XF0rw4Rd3D}

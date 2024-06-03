# Đề bài:
![Screenshot (515)](https://github.com/ductohno/ehc-adward/assets/152991010/6a010c9e-ac76-4c35-97c2-e1ca3022b255)

# Note:
- Trước khi làm bài này, ta cần tải snicat

- Link github: ```https://github.com/CTFd/snicat```

- Download trên github:
  ```
  wget "https://github.com/CTFd/snicat/releases/latest/download/sc_`uname`_`uname -m`" -O sc
  chmod +x sc
  ```

- Truy cập: ./sc_link, ví dụ link bài là ```sc nopsctf-3e1c71d4b17b-jojo_chat_v1-0.chals.io```, ta nhập ```./sc nopsctf-3e1c71d4b17b-jojo_chat_v1-0.chals.io```

# Phân tích code:
- Code này sẽ ghi tên vô 1 file, tạo 1 file với tên đó, sau đó cho phép người dùng tùy ý viết vào file đó
```
log = open(f"./log/{name}", 'w')
```

- Hãy chú ý đoạn code này:
```
 option = input("\nChoose an option:\n1) See messages\n2) Send a message\n3) Logout\n")
        match option:
            case "1":
                print()
                messages = get_all_messages()
                for message in messages:
                    print(f"{message[0]} : {message[1][20:]}", end="")
            case "2":
                send_message(name)
            case "3":
                print("\nYou successfully logged out!")
                connected = False
            case "admin":
                if name == "admin":
                    admin()
```

- Khi ta nhập admin như 1 lựa chọn ngoài 3 lựa chọn trên, nó sẽ check xem name có phải là admin không, nếu phải thì nó sẽ admin(), có vẻ như là flag
- => Nhiệm vụ của chúng ta là phải đăng nhập với tên là admin

# Khai thác:
- Lỗ hổng của bài nằm ở đoạn code ```log = open(f"./log/{name}", 'w')```, nó là vuln python path travesal

- Bài này em sử dụng payload: ```../log/admin``` như là username

- Chi tiết:

![Screenshot (516)](https://github.com/ductohno/ehc-adward/assets/152991010/c95cf052-a4e2-4c4c-b2aa-eb62582d9a95)

# Flag:
N0PS{pY7h0n_p4Th_7r4v3r54l}


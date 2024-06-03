# Đề bài:
![Screenshot (506)](https://github.com/ductohno/ehc-adward/assets/152991010/710fafc6-9d84-43c8-9291-4816108b16a7)

## Giao diện trang web:
![Screenshot (507)](https://github.com/ductohno/ehc-adward/assets/152991010/c7c4f5a6-e4df-4bcb-af35-34a5f3e6d76d)

# Exploit:
- Trước tiên cứ thử nhập username vào, ta có:

![Screenshot (508)](https://github.com/ductohno/ehc-adward/assets/152991010/e9e011fa-9fb3-4125-9cd4-3a3b01233f77)

- Đọc hint, ta nhận thấy viêc mình cần làm là decode base64 của cookie, sửa isadmin=1, sau đó thay thế giá trị cookie cũ bằng cái mình vừa encode xong.

![Screenshot (509)](https://github.com/ductohno/ehc-adward/assets/152991010/049f951c-097a-4910-b8ee-1a4bd7489e59)

- Sửa isadmin=1

![Screenshot (489)](https://github.com/ductohno/ehc-adward/assets/152991010/d6ffa08b-9016-4932-baaa-af2a7a61f599)

- Thay thế giá trị cookie bằng cái vừa tìm dc:

![Screenshot (510)](https://github.com/ductohno/ehc-adward/assets/152991010/b36a2d79-a871-4c5a-862c-e2f3f1ad47f1)

- Kết quả:

![Screenshot (511)](https://github.com/ductohno/ehc-adward/assets/152991010/e991c223-3560-4fb9-beb0-7e11e4d38dbe)

# Flag:
 N0PS{y0u_Kn0W_H0w_t0_c00K_n0W}

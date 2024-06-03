# Đề bài:
![Screenshot (517)](https://github.com/ductohno/ehc-adward/assets/152991010/909bfb6e-8e3c-4b69-a67b-29778874a529)

# Giao diện trang web:
![Screenshot (518)](https://github.com/ductohno/ehc-adward/assets/152991010/d36dbacf-1277-459a-b124-6a67561fba11)

![Screenshot (519)](https://github.com/ductohno/ehc-adward/assets/152991010/b7f7cc90-077b-40bb-99ce-426b2acedac9)

# Tính năng trang web: Trang web bao gồm:
- Login
- Register
- Cập nhật thông tin
- Bảng id
- Logout
# Khai thác:
- Lỗ hổng của bài là Idor, nằm ở phần cập nhật thông tin:

![Screenshot (520)](https://github.com/ductohno/ehc-adward/assets/152991010/1b87b369-767c-4c19-aa37-0315354a61be)


- Ta không thể chỉnh username, nhưng trong burp suite ta có thể chỉnh cả username lẫn id

![Screenshot (521)](https://github.com/ductohno/ehc-adward/assets/152991010/69ff7400-3c43-4d19-81f9-b1f7c8e59c4d)

- Vậy sẽ ra sao nếu ta chỉnh id=1 và username= admin ( id mới là thứ quyết định quyền admin)

![Screenshot (522)](https://github.com/ductohno/ehc-adward/assets/152991010/76baf02b-3e40-4d8a-877e-247329723a8c)

- Hay rồi, giờ thì log out và đăng nhập bằng tài khoản admin nào, lúc này bạn sẽ thấy 1 mục tên admin, click vào thì chúng ta có:

![Screenshot (523)](https://github.com/ductohno/ehc-adward/assets/152991010/525a93fd-eff8-4304-8f9e-e90424ff4e7d)

# Flag:
N0PS{1d0R_p4zZw0rD_r3Z3t}

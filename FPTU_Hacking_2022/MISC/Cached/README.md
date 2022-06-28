# Cached
### Challenge
Just a song ? [image.zip](https://github.com/shine102/CTF-Writeups/files/8989829/image.zip)
### Writeup
Sau khi giải nén, thứ mà mình nhận được là một file image. Qua kiểm tra, mình biết được đây là một file Ext4
```
$ file image
image: Linux rev 1.0 ext4 filesystem data, UUID=7e5e258e-289e-4826-8f84-d716bc8de38e, volume name "cache" (needs journal recovery) (extents) (large files) (huge files)
```
Mình đã mount nó để kiểm tra trực tiếp
```
$ sudo mkdir /mnt/Cached
$ sudo mount image /mnt/Cached/
$ cd /mnt/Cached/
$ ls -l
total 36
drwx------. 2 t0t1ph4m t0t1ph4m  4096 Jun 27 15:00 backup
drwx------. 2 t0t1ph4m t0t1ph4m  4096 Jun  4 16:50 backup_stage
drwxr-xr-x  2 root     root      4096 Jun 25 15:44 image
drwxrwx---. 2 root     root     16384 Feb  3  2021 lost+found
drwxrwx---. 2 t0t1ph4m     2001  4096 Jun  4 16:50 recovery

```
Quá trình mount diễn ra một cách bình thường và suôn sẻ. Qua kiểm tra các directory bên trong, mình phát hiện chỉ duy nhất có một file âm thanh trong backup/. 
```
$ cd backup
$ ls -l
total 404
-rw-r--r--. 1 root root 413510 Jun  8 23:30 backup.1
$ file backup.1 
backup.1: MPEG ADTS, layer III, v1, 192 kbps, 48 kHz, JntStereo
```
Dường như chưa có gì bất bình thường khi nghe thử bài nhạc trên ngoại trừ việc nghe nó rất bắt tai.

Ban đầu, mình đã đoán có gì mờ ám trong file nhạc này và quyết định sử dụng binwalk để thử xuất ra các tệp được nhúng bên trong nó. Tuy nhiên kết quả trên không khả quan buộc mình đi đến quyết định sai lầm tiếp theo. Mình đã ngồi phân tích đề bài và cho rằng có lẽ mục tiêu cần tìm liên quan tới một file nào đó đã bị ẩn/xoá mất. Do vậy, mình đã quyết định sử dụng công cụ autopsy để thử khôi phục lại các file đã bị xoá. Kết quả trả về vẫn không có gì bất ngờ. Sau đó mình đã thử đem file âm thanh trên để đưa vào Hex Editor và nhận thấy có điều đáng chú ý. 

![image](https://user-images.githubusercontent.com/19572025/175980701-11e1fb1f-e717-456a-84e8-a276807a06db.png)

Có vẻ như đã có một gợi ý lớn ở đây, nếu đúng như dự đoán của mình cộng theo lời hint trong đề bài thì dường như có một file flag.txt đã được ém trong file nhạc này. Để lấy ra được nó khiến mình loay hoay hồi lâu và có vẻ như mình đã bỏ qua một chi tiết hết sức đáng ngờ. Bắt đầu từ địa chỉ `0x00000069`, mỗi byte được ngăn cách nhau bởi byte 00. Mình cũng biết đây là ý đồ của tác giả nên đã bắt đầu research từ nơi bắt đầu nó. Mình đã loại bỏ đi các byte gây rối kia và nhận được `50 4B 03 04 ` thì theo như [danh sách này](https://en.wikipedia.org/wiki/List_of_file_signatures), nó chính là file signature cho tệp zip. Vậy là mình đã xác định được đây là file zip có chứa một flag.txt bên trong và đã được mã hoá bằng password.

Và đúng như dự đoán:

![image](https://user-images.githubusercontent.com/19572025/175992192-0a0e0686-f0bf-4a8e-9e91-b0da1af4a36f.png)

Và vấn đề tiếp theo chúng ta cần làm là đi tìm ra được password. Tất nhiên mình cũng đã thử các password có khả năng như Cached (tên đề bài), song (just a song: chỉ là "song") hay toàn bộ chuỗi "just a song". Không có cái nào thành công và buộc mình phải đi bruteforce hoặc thử tìm kiếm lại đâu đó từ đầu để tìm ra manh mối cho password. Với bruteforce, `john-the-criper` đến với mình khá lâu, phải research một hồi thì mình mới nhớ đến nó. Quá trình cài đặt gặp nhiều vấn đề nhưng rất may là cũng đâu vào đó.

```
$ john-the-ripper.zip2john cached.zip > cached.hashes
$ john-the-ripper cached.hashes 
Warning: detected hash type "ZIP", but the string is also recognized as "ZIP-opencl"
Use the "--format=ZIP-opencl" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
No password hashes left to crack (see FAQ)
$ john-the-ripper --show cached.hashes 
cached.zip/flag.txt:infected:flag.txt:cached.zip:cached.zip

1 password hash cracked, 0 left
```
Kết quả bruteforce cho kết quả password cần tìm là `infected`.

Flag: `FPTUHacking{1265654407417514740_wouldn_t_you_say_there_s_a_light_in_the_darkest_moment}`

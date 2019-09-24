#### StringBufferedReader(Reader in)

한줄을 통채로 입력받는 방법으로 주로 쓰임

- readLine() -> 개행문자를 포함해 String값으로 한줄을 읽음
  - 비슷하게 Scanner.nextLine()도 String값과 개행문자를 같이 읽음
- read() -> int값으로 변형하여 읽어오는 방식
  - 텍스트파일의 1이라는 숫자를 read()를 통해 읽어오면 아스키코드 1로 인식되어 결국 49를 읽는것이됨
  - 이를 방지하려면 Integer.parseInt() 메소드 사용

#### StringTokenizer(String str, String delim)

지정된 구분자가 없을때 위 readLine()을 통해 입력받는것을 space(" ") 기준으로 쪼갬(Tokenizer)

- hasMoreTokens() -> 토큰 존재여부를 알려줌
- nextToken() -> 처음토큰부터 다음 토큰값을 받아옴


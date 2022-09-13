# [[Java] getS3Object 후 스트림을 닫아야하는 이유](https://velog.io/@jomminii/Java-getS3Object-%ED%9B%84-%EC%8A%A4%ED%8A%B8%EB%A6%BC%EC%9D%84-%EB%8B%AB%EC%95%84%EC%95%BC%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0)

`S3 object` 를 조회해오는 코드를 살펴보다가 낯선 코드가 보여서 한 번 살펴봤습니다.


```java
public S3Object getObject(String filePath) {
	...
     try (S3Object s3Object = amazonS3Client.getObject(bucket, filePath)) {
	...
    } catch (Exception e) {
    ...
    }
}

```

일단 `try() {}` 구문 자체도 눈에 익지 않아서 찾아보니 `try-with-resources-Statement` 라는 try 리소스 문 이더라고요.

Java 7 부터 추가된 문법인데 `()` 안에 `java.io.Closeable`을 구현하는 리소스들은 Statement 가 끝날 때 리소스가 닫히는걸 보장한다고 해요.

이전에 닫는걸 보장해줬던 것들은 `BufferedReader`나 DB 커넥션 같은 것들이었는데, S3Object 는 받아오면 끝일거 같은데 왜 닫아줘야하지? 라는 생각을 했는데요.

[Closeable S3Objects - AWS](https://aws.amazon.com/ko/blogs/developer/closeable-s3objects/)에 따르면 2013년부터 `S3Object` 클래스는 `Closeable` 인터페이스를 구현하고 있고, S3Object는 HTTP connection으로부터 데이터를 스트리밍할 수 있는 S3ObjectInputStream을 포함하고 있습니다.

`S3Object`를 조회하기 위해 `getObject` 를 호출하면 HTTP Connection이 계속해서 열려있고 대기중이므로 스트림을 빠르게 읽고 HTTP 연결이 제대로 해제될 수 있도록 스트림을 닫아야한다고 안내하고 있습니다.

그러므로 `getObject`를 한 후에도 스트림을 닫아줘야합니다.

닫아주는 방식은 위의 코드처럼 try 리소스 문을 활용해 저절로 닫히게 할 수도 있고, 아니면 `s3Object.close()`를 직접 호출해서 닫아줘도 됩니다.


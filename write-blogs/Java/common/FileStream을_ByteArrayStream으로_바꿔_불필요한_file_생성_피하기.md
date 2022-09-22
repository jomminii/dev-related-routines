# [[Java] FileStream 을 ByteArrayStream 으로 바꿔 불필요한 file 생성 피하기](https://velog.io/@jomminii/remove-unnecessary-io)

비즈니스 로직 중 가공한 데이터를 엑셀로 가공해서 S3로 올리는 로직이 있습니다. apache.poi 라이브러리 중 Workbook 을 활용하는건데요.

처음 로직을 작성할 때는 엑셀 저장을 위한 파일을 만들고, `FileOutputStream`을 통해 엑셀 내용을 담아 이 파일 자체를 S3에 업로드 하는 로직을 짰습니다.

<br>


```java
private <T> String dealWithExcel(List<ExcelData> excelData, HttpServletResponse response) {
    Date currentDate = new Date();

    String fileName = "fileName";
    File newExcelFile = new File(fileName);

    ExcelCreator<T> excelCreator = makeDataForExcel(excelData, currentDate);

    try (ExcelWorkbook excelWorkbook = excelCreator.createExcel();
        FileOutputStream fileOutputStream = new FileOutputStream(newExcelFile);
    ) {
        excelS3Upload(newExcelFile, excelWorkbook, fileOutputStream);

        return fileName;
    } catch (Exception e) {
        log.warn("dealWithExcel error : ", e);
    } finally {
        File f = new File(fileName);
        if (f.exists()) {
            f.delete();
        }
    }
    ...
}

private void excelS3Upload(File resultExcelFile, ExcelWorkbook excelWorkbook, FileOutputStream fileOutputStream)
    throws IOException {
    excelWorkbook.write(fileOutputStream);
    ObjectMetadata objectMetadata = getObjectMetadata();

    s3Adapter.putObject(filePath, resultExcelFile, objectMetadata);
}
```


처음 짰을 때는 처음`OutputStream` 개념을 사용한거라 뿌듯하기도 했는데, 이상하게 테스트나 api 를 호출해보면 디스크에 엑셀 파일이 생성되더라고요.

그래서 찾아보니 파일은 `new File` 로 파일을 만들었으니 파일은 무조건 생기는 것 같고 이걸 지워야겠다 생각해서 파일을 삭제하는 로직을 `finally` 에 추가했습니다.

<br>


```java
        File f = new File(fileName);
        if (f.exists()) {
            f.delete();
        }
```

그럼에도 뭔가 찝찝하긴 했는데 코드리뷰를 받고 나니 파일을 아예 생성하지 않고 엑셀을 업로드 할 수 있는 방법이 있는걸 알게 됐습니다.

S3에서 파일이 아닌 `InputStream`으로 변환해서 올려도 `put`이 가능하도록 관련 메소드를 제공하고 있었고, 최초에 `OutputStream`에 엑셀 내용을 받을 때도 파일이 아닌 `ByteArray` 를 통해 출력 내용을 담을 수 있었습니다.

그래서 파일을 생성하는 로직을 제거하고, `ByteArrayOutputStream`에 엑셀 데이터를 출력하도록 담고, 이 데이터를 `ByteArray`에 담은 다음, 이걸 읽는 `ByteArrayInputStream`을 생성했습니다.

마지막으로 이 `InputStream`을 s3 put object 메소드를 이용해 업로드해주었습니다.

<br>


```java
private <T> String dealWithExcel(List<ExcelData> excelData, HttpServletResponse response) {
    Date currentDate = new Date();
    String fileName = "fileName";

    ExcelCreator<T> excelCreator = makeDataForExcel(excelData, currentDate);

    try (ExcelWorkbook excelWorkbook = excelCreator.createExcel()
    ) {
        excelS3Upload(excelWorkbook);

        return fileName;
    } catch (Exception e) {
        log.warn("dealWithExcel error : ", e);
    }
    ...
}

private void excelS3Upload(ExcelWorkbook excelWorkbook)
    throws IOException {
        ByteArrayInputStream bis = null;
        try (
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ) {
            excelWorkbook.write(bos);
            byte[] excelBytes = bos.toByteArray();
            bis = new ByteArrayInputStream(excelBytes);

            ObjectMetadata objectMetadata = getObjectMetadata();
            s3Adapter.putObject(filePath, bis, objectMetadata);
        } finally {
            if (bis != null) {
                bis.close();
            }
        }
}
```

이렇게 로직을 바꾸니 파일을 굳이 생성하지 않아도 되서 불필요한 IO 가 생기지 않게 되었고, 파일 삭제 로직 또한 빼도 되었습니다.

`excelS3Upload`에 필요한 파라미터도 많이 줄어들어서 보기 더 깔끔해졌습니다.

이번 경험을 통해서 `Stream`을 활용하는 방법을 하나 더 알아가네요!


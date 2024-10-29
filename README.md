# 영화 평점 크롤링하기

xlsx 모듈을 사용하여 엑셀 파일을 읽고, 시트 데이터를 JSON 형태로 변환하여 각 URL 링크와 제목을 쉽게 접근할 수 있도록 설정했습니다.
```javascript
const xlsx = require('xlsx'); 
const add_to_sheet = require('./add_to_sheet');
const puppeteer = require('puppeteer');

// 기존 엑셀 파일을 불러오고, 데이터 시트를 JSON 형태로 변환
const workbook = xlsx.readFile('xlsx/data.xlsx');
const ws = workbook.Sheets.Sheet1;
const records = xlsx.utils.sheet_to_json(ws);
```




각 페이지에 접속하여 평점을 크롤링하고, 이를 엑셀 파일에 추가로 저장하는 작업입니다. page.evaluate 내에서 DOM을 통해 원하는 데이터를 추출하고, add_to_sheet 함수를 통해 실시간으로 엑셀 시트에 저장합니다.
```javascript
 const crawler = async () => {
    const browser = await puppeteer.launch({ headless: false });
    const page = await browser.newPage();
    await page.setUserAgent('Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36');

    // 시트에 '평점' 헤더 추가
    add_to_sheet(ws, 'C1', 's', '평점');

    for (const [i, r] of records.entries()) {
        await page.goto(r.링크);
        const text = await page.evaluate(() => {
            const score = document.querySelector('.info_group:nth-of-type(3) dd');
            return score ? score.textContent.trim() : null;
        });
 
        if (text) {
            console.log(r.제목, '평점', text);
            add_to_sheet(ws, `C${i + 2}`, 'n', parseFloat(text)); // 엑셀 파일에 평점 추가
        }
        await new Promise(resolve => setTimeout(resolve, 1000)); // 요청 간 지연
    }<br>
    await browser.close();
    xlsx.writeFile(workbook, 'xlsx/result.xlsx'); // 결과를 엑셀 파일로 저장
 };
```

 
전체 크롤링 프로세스에서 발생할 수 있는 오류를 캡처하고 로그로 남겨 문제 해결에 도움이 되도록 구성하였습니다
```javascript
  try {
     await crawler();
 } catch (e) {
     console.error('크롤링 중 에러 발생:', e);
 }
```


![image](https://github.com/user-attachments/assets/7b84a889-601f-4fb9-8267-089cccfc54a9)
결과물 입니다.




# 영화 평점 크롤링하기

> const xlsx = require('xlsx'); <br>
> const add_to_sheet = require('./add_to_sheet');<br>
> const puppeteer = require('puppeteer');<br>
><br>
> // 기존 엑셀 파일을 불러오고, 데이터 시트를 JSON 형태로 변환<br>
>  const workbook = xlsx.readFile('xlsx/data.xlsx');<br>
> const ws = workbook.Sheets.Sheet1;<br>
> const records = xlsx.utils.sheet_to_json(ws);<br>

 xlsx 모듈을 사용하여 엑셀 파일을 읽고, 시트 데이터를 JSON 형태로 변환하여 각 URL 링크와 제목을 쉽게 접근할 수 있도록 설정했습니다. <br>

> const crawler = async () => {<br>
>    const browser = await puppeteer.launch({ headless: false });<br>
>    const page = await browser.newPage();<br>
>    await page.setUserAgent('Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36');<br>
>
>    // 시트에 '평점' 헤더 추가<br>
>    add_to_sheet(ws, 'C1', 's', '평점');<br>
>
>    for (const [i, r] of records.entries()) {<br>
>        await page.goto(r.링크);<br>
>        const text = await page.evaluate(() => {<br>
>            const score = document.querySelector('.info_group:nth-of-type(3) dd');<br>
>            return score ? score.textContent.trim() : null;<br>
>        });<br>
>
>        if (text) {<br>
>            console.log(r.제목, '평점', text);<br>
>            add_to_sheet(ws, `C${i + 2}`, 'n', parseFloat(text)); // 엑셀 파일에 평점 추가<br>
>        }<br>
>        await new Promise(resolve => setTimeout(resolve, 1000)); // 요청 간 지연<br>
>    }<br>
>    await browser.close();<br>
>    xlsx.writeFile(workbook, 'xlsx/result.xlsx'); // 결과를 엑셀 파일로 저장<br>
> };<br>


 각 페이지에 접속하여 평점을 크롤링하고, 이를 엑셀 파일에 추가로 저장하는 작업입니다. page.evaluate 내에서 DOM을 통해 원하는 데이터를 추출하고, add_to_sheet 함수를 통해 실시간으로 엑셀 시트에 저장합니다. <br>


>  try {<br>
>     await crawler();<br>
> } catch (e) {<br>
>     console.error('크롤링 중 에러 발생:', e);<br>
> }<br>


 전체 크롤링 프로세스에서 발생할 수 있는 오류를 캡처하고 로그로 남겨 문제 해결에 도움이 되도록 구성하였습니다.

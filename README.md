# Python에서 AIOHTTP로 Webスクレイピング하기

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/) 

이 가이드는 Python에서 Webスクレイピング을 위해 AIOHTTP를 사용하는 기본 사항을 설명합니다.

- [AIOHTTP란?](#what-is-aiohttp)
- [AIOHTTP로 スクレイピング하기: 단계별 튜토리얼](#scraping-with-aiohttp-step-by-step-tutorial)
  - [Step #1: スクレイピング 프로젝트 설정](#step-1-setting-up-a-scraping-project)
  - [Step #2: スクレイピング 라이브러리 설정](#step-2-setting-up-the-scraping-libraries)
  - [Step #3: 대상 페이지의 HTML 가져오기](#step-3-getting-the-html-of-the-target-page)
  - [Step #4: HTML 파싱](#step-4-parsing-the-html)
  - [Step #5: 데이터 추출 로직 작성](#step-5-writing-the-data-extraction-logic)
  - [Step #6: スクレイピング된 데이터 내보내기](#step-6-exporting-the-scraped-data)
  - [Step #7: 전체 통합](#step-7-putting-it-all-together)
- [Webスクレイピング을 위한 AIOHTTP: 고급 기능 및 기법](#aiohttp-for-web-scraping-advanced-features-and-techniques)
  - [커스텀 ヘッダー 설정](#setting-custom-headers)
  - [커스텀 User Agent 설정](#setting-a-custom-user-agent)
  - [Cookie 설정](#setting-cookies)
  - [プロキシ 통합](#proxy-integration)
  - [오류 처리](#error-handling)
  - [실패한 リクエスト リトライ](#retrying-failed-requests)
- [Webスクレイピング을 위한 AIOHTTP vs Requests](#aiohttp-vs-requests-for-web-scraping)
- [결론](#conclusion)

## What Is AIOHTTP?

[AIOHTTP](https://docs.aiohttp.org/en/stable/)는 Python의 [`asyncio`](https://docs.python.org/3/library/asyncio.html) 라이브러리를 기반으로 구축된 비동기 클라이언트/서버 HTTP 프레임워크입니다. 기존 HTTP 클라이언트와 달리 AIOHTTP는 클라이언트 세션을 사용하여 여러 リクエスト에 걸친 연결을 관리하므로, 고 同時接続 및 セッション 기반 작업에 매우 효율적인 선택입니다.


**⚙️ Features**

- HTTP 프로토콜의 클라이언트와 서버 구현을 모두 지원합니다.  
- 클라이언트와 서버 모두에 대해 WebSockets를 네이티브로 지원합니다.  
- 웹 서버 구축을 위한 미들웨어 및 플러그인 가능한 라우팅을 제공합니다.  
- 대용량 데이터 스트리밍을 효율적으로 관리합니다.  
- 클라이언트 세션 지속성을 포함하여 연결 재사용을 가능하게 하고 여러 リクエスト에 대한 오버헤드를 최소화합니다.  


## Scraping with AIOHTTP: Step-By-Step Tutorial

Webスクレイピング 맥락에서 AIOHTTP는 페이지의 원시 HTML 콘텐츠를 가져오기 위한 HTTP 클라이언트에 불과합니다. 해당 HTML에서 데이터를 파싱하고 추출하려면 [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)와 같은 HTML 파서가 필요합니다.

> **Warning**:\
> AIOHTTP는 주로 프로세스의 초기 단계에서 활용되지만, 이 가이드는 전체 スクレイピング 워크플로를 안내합니다. 더 고급 AIOHTTP Webスクレイピング 기법을 찾고 계시다면 Step 3을 완료한 뒤 다음 장으로 건너뛰실 수 있습니다.

### Step #1: Setting Up a Scraping Project

Python3+를 설치하고 AIOHTTP スクレイピング 프로젝트를 위한 디렉터리를 생성합니다:

```bash
mkdir aiohttp-scraper
```

해당 디렉터리로 이동한 다음 [가상 환경](https://docs.python.org/3/library/venv.html)을 설정합니다:

```bash
cd aiohttp-scraper
python -m venv env
```

선호하는 Python IDE에서 프로젝트 폴더를 열고, 프로젝트 폴더 내에 `scraper.py`라는 파일을 생성합니다.


IDE의 터미널에서 가상 환경을 활성화합니다. Linux 또는 macOS에서는 다음을 사용합니다:

```bash
./env/bin/activate
```

Windows에서는 다음을 실행합니다:

```powershell
env/Scripts/activate
```

### Step #2: Setting Up the Scraping Libraries

AIOHTTP와 BeautifulSoup를 설치합니다:

```bash
pip install aiohttp beautifulsoup4
```

설치된 [`aiohttp`](https://docs.aiohttp.org/en/stable/) 및 [`beautifulsoup4`](https://pypi.org/project/beautifulsoup4/) 의존성을 `scraper.py` 스크립트로 가져옵니다:

```python
import asyncio
import aiohttp 
from bs4 import BeautifulSoup
```

> **Note**:\
> `aiohttp`는 작동을 위해 `asyncio`가 필요합니다.

이제 다음 `async` 함수 워크플로를 `scrper.py` 파일에 추가합니다:

```python
async def scrape_quotes():
    # Scraping logic...

# Run the asynchronous function
asyncio.run(scrape_quotes())
```

`scrape_quotes()`는 スクレイピング 로직이 블로킹 없이 同時接続으로 실행되는 비동기 함수를 정의합니다. 마지막으로 `asyncio.run(scrape_quotes())`가 비동기 함수를 시작하고 실행합니다.

### Step #3: Getting the HTML of the Target Page

이 예시는 [“Quotes to Scrape”](https://quotes.toscrape.com/) 사이트에서 데이터를 スクレイピング하는 방법을 설명합니다:

![The target site](https://github.com/luminati-io/aiohttp-web-scraping/blob/main/Images/s_C07E0B72CB9153F9B6E6EF6B76FDCD439C9910ACC1C4E94E70E103EE716CD2E2_1737465124750_image.png)

Requests 또는 AIOHTTP 같은 라이브러리를 사용하면 GET リクエスト를 수행하는 것만으로 페이지의 HTML 콘텐츠를 바로 가져올 수 있습니다. 그러나 AIOHTTP는 [다른 リクエスト 라이프사이클](https://docs.aiohttp.org/en/stable/http_request_lifecycle.html)로 동작합니다.  

AIOHTTP의 주요 구성 요소는 [`ClientSession`](https://docs.aiohttp.org/en/stable/client_reference.html)이며, 이는 연결 풀을 관리하고 기본적으로 [`Keep-Alive`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Keep-Alive)를 지원합니다. 각 リクエスト마다 새 연결을 여는 대신 기존 연결을 재사용하여 성능을 향상합니다.

リクエ스트를 수행하는 과정은 일반적으로 다음 세 가지 핵심 단계로 이루어집니다:

1. `ClientSession()`을 통해 セッション을 엽니다.
2. [`session.get()`](https://docs.aiohttp.org/en/stable/client_reference.html#aiohttp.ClientSession.get)으로 비동기적으로 GET リクエスト를 전송합니다.
3. `await response.text()` 같은 메서드로 レスポンス 데이터를 접근합니다.

이 설계는 작업 사이에 이벤트 루프가 블로킹 없이 서로 다른 [`with` 컨텍스트](https://docs.python.org/3/reference/datamodel.html#context-managers)를 사용할 수 있도록 하여, 고 同時接続 작업에 이상적입니다.

이를 염두에 두고, 다음 접근 방식으로 AIOHTTP를 사용해 홈페이지의 HTML을 가져올 수 있습니다:

```python
async with aiohttp.ClientSession() as session:
    async with session.get("http://quotes.toscrape.com") as response:
        # Access the HTML of the target page
        html = await response.text()
```

백그라운드에서 AIOHTTP는 서버로 リクエスト를 전송하고, 페이지의 HTML 콘텐츠를 포함한 서버의 レスポンス를 기다립니다. レスポンス를 받은 뒤 `await response.text()` 메서드는 HTML 콘텐츠를 문자열로 가져옵니다.

`html` 변수를 출력하면 다음과 같이 표시됩니다:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Quotes to Scrape</title>
    <link rel="stylesheet" href="/static/bootstrap.min.css">
    <link rel="stylesheet" href="/static/main.css">
</head>
<body>
    <!-- omitted for brevity... -->
</body>
</html>
```

### Step #4: Parsing the HTML

HTML 콘텐츠를 BeautifulSoup 생성자에 전달하여 파싱합니다:

```python
# Parse the HTML content using BeautifulSoup
soup = BeautifulSoup(html, "html.parser")
```

[`html.parser`](https://docs.python.org/3/library/html.parser.html)는 콘텐츠를 처리하기 위해 사용되는 기본 Python HTML 파서입니다.

`soup` 객체는 파싱된 HTML을 포함하며, 필요한 데이터를 추출하기 위한 메서드를 제공합니다.  

### Step #5: Writing the Data Extraction Logic

다음 코드는 페이지에서 인용구 데이터를 スクレイピング하는 데 사용할 수 있습니다:

```python
# Where to store the scraped data
quotes = []

# Extract all quotes from the page
quote_elements = soup.find_all("div", class_="quote")

# Loop through quotes and extract text, author, and tags
for quote_element in quote_elements:
    text = quote_element.find("span", class_="text").get_text().get_text().replace("“", "").replace("”", "")
    author = quote_element.find("small", class_="author")
    tags = [tag.get_text() for tag in quote_element.find_all("a", class_="tag")]

    # Store the scraped data
    quotes.append({
        "text": text,
        "author": author,
        "tags": tags
    })
```

이 코드 스니펫은 スクレイピング된 데이터를 저장하기 위해 `quotes`라는 리스트를 초기화합니다. 모든 인용구 HTML 요소를 찾아 순회하면서 인용문 텍스트, 저자, 태그 등의 세부 정보를 추출합니다. 추출된 각 인용구는 딕셔너리로 `quotes` 리스트에 저장되어, 데이터에 쉽게 접근하거나 내보내기 할 수 있도록 구성됩니다.

### Step #6: Exporting the Scraped Data

다음 코드를 사용하여 スクレイピング된 데이터를 CSV 파일로 내보낼 수 있습니다:

```python
# Open the file for export
with open("quotes.csv", mode="w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=["text", "author", "tags"])
    
    # Write the header row
    writer.writeheader()
    
    # Write the scraped quotes data
    writer.writerows(quotes)
```

위 스니펫은 `quotes.csv`라는 파일을 쓰기 모드로 엽니다. 그런 다음 열 헤더(`text`, `author`, `tags`)를 설정하고, 헤더를 기록한 뒤 `quotes` 리스트의 각 딕셔너리를 CSV 파일에 작성합니다.

[`csv.DictWriter`](https://docs.python.org/3/library/csv.html#csv.DictWriter)는 데이터 포맷팅을 단순화하여 구조화된 데이터를 더 쉽게 저장할 수 있도록 해줍니다. 이를 사용하려면 Python 표준 라이브러리에서 `csv`를 가져와야 합니다:

```python
import csv
```

### Step #7: Putting It All Together

다음은 완전한 AIOHTTP Webスクレイピング 스크립트입니다:

```python
import asyncio
import aiohttp
from bs4 import BeautifulSoup
import csv

# Define an asynchronous function to make the HTTP GET request
async def scrape_quotes():
    async with aiohttp.ClientSession() as session:
        async with session.get("http://quotes.toscrape.com") as response:
            # Access the HTML of the target page
            html = await response.text()

            # Parse the HTML content using BeautifulSoup
            soup = BeautifulSoup(html, "html.parser")

            # List to store the scraped data
            quotes = []

            # Extract all quotes from the page
            quote_elements = soup.find_all("div", class_="quote")

            # Loop through quotes and extract text, author, and tags
            for quote_element in quote_elements:
                text = quote_element.find("span", class_="text").get_text().replace("“", "").replace("”", "")
                author = quote_element.find("small", class_="author").get_text()
                tags = [tag.get_text() for tag in quote_element.find_all("a", class_="tag")]

                # Store the scraped data
                quotes.append({
                    "text": text,
                    "author": author,
                    "tags": tags
                })

            # Open the file name for export
            with open("quotes.csv", mode="w", newline="", encoding="utf-8") as file:
                writer = csv.DictWriter(file, fieldnames=["text", "author", "tags"])

                # Write the header row
                writer.writeheader()

                # Write the scraped quotes data
                writer.writerows(quotes)

# Run the asynchronous function
asyncio.run(scrape_quotes())
```

다음으로 실행할 수 있습니다:

```bash
python scraper.py
```

또는 Linux/macOS에서는 다음을 사용합니다:

```bash
python3 scraper.py
```

프로젝트의 루트 폴더에 `quotes.csv` 파일이 생성됩니다. 이를 열면 다음과 같이 표시됩니다:

![The final quotes file](https://github.com/luminati-io/aiohttp-web-scraping/blob/main/Images/s_C07E0B72CB9153F9B6E6EF6B76FDCD439C9910ACC1C4E94E70E103EE716CD2E2_1737466185816_image.png)

## AIOHTTP for Web Scraping: Advanced Features and Techniques

다음 예시에서는 대상 사이트로 [HTTPBin.io `/anything` endpoint](https://httpbin.io/anything)를 사용합니다. 이 API는 요청자가 전송한 IPアドレス, ヘッダー 및 기타 데이터를 반환합니다.

### Setting Custom Headers

AIOHTTP リクエスト에서 `headers` 인수를 사용해 [커스텀 ヘッダー를 지정](https://docs.aiohttp.org/en/stable/client_advanced.html#custom-request-headers)할 수 있습니다:

```python
import aiohttp
import asyncio

async def fetch_with_custom_headers():
    # Custom headers for the request
    headers = {
        "Accept": "application/json",
        "Accept-Language": "en-US,en;q=0.9,fr-FR;q=0.8,fr;q=0.7,es-US;q=0.6,es;q=0.5,it-IT;q=0.4,it;q=0.3"
    }

    async with aiohttp.ClientSession() as session:
        # Make a GET request with custom headers
        async with session.get("https://httpbin.io/anything", headers=headers) as response:
            data = await response.json()
            # Handle the response...
            print(data)

# Run the event loop
asyncio.run(fetch_with_custom_headers())
```

이렇게 하면 AIOHTTP가 [`Accept`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) 및 [`Accept-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language) ヘッダー가 설정된 GET HTTP リクエスト를 수행합니다.

### Setting a Custom User Agent

[`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent)는 [Webスクレイピング을 위한 가장 중요한 HTTP 헤더 중 하나](https://brightdata.co.kr/blog/web-data/http-headers-for-web-scraping)입니다. 기본적으로 AIOHTTP는 다음 `User-Agent`를 사용합니다:

```
Python/<PYTHON_VERSION> aiohttp/<AIOHTTP_VERSION>
```

위에서 언급한 기본 값은 요청이 자동화 스크립트에서 온 것임을 쉽게 식별 가능하게 하여, 대상 사이트에서 차단될 가능성을 높일 수 있습니다.

탐지될 확률을 줄이려면 이전과 같이 커스텀 실사용 `User-Agent`를 설정할 수 있습니다:

```python
import aiohttp
import asyncio

async def fetch_with_custom_user_agent():
    # Define a Chrome-like custom User-Agent
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/132.0.0.0 Safari/537.36"
    }

    async with aiohttp.ClientSession(headers=headers) as session:
        # Make a GET request with the custom User-Agent
        async with session.get("https://httpbin.io/anything") as response:
            data = await response.text()
            # Handle the response...
            print(data)

# Run the event loop
asyncio.run(fetch_with_custom_user_agent())
```

### Setting Cookies

HTTP ヘッダー와 마찬가지로 `ClientSession()`에서 `cookies`를 사용해 [커스텀 Cookie를 설정](https://docs.aiohttp.org/en/v3.7.3/client_advanced.html#custom-cookies)할 수 있습니다:

```python
import aiohttp
import asyncio

async def fetch_with_custom_cookies():
    # Define cookies as a dictionary
    cookies = {
        "session_id": "9412d7hdsa16hbda4347dagb",
        "user_preferences": "dark_mode=false"
    }

    async with aiohttp.ClientSession(cookies=cookies) as session:
        # Make a GET request with custom cookies
        async with session.get("https://httpbin.io/anything") as response:
            data = await response.text()
            # Handle the response...
            print(data)

# Run the event loop
asyncio.run(fetch_with_custom_cookies())
```

Cookie를 사용하면 Webスクレイピング リクエスト에 필수적인 セッション 데이터를 포함할 수 있습니다.

> **Note**:\
> `ClientSession`에 설정된 Cookie는 해당 セッション으로 이루어지는 모든 リクエスト에 공유됩니다. セッション Cookie에 접근하려면 [`ClientSession.cookie_jar`](https://docs.aiohttp.org/en/v3.7.3/client_reference.html#aiohttp.ClientSession.cookie_jar)를 참조하십시오.

### Proxy Integration

AIOHTTP에서는 IP 차단 위험을 줄이기 위해 プロキシ 서버를 통해 リクエスト를 라우팅할 수 있습니다. 이를 위해 `session`의 HTTP 메서드 함수에서 [`proxy` 인수](https://docs.aiohttp.org/en/v3.7.3/client_advanced.html#proxy-support)를 사용합니다:

```python
import aiohttp
import asyncio

async def fetch_through_proxy():
    # Replace with the URL of your proxy server
    proxy_url = "<YOUR_PROXY_URL>"

    async with aiohttp.ClientSession() as session:
        # Make a GET request through the proxy server
        async with session.get("https://httpbin.io/anything", proxy=proxy_url) as response:
            data = await response.text()
            # Handle the response...
            print(data)

# Run the event loop
asyncio.run(fetch_through_proxy())
```

### Error Handling

기본적으로 AIOHTTP는 연결 또는 네트워크 이슈에 대해서만 오류를 발생시킵니다. [`4xx`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#client_error_responses) 및 [`5xx`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#server_error_responses) 상태 코드의 HTTP レスポンス에 대해 예외를 발생시키려면 다음 접근 방식 중 하나를 사용할 수 있습니다:

1. **`ClientSession` 생성 시 `raise_for_status=True` 설정**: レスポンス 상태가 `4xx` 또는 `5xx`일 경우, 해당 セッション을 통해 수행되는 모든 リクエスト에 대해 자동으로 예외를 발생시킵니다.
2. **リクエスト 메서드에 `raise_for_status=True`를 직접 전달**: 다른 요청에 영향을 주지 않고, 개별 リクエスト 메서드(`session.get()` 또는 `session.post()` 등)에 대해서만 오류 발생을 활성화합니다.
3. **`response.raise_for_status()`를 수동으로 호출**: 예외를 발생시키는 시점을 완전히 제어할 수 있어, リクエ스트별로 처리 여부를 결정할 수 있습니다.

옵션 #1 예시:

```python
import aiohttp
import asyncio

async def fetch_with_session_error_handling():
    async with aiohttp.ClientSession(raise_for_status=True) as session:
        try:
            async with session.get("https://httpbin.io/anything") as response:
                # No need to call response.raise_for_status(), as it is automatic
                data = await response.text()
                print(data)
        except aiohttp.ClientResponseError as e:
            print(f"HTTP error occurred: {e.status} - {e.message}")
        except aiohttp.ClientError as e:
            print(f"Request error occurred: {e}")

# Run the event loop
asyncio.run(fetch_with_session_error_handling())
```

セッション 레벨에서 `raise_for_status=True`가 설정되면, 해당 セッション을 통해 수행되는 모든 リクエスト는 `4xx` 또는 `5xx` レスポンス에 대해 `aiohttp.ClientResponseError`를 발생시킵니다.

옵션 #2 예시:

```python
import aiohttp
import asyncio

async def fetch_with_raise_for_status():
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get("https://httpbin.io/anything", raise_for_status=True) as response:
                # No need to manually call response.raise_for_status(), it is automatic
                data = await response.text()
                print(data)
        except aiohttp.ClientResponseError as e:
            print(f"HTTP error occurred: {e.status} - {e.message}")
        except aiohttp.ClientError as e:
            print(f"Request error occurred: {e}")

# Run the event loop
asyncio.run(fetch_with_raise_for_status())
```

이 경우 `raise_for_status=True` 인수가 `session.get()` 호출에 직접 전달됩니다. 이를 통해 `4xx` 또는 `5xx` 상태 코드에 대해 자동으로 예외가 발생합니다.

옵션 #3 예시:

```python
import aiohttp
import asyncio

async def fetch_with_manual_error_handling():
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get("https://httpbin.io/anything") as response:
                response.raise_for_status()  # Manually raises error for 4xx/5xx
                data = await response.text()
                print(data)
        except aiohttp.ClientResponseError as e:
            print(f"HTTP error occurred: {e.status} - {e.message}")
        except aiohttp.ClientError as e:
            print(f"Request error occurred: {e}")

# Run the event loop
asyncio.run(fetch_with_manual_error_handling())
```

개별 リクエスト에 대해 더 큰 제어를 원한다면, リクエスト를 수행한 후 `response.raise_for_status()`를 수동으로 호출할 수 있습니다. 이 접근 방식은 오류를 처리할 정확한 시점을 결정할 수 있게 해줍니다.  


### Retrying Failed Requests

AIOHTTP는 リクエスト 자동 リトライ에 대한 내장 지원을 제공하지 않습니다. 이를 구현하려면 커스텀 로직을 사용하거나 [`aiohttp-retry`](https://github.com/inyutin/aiohttp_retry) 같은 서드파티 라이브러리를 사용해야 합니다. 이를 통해 실패한 リクエスト에 대한 リトライ 로직을 구성할 수 있으며, 일시적인 네트워크 이슈, タイムアウト 또는 レート制限을 처리하는 데 도움이 됩니다.

[`aiohttp-retry`](https://pypi.org/project/aiohttp-retry/)를 설치합니다:

```bash
pip install aiohttp-retry
```

코드에서 사용하는 방법은 다음과 같습니다:

```python
import asyncio
from aiohttp_retry import RetryClient, ExponentialRetry

async def main():
    retry_options = ExponentialRetry(attempts=1)
    retry_client = RetryClient(raise_for_status=False, retry_options=retry_options)
    async with retry_client.get("https://httpbin.io/anything") as response:
        print(response.status)
        
    await retry_client.close()
```

이는 지수 백오프 전략으로 リトライ 동작을 구성합니다. 자세한 내용은 [공식 문서](https://github.com/inyutin/aiohttp_retry?tab=readme-ov-file#documentation)에서 확인하십시오.

## AIOHTTP vs Requests for Web Scraping

아래는 AIOHTTP와 [Webスクレイピング을 위한 Requests](https://brightdata.co.kr/blog/web-data/python-requests-guide)를 비교한 요약 표입니다:

| **Feature** | **AIOHTTP** | **Requests** |
| --- | --- | --- |
| **GitHub stars** | 15.3k | 52.4k |
| **Client support** | ✔️  | ✔️  |
| **Sync support** | ❌   | ✔️  |
| **Async support** | ✔️  | ❌   |
| **Server support** | ✔️  | ❌   |
| **Connection pooling** | ✔️  | ✔️  |
| **HTTP/2 support** | ❌   | ❌   |
| **User-agent customization** | ✔️  | ✔️  |
| **Proxy support** | ✔️  | ✔️  |
| **Cookie handling** | ✔️  | ✔️  |
| **Retry mechanism** | Available only via a third-party library | Available via `HTTPAdapter`s |
| **Performance** | High | Medium |
| **Community support and popularity** | Medium | Large |

완전한 비교를 위해서는 [Requests vs HTTPX vs AIOHTTP](https://brightdata.co.kr/blog/web-data/requests-vs-httpx-vs-aiohttp)에 대한 블로그 게시글을 확인해 보십시오.

## Conclusion

AIOHTTP는 온라인 데이터 수집을 위한 HTTP リクエスト를 빠르고 안정적으로 수행하는 도구입니다. 하지만 자동화된 HTTP リクエスト는 공용 IPアドレス를 노출할 수 있습니다. 개인정보와 보안을 보호하기 위해, IPアドレス를 마스킹할 수 있는 Bright Data의 プロキシ 서버 사용을 고려해 보십시오.

- [Datacenter proxies](https://brightdata.co.kr/proxy-types/datacenter-proxies) – 770,000개 이상의 データセンタープロキシ IP.
- [Residential proxies](https://brightdata.co.kr/proxy-types/residential-proxies) – 195개 이상의 국가에서 7,200만 개 이상의 レジデンシャルプロキシ IP.
- [ISP proxies](https://brightdata.co.kr/proxy-types/isp-proxies) – 700,000개 이상의 ISPプロキシ IP.
- [Mobile proxies](https://brightdata.co.kr/proxy-types/mobile-proxies) – 700만 개 이상의 モバイルプロキシ IP.

오늘 무료 Bright Data 계정을 생성하여 당사의 プロキシ 및 スクレイピング 솔루션을 테스트해 보십시오!
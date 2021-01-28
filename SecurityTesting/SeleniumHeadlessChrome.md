# RUNNING SELENIUM WITH HEADLESS CHROME

> https://intoli.com/blog/running-selenium-with-headless-chrome/ 기사를 요약함
>
> 2017년 기사 



## 배경

- 구글이 웹 크롤링을 할 때 크롬의 headless 버전을 사용한다는 오래된 루머가 있었고, 이를 공개한다고 한다. 
- 이러한 이야기가 headless를 사용하지 않았던 사람은 충격적이지 않겠지만 놀라운 일이다. 
- PhantomJS도 headless 브라우징을 제공하고 있지만.. 언제까지 개발이 유지 될 지 예측하기 어렵다. 그런 면에서 구글 headless 지원은 기대되는 소식이라고 볼 수 있다.
- 이 글에는 headless 크롬을 기반으로 selenium을 사용하는 방법에 대해서 다룬다.



## 설치

### Chrome 설치 

- headless 기능을 제공하는 크롬을 설치해야 한다 (59 버전 이상 설치)
- 크롬이 설치되어 있지 않다면 크롬 다운로드 페이지에서 최신 버전을 다운로드 한다. 



### 가상환경 설정 및 selenium 설치

~~~
$ mkdir [project_dir]
$ cd [project_dir]
$ virtualenv evn
$ . env/bin/activate
$ pip install selenium

~~~



### chromedriver 설치 

- 맥OS의 경우 brew 명령어를 이용하여 설치 
- 그 외의 OS의 경우 [ChromeDriver- Web Driver for Chrome](https://sites.google.com/a/chromium.org/chromedriver/home)

~~~
brew install chromedriver
~~~



## 설정

- 안정 버전이 아닌 크롬을 설치 할 경우 크롬의 실행 파일 위치가 설정되지 않는 경우가 있는데 리눅스의 경우 `/usr/bin/google-chrome-unstable` 에 위치하며 맥OS의 경우 `/Applications/Google Chrome 2.app/Contents/MacOS/Google Chrome` 에 위치한다. 
- 이러한 크롬 바이너리의 위치를 python 코딩 시 `ChromeOption` 오브젝트의 `binary_location` 에 경로 설정을 해줘야 한다.

~~~python
from selenium import webdriver

options = webdriver.ChromeOptions()

options.binary_location = '/usr/bin/google-chrome-unstable'
~~~



- 우리는 headless 모드의 크롬을 사용하길 원하기 때문에 `ChromeOption` 오브젝트의 add_argument 메소드를 이용하여 `headless` 를 설정하거나 혹은 chrome 커맨드 실행 시 `--headless`  옵션을 추가해야 한다. 

  ~~~python
  options.add_argument('headless')
  ~~~

  

- 이외에 추가적인 옵션을 `add_argument` 함수를 이용하여 추가 할 수 있으며, 설정한 options(ChromeOptions 오브젝트) 을 파라미터로 입력하여 드라이버를 초기화 한다. 

  ~~~python
  #set the window size
  options.add_argument('window-size=1200x600')
  # initialize the driver
  driver.webdriver.Chrome(chrome_options=options)
  ~~~

  

## 네이버와 연결하기

- 위의 코드를 추가함으로서 headless 크롬 인스턴스에 웹 드라이버를 연결하였다. 이 연결에 대해 Selenium API를 통해 테스트, 웹사이트 스크랩 등 여러가지 작업을 수행할 수 있다. 

- 대상 사이트에 연결하기 위해서는 다음 함수를 사용

  ~~~python
  driver.get('https://naver.com')
  
  #웹 페이지의 엘리먼트들을 사용할 수 있는 시점(로드 완료) 할 때까지 10초 기다림 
  driver.implicitly_wait(10)
  
  
  email = driver.find_element_by_css_selector('input[type=email]')
  password = driver.find_element_by_css_selector('input[type=password]')
  login = driver.find_element_by_css_selector('input[value="Log In"]')
  email.send_keys('abc@naver.com')
  password.send_keys('1234')
  
  login.click()
  
  driver.get_screenshot_as_file('main-page.png')
                                              
                                            
  ~~~

  

- 위와 같이 webdriver의 여러 함수를 이용하여 대상 웹 페이지의 여러 요소들을 추출할 수 있으며 
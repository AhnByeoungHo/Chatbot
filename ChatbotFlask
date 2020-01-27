from flask import Flask, request
import pprint
import requests

app = Flask(__name__)
buylist = {}
book_list = []
movie = ''

@app.route('/action', methods=['GET', 'POST','POST','POST', 'POST'])
def process_webhook():
    request_json = request.json
    # pprint.pprint(request_json)
    if request_json['queryResult']['intent']['displayName'] == 'movie_search':
        return movie_search(request_json)
    elif request_json['queryResult']['intent']['displayName'] == 'movie_search_buy':
        return movie_search_buy(request_json)
    elif request_json['queryResult']['intent']['displayName'] == 'buy_search':
        return buy_search(request_json)
    elif request_json['queryResult']['intent']['displayName'] == 'movie_search_select':
        return movie_search_select(request_json)

def movie_search(request_json):
    # naver-book 인텐트 중 book 파라미터 데이터 추출
    query = request_json["queryResult"]["parameters"]["movie"]
    global book_list
    global movie
    movie = ''
    book_list = []
    # 인증 정보
    client_id = "You should write own id"
    client_secret = "You should write own key"

    # 기본 url 정보
    url = "https://openapi.naver.com/v1/search/movie.json"

    # url 호출 시 전달할 요청 변수 정보
    params = {"query": query,
              "display": 5,
              "sort": "count"}

    # requests 라이브러리를 이용한 책 검색 api 호출
    # get 방식으로 호출(url)/ 요청 변수 전달(params)/ 인증 정보 및 인코딩 정보 전달(header)
    response = requests.get(url=url, params=params,
                            headers={"X-Naver-Client-Id": client_id,
                                     "X-Naver-Client-Secret": client_secret,
                                     "Content-Type": "application/json; charset=utf-8"})
    # 호출 처리 상태 정보 recode 변수에 할당
    rescode = response.status_code

    if (rescode == 200):
        # 호출 처리 상태가 정상(200) 일 경우리턴 받은 책 조회 정보 출력
        pprint.pprint(response.json())
        data = response.json()
    else:
        print("Error Code:", rescode)

    # Naver 책 검색 API 응답 중 실제 책 아이템 데이터 추출 및 출력
    item_list = data["items"]
    # pprint.pprint(item_list)

    # Dialogflow로 응답되는 최종 문자열 데이터 구성
    text = ''
    i=1
    for item in item_list:
        title = str(item["title"])
        title = title.replace('<b>', '')
        title = title.replace('</b>', '')
        book_list.append(title)
        text += str(i) + '. ' + title + "\n"
        i = i + 1

    # Dialogflow로 응답되는 최종 데이터 확인
    print(book_list)
    if len(book_list) == 0:
        return {"fulfillmentText" : "검색하신 영화가 존재하지 않아 처음으로 돌아갑니다."}
    return {"fulfillmentText" : text + "영화 번호를 선택해주세요."}

def movie_search_buy(request_json):
    pprint.pprint(request_json)
    name = request_json['queryResult']['outputContexts'][1]['parameters']['name']
    global movie
    global buylist
    global book_list

    if name in buylist:
        if movie in buylist[name]:
            return {"fulfillmentText": "이미 구매하신 영화입니다."}
        buylist[name].append(movie)
    else:
        buylist[name] = []
        buylist[name].append(movie)

    print(buylist)

    text = movie
    book_list = []
    movie = ''
    return {"fulfillmentText": text + "구매 완료했습니다."}
def buy_search(request_json):
    name = request_json['queryResult']['outputContexts'][0]['parameters']['name']
    if name in buylist:
        print(buylist[name])
        total =""
        i = 1
        for list in buylist[name]:
            total += str(i) + ". " + list + "\n"
            i += 1
        total = total.rstrip()
        return {"fulfillmentText": "구매 내역은 다음과 같습니다\n" + total}
    else:
        return {"fulfillmentText": "구매 내역이 없습니다."}

def movie_search_select(request_json):
    number = request_json['queryResult']['outputContexts'][0]['parameters']['number']
    global movie
    print(int(number[0]) > len(book_list))
    if(int(number[0]) > len(book_list)):
        return {"fulfillmentText": "잘못된 숫자를 입력하셔서 처음으로 돌아갑니다."}
    movie = book_list[int(number[0])-1]
    return {"fulfillmentText": movie + " 구매하시겠습니까?"}

if __name__ == '__main__':
    app.run(debug=True)

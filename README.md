# Video search engine
Поисковый движок коротких видеоклипов (до 1 минуты) нацеленный на русскоязычную аудиторию

# Установка
1) Склонируйте репозиторий, перейдите в папку и запустите скрипт настройки install.sh. После этого активируйте виртуальное окружение:
  ```git clone 
     cd ./video_search_engine/
     bash ./install.sh
     source .venv/bin/activate
  ```

2) Перед первым запуском необходимо скачать веса модели [GigaAM-RNNT](https://github.com/salute-developers/GigaAM):
   ```
   cd nn_utils
   bash downloader.sh
   ```
# Запуск
1) Запуск бекенд-сервера с тяжёлыми нейросетями:
   ```
   cd nn_utils
   python3 app.py
   ```
2) Запуск фронтенд-сервера непосредственно с поиском и индексацией видео:
   ```
   cd frontend
   python3 app.py
   ```
# Использование
User-friendly: открыть страницу <http://127.0.0.1:6001/>.
На данный момент решение развёрнуто по адресу <http://85.113.39.151/yappy_search/>.
## Использование API:
### Поиск ранее загруженных видео:
Python:
```
import requests,json

query = "котики" # string, your search request
k = 10 # int, number of results

result = requests.get("http://85.113.39.151/yappy_search/searchVideo",
                   params=[
                                ("query",query),
                                ("k",k)
                          ])
print(result.status_code)
result = result.json()['result']
for video in result:
    description = video['description']
    url = video['url']
    print(url,description) # print link and video description
```
Аналогичный запрос curl:
```
curl "http://85.113.39.151/yappy_search/searchVideo?k=10&query=котики"
```
### Индексация нового видео:
Python:
```
import requests,json

your_url = "https://cdn-st.rutubelist.ru/media/f4/8d/0766c7c04bb1abb8bf57a83fa4e8/fhd.mp4" # string, url for mp4 video
your_description = "#технологии #девайсы #technologies #гаджеты #смартчасы #умныечасы #миф" # string, description (optional)

result = requests.post("http://85.113.39.151/yappy_search/addVideoHandle",
                   params=[
                                ("url",your_url),
                                ("description",your_description)
                          ]) # It tooks about 25 seconds depends on video duration
print(result.status_code)
result = result.json()
print(result['result']) # ok or fail
if 'error' in result:
    print("Error:",result['error'])
    # It could be raised by connection error and etc...
    # Note: it raises error if url already exist in db. Use parameter force=1 for indexing
else:
    print("Indexing time:",result['time']) # Time (seconds)
    print("Size of DB:",result['storelen']) # Count of all videos which has been indexed
```
### Описание передаваемых параметров
1) На ручку поиска searchVideo
```
    query -- текст поискового запроса
    k -- число видео в выдаче (10 по умолчанию)
```
2) На ручку добавления в индекс addVideoHandle
```
    url -- прямая ссылка на mp4-файл видео
    description -- текстовое описание видео (пустая строка по умолчанию)
    stride -- частота семплирования кадров (5 по умолчанию)
    K -- количество кадров, сохраняемых в индекс (5 по умолчанию)
    force -- принудительная индексация ранее проиндексированного видео (False по умолчанию)
```
### Описание ошибок
```
200 -- OK
504 -- Сервер упал, просим связаться с администратором
500 -- В запросе переданы некорректные параметры
```
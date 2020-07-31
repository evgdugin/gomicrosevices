# Первый микросервис
Попробуем написать наш первый микросервис. Выше мы определили, что ядро нашего приложения — это сервис-хранилище фильмов. С него и начнём.
Пока сузим требования к этому сервису до одного действия — показать список всех имеющихся фильмов. Начнём с функции main, создадим файл main.go и напишем в нём:

```s
package main

import (
	"encoding/json"
	"log"
	"net/http"
)

const Addr = ":8081"

func main() {
	http.HandleFunc("/movies", movieListHandler)
	log.Printf("Starting on port %s", Addr)
	log.Fatal(http.ListenAndServe(Addr, nil))
}
```
Регистрируем по URL /movies обработчик, который выдаст список. Затем пишем в лог, что сервис запускается, заодно указываем, где именно. Ну и запускаем сам сервис.
Прежде чем написать обработчик, надо определиться со структурой, которую мы будем использовать для представления фильма. Очевидно, у нас должно быть имя фильма, ещё обычно удобен числовой идентификатор. У фильма должен быть адрес, по которому его можно посмотреть, для отображения в каталоге нужен URL с постером, и, так как у нас платный сервис, мы должны хранить признак, платный ли это фильм. Напишем для этого структуру и обработчик:
```s
type Movie struct {
	ID       int    `json:"id"`
	Name     string `json:"name"`
	Poster   string `json:"poster"`
	MovieUrl string `json:"movie_url"`
	IsPaid   bool   `json:"is_paid"`
}

func movieListHandler(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json; charset=utf-8")
	mm := []Movie{
		Movie{0, "Бойцовский клуб", "/static/posters/fightclub.jpg", "https://youtu.be/qtRKdVHc-cE", true},
		Movie{1, "Крестный отец", "/static/posters/father.jpg", "https://youtu.be/ar1SHxgeZUc", false},
		Movie{2, "Криминальное чтиво", "/static/posters/pulpfiction.jpg", "https://youtu.be/s7EdQ4FqbhY", true},
	}
	err := json.NewEncoder(w).Encode(mm)
	if err != nil {
		log.Printf("Render response error: %v", err)
		w.WriteHeader(http.StatusInternalServerError)
	}
	return
}

```

# Тестовое задание на стажировку AppSecCloudCamp

На выполнение тестового задания у вас есть 7 дней. 
Выполненное задание следует опубликовать на GitHub и отправить на почту **appseccloudcamp@cloud.ru**. Практическое задание состоит из нескольких частей, при этом, даже если вы сделали только одну часть задания - присылайте её на рассмотрение. 
**ВАЖНО** При отправке тестового задания на почту обязательно указывайте ваши фамилию, имя, контактный номер телефон, telegram (при его наличии)

## 1. Вопросы для разогрева

1. Расскажите, с какими задачами в направлении безопасной разработки вы сталкивались? 
2. Если вам приходилось проводить security code review или моделирование угроз, расскажите, как это было? 
3. Если у вас был опыт поиска уязвимостей, расскажите, как это было? 
4. Почему вы хотите участвовать в стажировке?

---

## 2. Security code review

### Часть 1. Security code review: GO

Требуется провести анализ кода на GO с точки зрения безопасности и подготовить отчет по следующим пунктам:
 - Какие уязвимости присутствуют в этом фрагменте кода?
 - Указать строки, в которых присутствуют уязвимости.
 - К каким последствиям может привести эксплуатация найденных уязвимостей злоумышленником?
 - Описать способы исправления уязвимостей.
 - Если уязвимость можно исправить несколькими способами, необходимо перечислить их, выбрать лучший по вашему мнению и аргументировать свой выбор.

```
package main

import (
    "database/sql"
    "fmt"
    "log"
    "net/http"
    "github.com/go-sql-driver/mysql"
)

var db *sql.DB
var err error

func initDB() {
    db, err = sql.Open("mysql", "user:password@/dbname")
    if err != nil {
        log.Fatal(err)
    }

err = db.Ping()
if err != nil {
    log.Fatal(err)
    }
}

func searchHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != "GET" {
        http.Error(w, "Method is not supported.", http.StatusNotFound)
        return
    }

searchQuery := r.URL.Query().Get("query")
if searchQuery == "" {
    http.Error(w, "Query parameter is missing", http.StatusBadRequest)
    return
}

query := fmt.Sprintf("SELECT * FROM products WHERE name LIKE '%%%s%%'", searchQuery)
rows, err := db.Query(query)
if err != nil {
    http.Error(w, "Query failed", http.StatusInternalServerError)
    log.Println(err)
    return
}
defer rows.Close()

var products []string
for rows.Next() {
    var name string
    err := rows.Scan(&name)
    if err != nil {
        log.Fatal(err)
    }
    products = append(products, name)
}

fmt.Fprintf(w, "Found products: %v\n", products)
}

func main() {
    initDB()
    defer db.Close()

http.HandleFunc("/search", searchHandler)
fmt.Println("Server is running")
log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### Часть 2: Security code review: Python

Требуется определить тип уязвимости в примерах кода на Python и ответить на следующие вопросы:
 - Указать строки, в которых присутствуют уязвимости.
 - К каким последствиям может привести эксплуатация данных уязвимостей злоумышленником?
 - Описать способы исправления уязвимостей.
 - Если уязвимость можно исправить несколькими способами, необходимо перечислить их, выбрать лучший по вашему мнению и аргументировать свой выбор.

**Пример №2.1**
```
from flask import Flask, request
from jinja2 import Template

app = Flask(name)

@app.route("/page")
def page():
    name = request.values.get('name')
    age = request.values.get('age', 'unknown')
    output = Template('Hello ' + name + '! Your age is ' + age + '.').render()
return output

if name == "main":
    app.run(debug=True)
```

**Пример №2.2**
```
from flask import Flask, request
import subprocess

app = Flask(name)

@app.route("/dns")
def dns_lookup():
    hostname = request.values.get('hostname')
    cmd = 'nslookup ' + hostname
    output = subprocess.check_output(cmd, shell=True, text=True)
return output
if name == "main":
    app.run(debug=True)
```

## 3. Моделировани угроз

Изучите диаграмму потоков данных (Data Flow Diagram, DFD) сервиса, обеспечивающего отправку информации в Telegram и Slack:

![DFD](https://github.com/appseccloudcamp/test-assignment/blob/main/test-dfd.png)

Краткое описание компонентов сервиса:
 - **User** - авторизованный пользователь системы. Может настраивать отправку уведомлений и загружать изображения для дальнейшего использования при отправке уведомлений;
 - **Microfront** - микрофронт, которые позволяет взаимодействовать с сервисом отправки информации;
 - **Backend application** - набор микросервисов реализующих бизнес-логику приложения и обеспечивающих взаимодействие со всеми внешними сервисами;
 - **Auth** - сервис отвечающий за аутентификацию и авторизацию клиентов сервиса отправки информации;
 - **S3** - объектное хранилище, предназначенное для хранения статического контента сервиса отправки информации;
 - **PostgreSQL** - база данных, предназначенная для хранения пользовательских конфигураций сервиса отправки информации.    

Проанализируйте диаграмму потоков данных приложения и ответьте на следующий вопросы:
 - Расскажите, какие потенциальные проблемы безопасности существуют для данного сервиса?
 - Расскажите, к каким последствиям может привести эксплуатация проблем, найденных вами?
 - Расскажите, какие способы исправления уязвимостей и смягчения рисков вы можете предложить по отмеченным вами проблемам безопасности?
 - Напишите список уточняющих вопросов, которые вы бы задали разработчикам данного сервиса?

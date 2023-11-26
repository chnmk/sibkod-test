<template>
    <div>
        <div style="margin-top:10px;">
            <h1>Данные о диетах пользователей</h1>
            <select name="dataFilter" id="dataFilter" style="margin-top:10px;">
                <option value="all" @click="resetFilter">
                    Все данные
                </option>
                <option value="startafter" @click="startafterFilter">
                    Начинается через X дней
                </option>
                <option value="endtoday" @click="endtodayFilter">
                    Заканчивается сегодня
                </option>
                <option value="endtomorrow" @click="endtomorrowFilter">
                    Заканчивается завтра
                </option>
                <option value="endafter" @click="endafterFilter">
                    Заканчивается через Х дней
                </option>
                <option value="ended" @click="endedFilter">
                    Закончилось Х дней назад
                </option>
            </select>
        </div>
        <br>
        <div>
            <table>
                <thead>
                <tr>
                    <th scope="col">ID</th>
                    <th scope="col">Имя клиента</th>
                    <th scope="col">Диеты</th>
                    <th scope="col">Тарифы</th>
                    <th scope="col">Адрес</th>
                    <th scope="col">Номер телефона</th>
                    <th scope="col">Даты</th>
                    <th scope="col">Стоимость</th>
                    <th scope="col">Комментарий для курьера</th>
                    <th scope="col">Внутренний комментарий</th>
                    <th scope="col" 
                        @click="sortData" 
                        style="text-decoration: underline;"
                        >
                        Статус
                    </th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="value in currentData" :key="value.o_id">
                    <td style="background-color: lightgreen;">{{value.o_id}}</td>
                    <td>{{value.client_name}}</td>
                    <td>
                        <!--Перечисление всех значений в объекте через <hr>-->
                        <div v-for="(diet, index) in value.diets">
                            {{ diet }}
                            <!--Предотвращает <hr> после последнего элемента:-->
                            <hr v-if="index != value.diets.length - 1">
                        </div>
                    </td> 
                    <td>
                        <div v-for="(myTariff, index) in value.tariff">
                            {{ myTariff }}
                            <hr v-if="index != value.tariff.length - 1">
                        </div>
                    </td>
                    <td>{{value.address}}</td>
                    <td>{{value.phone}}</td>
                    <td>
                        <div v-for="(date, index) in value.dates">
                            {{ date.start_date }} - {{ date.end_date }}
                            <hr v-if="index != value.dates.length - 1">
                        </div>
                    </td>
                    <td>
                        <!--Значения объединены как в примере-->
                        <div>Стоимость: {{value.order_sum}}</div>
                        <div>Скидка: {{value.discount}}</div>
                        <div>Оплачено: {{value.order_payed}}</div>
                        <div>Состояние: {{value.pay_status}}</div>
                    </td>
                    <td>{{value.courier_comment}}</td>
                    <td>{{value.inner_comment}}</td>
                    <td>
                        <!--Вызов функции для подсчета времени-->
                        <div v-for="(date, index) in value.dates">
                            {{ displayDifference(date) }}
                            <hr v-if="index != value.dates.length - 1">
                        </div>
                    </td>
                </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script setup>
//Composition API
import jsonRaw from "../users/data.json"
import { ref, toRaw } from 'vue'

//Реактивность:
const currentData = ref(jsonRaw)

//Получение текущей даты:
const newDate = new Date()
const day = newDate.getDate()
const month = newDate.getMonth() + 1
const year = newDate.getFullYear()

const currentDateString = `${year}-${month}-${day}`
const currentDate = new Date(currentDateString)

//Грамматика. Можно улучшить, тут не все случаи:
function russianDateDisplay(num) {
    let output = ''
    if (num == 1) {
        output = ' день'
    } else if (num>1 && num<5) {
        output = ' дня'
    } else {
        output = ' дней' 
    }
    return output
}

//Объявление функции для отображения статуса:
function displayDifference(currentValue) {
    const endDate = Date.parse(currentValue.end_date)
    const startDate = Date.parse(currentValue.start_date)
    let outputString = ''
    //sortDays не отображается. Используется для сортировки по статусу:
    let sortDays = 0
    if (startDate > currentDate) {
        const diff = startDate-currentDate
        //Перевод миллисекунд в дни:
        const days = Math.floor(diff / (24*60*60*1000))
        outputString = 'Начинается через ' + days + russianDateDisplay(days)
        sortDays = days
    } else if (endDate > currentDate) {
        const diff = endDate-currentDate
        const days = Math.floor(diff / (24*60*60*1000))
        if (days == 1) {
            outputString = 'Заканчивается завтра'
            //Минус необходим для сортировки в порядке, указанном в задании:
            sortDays = -days
        } else {
            outputString = 'Заканчивается через ' + days + russianDateDisplay(days)
            sortDays = -days
        }  
    } else if (endDate < currentDate) {
        const diff = currentDate-endDate
        const days = Math.floor(diff / (24*60*60*1000))
        outputString = 'Закончилось ' + days + russianDateDisplay(days) + ' назад'
        //Умножаем на миллион, чтобы отправить завершенные заказы в конец:
        sortDays = -days*1000000
    } else {
        outputString = 'Заканчивается сегодня'
        sortDays = 0
    }
    //Запись отображаемой строки в объект (для последующей фильтрации)
    currentValue.mutatedStatus = outputString
    //Запись условного числа для сортировки
    currentValue.mutatedDays = sortDays
    return outputString
}

//Объявление функций сортировки:
let isReversed = false
function compareMutatedDays(foo, bar) {
    //Сравниваем минимальные значения в ячейке в соответствии с заданием:
    return Math.min(...foo.dates.map(a => a.mutatedDays)) - Math.min(...bar.dates.map(a => a.mutatedDays))
}
function compareMutatedDaysReverse(foo, bar) {
    return Math.min(...bar.dates.map(a => a.mutatedDays)) - Math.min(...foo.dates.map(a => a.mutatedDays))
}

//Функция для чередования порядка сортировки при нажатии:
function sortData() {
    let sortedData = toRaw(currentData.value)
    if (isReversed == true) {
        sortedData.sort(compareMutatedDays)
        isReversed = false
    } else {
        sortedData.sort(compareMutatedDaysReverse)
        isReversed = true
    }
    //Слайс вызывает реактивность:
    currentData.value = sortedData.slice()
}

//Фильтрация
function resetFilter() {
    //Восстановление стандартной таблицы:
    currentData.value = jsonRaw
}

function startafterFilter() {
    currentData.value = jsonRaw
    //Начинается через Х дней:
    let filteredData = toRaw(currentData.value)
    let newData = []
    //Ищем подходящие элементы и добавляем в новый массив:
    filteredData.forEach((element, index) => {
        let currentDays = Math.min(...element.dates.map(a => a.mutatedDays))
        if (currentDays > 0){
            newData.push(element)
        }
    })
    currentData.value = newData.slice()
}


function endtodayFilter() {
    currentData.value = jsonRaw
    //Заканчивается сегодня:
    let filteredData = toRaw(currentData.value)
    let newData = []
    filteredData.forEach((element, index) => {
        let currentDays = Math.min(...element.dates.map(a => a.mutatedDays))
        if (currentDays == 0){
            newData.push(element)
        }
    })
    currentData.value = newData.slice()
}

function endtomorrowFilter() {
    currentData.value = jsonRaw
    //Заканчивается завтра:
    let filteredData = toRaw(currentData.value)
    let newData = []
    filteredData.forEach((element, index) => {
        let currentDays = Math.min(...element.dates.map(a => a.mutatedDays))
        if (currentDays == -1){
            newData.push(element)
        }
    })
    currentData.value = newData.slice()
}

function endafterFilter() {
    currentData.value = jsonRaw
    //Заканчивается через Х дней:
    let filteredData = toRaw(currentData.value)
    let newData = []
    filteredData.forEach((element, index) => {
        let currentDays = Math.min(...element.dates.map(a => a.mutatedDays))
        if (currentDays < -1 && currentDays > -1000000){
            newData.push(element)
        }
    })
    currentData.value = newData.slice()
}

function endedFilter() {
    currentData.value = jsonRaw
    //Закончилось Х дней назад:
    let filteredData = toRaw(currentData.value)
    let newData = []
    filteredData.forEach((element, index) => {
        let currentDays = Math.min(...element.dates.map(a => a.mutatedDays))
        if (currentDays < -1000000){
            newData.push(element)
        }
    })
    currentData.value = newData.slice()
}
</script>

<style scoped>
/*Границы в таблице*/
table, th, td {
  border: 1px solid black;
}

/*Чтобы <hr> между частами ячеек не бросался в глаза:*/
hr {
  color: lightgray; 
  margin: 5px;
}

/*Можно спокойно убирать/заменять на необходимые стили:*/
table { 
    min-width: 1600px;
}

td {
    padding: 5px;
    text-align: center;
}



</style>
<template>
    <div>
        <div style="margin-top:10px;">
            <h1>Данные о диетах пользователей</h1>
            <select name="dataFilter" id="dataFilter" style="margin-top:10px;">
                <option value="all" @click="resetFilter">Все данные</option>
                <option value="startafter" @click="startafterFilter">Начинается через X дней</option>
                <option value="endtoday">Заканчивается сегодня</option>
                <option value="endtomorrow">Заканчивается завтра</option>
                <option value="endtomorrow">Заканчивается через Х дней </option>
                <option value="endtomorrow">Закончилось Х дней назад </option>
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
                    <th scope="col" @click="sortData" style="text-decoration: underline;">Статус</th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="value in currentData" :key="value.o_id">
                    <td class="idColor">{{value.o_id}}</td>
                    <td>{{value.client_name}}</td>
                    <td>
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
                        <div>Стоимость: {{value.order_sum}}</div>
                        <div>Скидка: {{value.discount}}</div>
                        <div>Оплачено: {{value.order_payed}}</div>
                        <div>Состояние: {{value.pay_status}}</div>
                    </td>
                    <td>{{value.courier_comment}}</td>
                    <td>{{value.inner_comment}}</td>
                    <td>
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
import jsonRaw from "../users/data.json"
import { ref, toRaw } from 'vue'
const currentData = ref(jsonRaw)

//debug data:
//console.log(jsonRaw)

//get current date:
const newDate = new Date()
const day = newDate.getDate()
const month = newDate.getMonth() + 1
const year = newDate.getFullYear()

const currentDateString = `${year}-${month}-${day}`
const currentDate = new Date(currentDateString)

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

//end date display funtcion:
function displayDifference(currentValue) {
    const endDate = Date.parse(currentValue.end_date)
    const startDate = Date.parse(currentValue.start_date)
    let outputString = ''
    let sortDays = 0
    if (startDate > currentDate) {
        const diff = startDate-currentDate
        const days = Math.floor(diff / (24*60*60*1000))
        sortDays = days
        outputString = 'Начинается через ' + days + russianDateDisplay(days)
    } else if (endDate > currentDate) {
        const diff = endDate-currentDate
        const days = Math.floor(diff / (24*60*60*1000))
        if (days == 1) {
            sortDays = -days
            outputString = 'Заканчивается завтра'
        } else {
            outputString = 'Заканчивается через ' + days + russianDateDisplay(days)
            sortDays = -days
        }  
    } else if (endDate < currentDate) {
        const diff = currentDate-endDate
        const days = Math.floor(diff / (24*60*60*1000))
        outputString = 'Закончилось ' + days + russianDateDisplay(days) + ' назад'
        sortDays = -days*1000
    } else {
        outputString = 'Заканчивается сегодня'
        sortDays = 0
    }
    currentValue.mutatedStatus = outputString
    currentValue.mutatedDays = sortDays
    return outputString
}

let isReversed = false

function compareMutatedDays(foo, bar) {
  return Math.min(...foo.dates.map(a => a.mutatedDays)) - Math.min(...bar.dates.map(a => a.mutatedDays))
}

function compareMutatedDaysReverse(foo, bar) {
  return Math.min(...bar.dates.map(a => a.mutatedDays)) - Math.min(...foo.dates.map(a => a.mutatedDays))
}


function sortData() {
    let sortedData = toRaw(currentData.value)
    if (isReversed == true) {
        sortedData.sort(compareMutatedDays)
        isReversed = false
    } else {
        console.log("ZA Sort start...")
        sortedData.sort(compareMutatedDaysReverse)
        isReversed = true
    }
    currentData.value = sortedData.slice()
}

///FILTERS///

function resetFilter() {
    currentData.value = jsonRaw
    console.log(currentData)
}

function startafterFilter() {
    //currentData.value = currentData.value.filter((item) => item.mutatedStatus.match(/Foo/))
    console.log(currentData.value[1].dates[1].mutatedStatus)
    //currentData.value = currentData.value.filter((item) => item.dates.mutatedStatus.includes("Начинается через"))
    let test1 = currentData.value.filter((item) => item.dates)
    console.log("test1: " + test1)
}

///FILTERS///

</script>

<style scoped>
table, th, td {
  border: 1px solid black;
}

table { 
    min-width: 1600px;
}

td {
    padding: 5px;
    text-align: center;
}

td.idColor { background-color: lightgreen; }

hr {
  color: lightgray; 
  margin: 5px;
}

</style>
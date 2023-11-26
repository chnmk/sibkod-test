<template>
    <div>
        <div style="margin-top:10px;">
            <h1>Данные о диетах пользователей</h1>
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
                    <th scope="col">Статус</th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="value in jsonRaw" :key="value.o_id">
                    <td class="idColor">{{value.o_id}}</td>
                    <td>{{value.client_name}}</td>
                    <td> <!--Либо {{value.diets.join(" / ")}}--> 
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

//debug data:
const tableHeader = jsonRaw[1]
//console.log(tableHeader)

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
    } else if (1>num>5) {
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
    if (startDate > currentDate) {
        const diff = startDate-currentDate
        const days = Math.floor(diff / (24*60*60*1000))
        outputString = 'Начинается через ' + days + russianDateDisplay(days)
    } else if (endDate > currentDate) {
        const diff = endDate-currentDate
        const days = Math.floor(diff / (24*60*60*1000))
        if (days == 1) {
            outputString = 'Заканчивается завтра'
        } else {
            outputString = 'Заканчивается через ' + days + russianDateDisplay(days)
        }  
    } else if (endDate < currentDate) {
        const diff = currentDate-endDate
        const days = Math.floor(diff / (24*60*60*1000))
        outputString = 'Закончилось ' + days + russianDateDisplay(days) + ' назад'
    } else {
        outputString = 'Заканчивается сегодня'
    }
    return outputString
}


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
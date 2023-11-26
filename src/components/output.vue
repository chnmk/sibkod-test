<template>
    <div>
        Hello world!
        {{ jsonRaw }}
    </div>
</template>

<script setup>
import jsonRaw from "../users/data.json"
console.log(jsonRaw)
</script>
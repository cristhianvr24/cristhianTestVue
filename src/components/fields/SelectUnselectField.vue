<template>
    <div class="select-unselect-container">
        <div class="options-container">
            <div class="options-box">
                <h4>Available Options</h4>
                <ul>
                    <li v-for="option in availableOptions" :key="option.id" @click="toggleOption(option)">
                        {{ option.label }}
                    </li>
                </ul>
            </div>
            <div class="options-box">
                <h4>Disabled options</h4>
                <ul>
                    <li v-for="option in disabledOptions" :key="option.id" @click="toggleOption(option)">
                        {{ option.label }}
                    </li>
                </ul>
            </div>
        </div>
    </div>
</template>

<script setup>
import { computed, defineProps, defineEmits, watch, ref } from 'vue';

const props = defineProps({
    field: {
        type: Object,
        required: true,
    },
    modelValue: {
        type: Array,
        default: () => [],
    },
});
const emit = defineEmits(['update']);

// Estado de las opciones seleccionadas, sincronizado con modelValue
const selectedOptions = ref([...props.modelValue]);

// Observar los cambios en selectedOptions y emitirlos al padre
watch(selectedOptions, (newValue) => {
    console.log("Opciones seleccionadas actualizadas:", newValue);
    emit('update', { id: props.field.id, value: newValue });
});

// Filtrar las opciones disponibles y deshabilitadas
const availableOptions = computed(() => {
  return props.field.options.filter(opt => !selectedOptions.value.includes(opt.id));
});

const disabledOptions = computed(() => {
  return props.field.options.filter(opt => selectedOptions.value.includes(opt.id));
});

// Alternar las opciones seleccionadas
const toggleOption = (option) => {
    let updatedOptions = [...selectedOptions.value];

    if (updatedOptions.includes(option.id)) {
        const index = updatedOptions.indexOf(option.id);
        updatedOptions.splice(index, 1); 
    } else {
        updatedOptions.push(option.id); 
    }

    selectedOptions.value = updatedOptions; 
}
</script>

<style scoped>
.select-unselect-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    color: white;
    width: 100%;
}
.options-container {
    display: flex;
    justify-content: space-around;
    width: 100%;
}
.options-box {
    width: 40%;
    padding: 10px;
    text-align: center;
}
ul {
    list-style: none;
    padding: 0;
    border: 1px solid white;
    min-height: 120px;
    background-color: black;
}
li {
    cursor: pointer;
    text-align: left;
    padding-left: 10px;
}
li:hover {
    background-color: grey;
}
</style>

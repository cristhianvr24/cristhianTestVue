
<template>  
    <div class="dynamic-form">
        <form @submit.prevent="onSubmit" class="form-group">
            <div v-for="(field) in fieldDefinitions" :key="field.id" class="form-field">
                <label>{{ field.label }}</label>
                <component :is="getComponentName(field)" :field="field"
                    :modelValue="modelValue ? modelValue[field.id] : ''" 
                    @update="updateFormData"
                    :ref="el => setFieldRef(field, el)" />
            </div>
            <div class="button-container">
                <button type="submit" class="button primary">Guardar</button>
            </div>

        </form>
        <label id="result-label">Result:</label><br/>
        <textarea id="result" rows="10" cols="50" disabled></textarea>
    </div>
</template>

<script setup>
import { ref, reactive, computed, defineAsyncComponent, isProxy, toRaw, watch } from 'vue';
import fieldMixin from './FieldMixin';
const emit = defineEmits(['update', 'formSubmit']);
const fieldRefs = {};

const props = defineProps({
    fieldDefinitions: {
        type: Array,
        required: true,
    },
    modelValue: {
        type: Object,
        default: () => ({}),
    }
});
let { handleChange, valueFromEvent } = fieldMixin.setup(props, { emit });

const formData = reactive({ ...props.modelValue });

watch(() => props.modelValue, (newVal) => {
    formData = toRaw(newVal);
});

const setFieldRef = (field, ref) => {
    fieldRefs[field.id] = ref;
}

const setSelected = (fieldId, value) => {
    let refs = fieldRefs;
    refs[fieldId].setSelected(value);
}

const updateFormData = (idAndValue) => {
    let { id, value } = idAndValue;
    console.log('update: ', idAndValue);

    if (Array.isArray(formData[id])) {
        let flatValue = Array.isArray(value) ? value.flat() : [value];

        formData[id].splice(0, formData[id].length, ...flatValue);
    } else {
        formData[id] = value;
    }

    emit('update', {
        id,
        value: reactive({...formData[id]}),
    });
};


const onSubmit = (event) => {

    emit('formSubmit', formData);

    // set in text area
    document.querySelector('#result').value = JSON.stringify(formData);

    event.preventDefault();
    event.stopPropagation();
};

const getComponentName = (field) => {

    if (field.type === 'SelectUnselect') {
        return defineAsyncComponent(() => import('./fields/SelectUnselectField.vue'));
    }

    let compName = field.type.charAt(0).toUpperCase() + field.type.slice(1);
    
    // Convert to CamelCase 
    compName = compName.replace(/[_-](\w)/g, (_, match) => match.toUpperCase());

    return defineAsyncComponent(() => import(`./fields/${compName}Field.vue`));
};

defineExpose({
    setSelected,
});

</script>

<style scoped>
.form-group {
    margin-bottom: 1rem;
}

.is-invalid {
    border-color: red;
}

label {
    display: inline-block;
    width: 100px;
    margin-bottom: 10px;
    color: white;

    &#result-label {
        width: 100%;
        text-align: left;
    }
}
input, select, textarea {
    padding: 5px;
    width: 100%;
    margin-bottom: 10px;
}

[disabled] {
    cursor: not-allowed;
}
.button-container {
    text-align: center;
    margin-bottom: 20px;

    button {
        margin: auto;
        color: white;
        background-color: black;
    }
}
.form-field {
    display: flex;
    align-items: center;
}
.dynamic-form{
    padding: 5%;
}
textarea{
    background-color: rgb(51, 50, 50);
}
</style>
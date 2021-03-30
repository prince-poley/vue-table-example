<template>
  <div v-if="selectedItem" class="item-info card ">
    <div class="card-body d-flex flex-column">
      <span>
        Выбран пользователь: 
        <strong>{{ selectedItem.firstName }}</strong>
      </span>
      <span>
        Описание: <textarea v-model="selectedItem.description" />
      </span>
      <span>
        Адрес проживания: 
        <strong>{{ textInfo('streetAddress') }}</strong>
      </span>
      <span>
        Город: 
        <strong>{{ textInfo('city') }}</strong>
      </span>
      <span>
        Провинция/штат: 
        <strong>{{ textInfo('state') }}</strong>
      </span>
      <span>
        Индекс: 
        <strong>{{ textInfo('zip') }}</strong>
      </span>
    </div>
  </div>
</template>

<script>
  export default {
    name: "ItemInfo",
    props: {
      selectedItem: {
        type: Object,
        default: () => null
      }
    },
    methods: {
      textInfo(field) {
        return this.selectedItem?.address?.[field] ?? "Данные отсутствуют";
      }
    }
  }
</script>

<style scoped>
  textarea {
    border: none;
    font-weight: 700;
    width: 100%;
    resize: none;
  }
</style>
<template>
  <div class="constructor__controler__add">
    <div class="constructor__controler__title">Координата № {{ turn }}</div>
    <hr />
    <label class="constructor__controler__input">
      <span>X:</span>
      <div>
        <input type="text" v-model.trim="formItem.x" />
        <span>Макс. допустимое число {{ limit.x }}</span>
      </div>
      <div class="constructor__controler__alert" v-if="formItem.invalid.x">
        Введите цифры
      </div>
    </label>
    <label class="constructor__controler__input">
      <span>Y:</span>
      <div>
        <input type="text" v-model.trim="formItem.y" />
        <span>Макс. допустимое число {{ limit.y }}</span>
      </div>
      <div class="constructor__controler__alert" v-if="formItem.invalid.y">
        Введите цифры
      </div>
    </label>
    <button
      class="constructor__controler__delete"
      @click="$emit('deleteForm', index)"
    >
      Удалить
    </button>
  </div>
</template>
<script>
export default {
  props: {
    formItem: {
      type: Object
    },
    index: {
      type: Number
    },
    turn: {
      type: Number
    },
    limit: {
      type: Object
    }
  },
  watch: {
    formItem: {
      handler() {
        this.formItem.x =
          this.formItem.x > this.limit.x ? this.limit.x : this.formItem.x;
        this.formItem.y =
          this.formItem.y > this.limit.y ? this.limit.y : this.formItem.y;
      },
      deep: true
    }
  }
};
</script>

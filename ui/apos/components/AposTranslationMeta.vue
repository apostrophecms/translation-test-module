<template>
  <AposTranslationIndicator
    v-if="isVisible"
    label="aposAutomaticTranslation:fieldMeta"
  />
</template>

<script>
import AposFieldMetaUtilsMixin from 'Modules/@apostrophecms/doc/mixins/AposFieldMetaUtilsMixin';

export default {
  name: 'AposTranslationMeta',
  mixins: [ AposFieldMetaUtilsMixin ],
  data() {
    return {
      isVisible: this.meta.data?.changed
    };
  }
};
</script>
